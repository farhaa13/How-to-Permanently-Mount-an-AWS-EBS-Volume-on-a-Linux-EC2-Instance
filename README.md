# How-to-Permanently-Mount-an-AWS-EBS-Volume-on-a-Linux-EC2-Instance
Permanent Mounting an AWS EBS Volume

This guide provides a complete, step-by-step walkthrough for creating an AWS EBS volume, attaching it to an EC2 Linux instance, and configuring it to mount automatically and permanently on every system boot.
Following these steps ensures that your storage is always available after a reboot, which is critical for applications that rely on persistent data.

# Part 1: Creating and Attaching the EBS Volume
1.In your AWS Console, navigate to the EC2 Dashboard.

2.Go to Elastic Block Store > Volumes and click Create volume.

3.Configure your volume:

Volume type: gp3 or gp2 (General Purpose SSD) is suitable for most use cases.

Size (GiB): Choose your desired size, for example, 12.

Availability Zone: This is critical. You must select the same Availability Zone as your EC2 instance.

Tags: It's good practice to add a Name tag (e.g., web-server-data) for easy identification.

4.Click Create volume.

2. Attach the Volume to Your EC2 Instance
1.Once the volume's status is Available, select it.

2.Click Actions > Attach volume.

3.Select your target EC2 instance from the Instance dropdown.

4.AWS will suggest a Device name (e.g., /dev/sdf). Note this down.

5.Click Attach volume.

# Part 2: Preparing the Volume in Linux
Now, we'll SSH into the instance to format and prepare the volume.

1. Connect to your EC2 Instance
Use SSH to connect to your instance.
```
ssh -i /path/to/your-key.pem ec2-user@your-instance-public-ip
```

2. Identify the New Volume
Run lsblk to list the block devices. You should see the new volume you attached. AWS might rename it (e.g., /dev/sdf may appear as /dev/xvdf).
```
lsblk
```
Example Output:
<img width="765" height="250" alt="vol-mount1" src="https://github.com/user-attachments/assets/ef170052-02a8-4020-870a-ba6285a5feb2" />

In this example, /dev/xvdf is our new 12 GiB volume.

3. Check for an Existing Filesystem
 Important: Before formatting, check if the volume already contains data. New volumes will have no data.
```
sudo file -s /dev/xvdf
```
For a new volume, the output will be: ``` /dev/xvdf: data ```

If it has a filesystem, it will show the type, e.g., ``` /dev/xvdf: Linux rev 1.0 ext4 filesystem data.... ``` Do not proceed with formatting if you need the data!

4. Create a Filesystem (for new volumes only)
If the volume is new (data), create a filesystem on it. ext4 is a safe and common choice.
```
sudo mkfs -t ext4 /dev/xvdf
```

5. Create a Mount Point
This is the directory where the volume's contents will be accessible.
```
sudo mkdir /var/www/html
```

# Part 3: Making the Mount Permanent with fstab
To ensure the volume mounts automatically on boot, we must add it to the /etc/fstab file. We will use the volume's unique ID (UUID) because device names like /dev/xvdf can sometimes change after a reboot.

1. Find the Volume's UUID
Use the blkid command to get the UUID of your newly formatted volume.
```
sudo blkid
```
Example Output:
<img width="1817" height="204" alt="vol-mount2" src="https://github.com/user-attachments/assets/74a521d6-633f-479b-896a-4b72a7baf8b0" />

Copy the UUID value for your device (/dev/xvdf in this case).

2. Back Up Your fstab File
Before editing, it's always wise to create a backup.
```
sudo cp /etc/fstab /etc/fstab.bak
```

3. Edit fstab and Add the Mount Entry
Open /etc/fstab with a text editor.
```
sudo nano /etc/fstab
```
Add the following line to the end of the file. Replace the UUID and mount point with your own values.
```
# <device_UUID>                             <mount_point>   <filesystem>  <options>         <dump> <pass>
UUID=1933a113-c31b-4cad-a2b7-372d4b3b263     /var/www/html   ext4          defaults,nofail   0      2
```

``` UUID ```=...: The unique ID you copied.

``` /var/www/html ```: Your mount point directory.

``` ext4 ```: The filesystem type you used.

``` defaults,nofail ```: defaults uses standard options. nofail prevents the system from getting stuck during boot if the EBS volume is missing or fails to mount. This is highly recommended for cloud instances.

``` 0 2 ```: Standard fstab values for error-checking.

Save the file and exit the editor.

# Part 4: Verifying the Permanent Mount
You can test your fstab entry without needing to reboot the entire system.

1. Mount the Volume Using fstab
The mount -a command mounts all filesystems listed in /etc/fstab that are not already mounted.
```
sudo mount -a
```

If this command runs without any errors, your fstab entry is correct.

2. Check the Mount
Verify that the volume is mounted correctly using the df command.
```
df -hT
```

Example Output:
```
Filesystem     Type      Size  Used Avail Use% Mounted on
...
/dev/xvdf      ext4       12G   45M   19G   1% /var/www/html
```

You can see that /dev/xvdf is now permanently mounted at /var/www/html































































