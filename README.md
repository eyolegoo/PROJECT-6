# PROJECT-6
WEB SOLUTION WITH WORDPRESS


## WEB SOLUTION WITH WORDPRESS

***Step 1 — Prepare a Web Server***

- Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

- Learn How to Add EBS Volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw)

<img width="519" alt="Creating volume" src="https://user-images.githubusercontent.com/115954100/218573209-8ddda957-acfd-4af0-8012-cf92a3b5fe49.png">

- Attach all three volumes one by one to your Web Server EC2 instance

<img width="505" alt="Creating volume 2" src="https://user-images.githubusercontent.com/115954100/218573744-981dbefa-dc32-408e-855b-0131fc32e756.png">

- Open up the Linux terminal to begin configuration

- Use lsblk [lsblk](https://man7.org/linux/man-pages/man8/lsblk.8.html) command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in **/dev/ directory**. Inspect it with ***ls /dev/*** and make sure you see all 3 newly created block devices there – their names will likely be ***xvdf, xvdh, xvdg***.

<img width="506" alt="Logical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218575935-d3a8bb4c-3cf7-4b3b-bca4-99a1b08ff742.png">

- Use `df -h` command to see all mounts and free space on your server

- Use `gdisk` utility to create a single partition on each of the 3 disks

- `sudo gdisk /dev/xvdf`

<img width="501" alt="Creating disk partition" src="https://user-images.githubusercontent.com/115954100/218576667-44861d66-a688-43b9-b339-9520c39eb25a.png">

- Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

- <img width="336" alt="Newly Configured Partitions" src="https://user-images.githubusercontent.com/115954100/218577029-9eb3cecd-3dee-4034-9ef1-a5f60e8e46a0.png">

- Install lvm2 [lvm2](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

- Note: Previously, in Ubuntu we used **apt** command to install packages, in RedHat/CentOS a different package manager is used, so we shall use **yum** command instead.

- Use [pvcreate](https://linux.die.net/man/8/pvcreate) utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

- `sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

- Verify that your Physical volume has been created successfully by running `sudo pvs`

<img width="469" alt="Physical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218651983-2ddcb7da-515e-430a-a755-342e40d80234.png">

- Use [vgcreate](https://linux.die.net/man/8/vgcreate) utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**

- `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

- Verify that your VG has been created successfully by running `sudo vgs`

<img width="530" alt="Volume group created and verified" src="https://user-images.githubusercontent.com/115954100/218653319-9b99c8e5-cc3a-44c9-b98f-ffe9cf31ea76.png">

- Use [lvcreate](https://www.example.com) utility to create 2 logical volumes. **apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE:** apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

- `sudo lvcreate -n apps-lv -L 14G webdata-vg` and `sudo lvcreate -n logs-lv -L 14G webdata-vg`

- Verify that your Logical Volume has been created successfully by running `sudo lvs`

<img width="506" alt="Logical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218654733-a2af8e65-593c-4bca-85e8-546ba331d156.png">

- Verify the entire setup

- `sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
`

![lsblk3](https://user-images.githubusercontent.com/115954100/218656019-dffaca38-8aad-475d-af3a-c155073db4bd.png)

- Use **mkfs.ext4** to format the logical volumes with [ext4](https://en.wikipedia.org/wiki/Ext4) filesystem

- `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv` and `sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

- Create **/var/www/html** directory to store website files

- `sudo mkdir -p /var/www/html`

- Create **/home/recovery/logs** to store backup of log data

- `sudo mkdir -p /home/recovery/logs`

- Mount **/var/www/html** on **apps-lv** logical volume

- `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

- Use [rsync](https://linux.die.net/man/1/rsync) utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (This is required before mounting the file system)

- `sudo rsync -av /var/log/. /home/recovery/logs/`

- Mount **/var/log** on **logs-lv** logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

- `sudo mount /dev/webdata-vg/logs-lv /var/log`

- Restore log files back into **/var/log** directory

- `sudo rsync -av /home/recovery/logs/. /var/log`

- Update **/etc/fstab** file so that the mount configuration will persist after restart of the server.

- The UUID of the device will be used to update the **/etc/fstab** file;

- `sudo blkid`

