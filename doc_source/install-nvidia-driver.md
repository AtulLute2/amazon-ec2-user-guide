# Installing the NVIDIA Driver on Linux Instances<a name="install-nvidia-driver"></a>

A GPU\-based accelerated computing instance must have the appropriate NVIDIA driver\. The NVIDIA driver that you install must be compiled against the kernel that you plan to run on your instance\.

Depending on the instance type, you can either download a public NVIDIA driver, use an NVIDIA Marketplace offering, or download a driver from Amazon S3 that is available only to AWS customers\.

**Topics**
+ [Public NVIDIA Drivers](#obtain-nvidia-driver-linux)
+ [NVIDIA GRID Drivers for G4 Instances](#nvidia-grid-g4)
+ [NVIDIA GRID Drivers for G3 Instances](#obtain-nvidia-GRID-driver-linux)
+ [Installing the NVIDIA Driver Manually](#Cluster_GPUs_Manual_Install_Driver)
+ [Using an Alternative NVIDIA Driver](#uninstall-provided-nvidia-packages)

## Public NVIDIA Drivers<a name="obtain-nvidia-driver-linux"></a>

For instance types other than G3, or if you are not using NVIDIA GRID functionality on a G3 instance, you can download the public NVIDIA drivers\.

Download the 64\-bit NVIDIA driver appropriate for your instance type from [http://www\.nvidia\.com/Download/Find\.aspx](http://www.nvidia.com/Download/Find.aspx)\.


| Instances | Product Type | Product Series | Product | 
| --- | --- | --- | --- | 
| G2 | GRID | GRID Series | GRID K520 | 
| G4 † | Tesla | T\-Series | T4 \(version 418 or later\) | 
| P2 | Tesla | K\-Series | K\-80 | 
| P3 | Tesla | V\-Series | V100 | 

† G4 instances require driver version 418\.87 or later\.

For more information about installing and configuring the driver, choose the **ADDITIONAL INFORMATION** tab on the download page for the driver on the NVIDIA website and choose the README link\.

## NVIDIA GRID Drivers for G4 Instances<a name="nvidia-grid-g4"></a>

There are two ways that you can use NVIDIA GRID software for graphics applications on G4 instances\. You can download AMIs with GRID preinstalled or download the NVIDIA GRID vGaming driver from Amazon S3 and install it on your G4 instances\.

**Option 1: Use an AMI with GRID for your G4 instances**  
To find an AMI, use this link: [NVIDIA Marketplace offerings](http://aws.amazon.com/marketplace/search/results/?page=1&filters=instance_types&instance_types=g4dn.xlarge&searchTerms=NVIDIA%20GRID)\.

**Option 2: Download the NVIDIA GRID vGaming driver**  
You can download the NVIDIA GRID driver from Amazon S3 using this link: [NVIDIA Linux Gaming Driver for G4 Instances](https://s3.amazonaws.com/nvidia-gaming/NVIDIA-Linux-x86_64-435.22-grid.run)\.

**Important**  
This download is available to AWS customers only\. By downloading, you agree to use the downloaded software only to develop AMIs for use with the NVIDIA Tesla T4 hardware\. Upon installation of the software, you are bound by the terms of the [NVIDIA GRID Cloud End User License Agreement](http://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX)\.

If you own GRID licenses, you should be able to use those licenses on your G4 instances\. For more information, see [NVIDIA GRID Software Quick Start Guide](https://docs.nvidia.com/grid/latest/grid-software-quick-start-guide/)\.

## NVIDIA GRID Drivers for G3 Instances<a name="obtain-nvidia-GRID-driver-linux"></a>

For G3 instances, you can download the NVIDIA GRID driver from Amazon S3 using the AWS CLI or SDKs\. To install the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\. Be sure to configure the AWS CLI to use your AWS credentials\. For more information, see [Quick Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration) in the *AWS Command Line Interface User Guide*\.

**Important**  
This download is available to AWS customers only\. By downloading, you agree to use the downloaded software only to develop AMIs for use with the NVIDIA Tesla M60 hardware\. Upon installation of the software, you are bound by the terms of the [NVIDIA GRID Cloud End User License Agreement](http://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX)\.

Use the following AWS CLI command to download the latest driver:

```
[ec2-user ~]$ aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
```

Multiple versions of the NVIDIA GRID driver are stored in this bucket\. You can see all of the available versions with the following command:

```
[ec2-user ~]$ aws s3 ls --recursive s3://ec2-linux-nvidia-drivers/
```

## Installing the NVIDIA Driver Manually<a name="Cluster_GPUs_Manual_Install_Driver"></a>

If you are using an AMI that does not have the required NVIDIA driver, you can install the driver on your instance\.

**To install the NVIDIA driver**

1. Update your package cache and get necessary package updates for your instance\.
   + For Amazon Linux, CentOS, and Red Hat Enterprise Linux:

     ```
     [ec2-user ~]$ sudo yum update -y
     ```
   + For Ubuntu and Debian:

     ```
     [ec2-user ~]$ sudo apt-get update -y
     ```

1. \(Ubuntu 16\.04 and later, with the `linux-aws` package\) Upgrade the `linux-aws` package to receive the latest version\.

   ```
   [ec2-user ~]$ sudo apt-get upgrade -y linux-aws
   ```

1. Reboot your instance to load the latest kernel version\.

   ```
   [ec2-user ~]$ sudo reboot
   ```

1. Reconnect to your instance after it has rebooted\.

1. Install the gcc compiler and the kernel headers package for the version of the kernel you are currently running\.
   + For Amazon Linux, CentOS, and Red Hat Enterprise Linux:

     ```
     [ec2-user ~]$ sudo yum install -y gcc kernel-devel-$(uname -r)
     ```
   + For Ubuntu and Debian:

     ```
     [ec2-user ~]$ sudo apt-get install -y gcc make linux-headers-$(uname -r)
     ```

1. Disable the `nouveau` open source driver for NVIDIA graphics cards\.

   1. Add `nouveau` to the `/etc/modprobe.d/blacklist.conf` blacklist file\. Copy the following code block and paste it into a terminal\.

      ```
      [ec2-user ~]$ cat << EOF | sudo tee --append /etc/modprobe.d/blacklist.conf
      blacklist vga16fb
      blacklist nouveau
      blacklist rivafb
      blacklist nvidiafb
      blacklist rivatv
      EOF
      ```

   1. Edit the `/etc/default/grub` file and add the following line:

      ```
      GRUB_CMDLINE_LINUX="rdblacklist=nouveau"
      ```

   1. Rebuild the Grub configuration\.
      + For CentOS and Red Hat Enterprise Linux:

        ```
        [ec2-user ~]$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
        ```
      + For Ubuntu and Debian:

        ```
        [ec2-user ~]$ sudo update-grub
        ```

1. Download the driver package that you identified earlier as follows\.
   + For P2 and P3 instances, the following command downloads the NVIDIA driver, where *xxx*\.*xxx* represents the version of the NVIDIA driver\.

     ```
     [ec2-user ~]$ wget http://us.download.nvidia.com/tesla/xxx.xxx/NVIDIA-Linux-x86_64-xxx.xxx.run
     ```
   + For G2 instances, the following command downloads the NVIDIA driver, where *xxx*\.*xxx* represents the version of the NVIDIA driver\.

     ```
     [ec2-user ~]$ wget http://us.download.nvidia.com/XFree86/Linux-x86_64/xxx.xxx/NVIDIA-Linux-x86_64-xxx.xxx.run
     ```
   + For G3 instances, you can download the driver from Amazon S3 using the AWS CLI or SDKs\. To install the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\. Use the following AWS CLI command to download the latest driver:

     ```
     [ec2-user ~]$ aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
     ```
**Important**  
This download is available to AWS customers only\. By downloading, you agree to use the downloaded software only to develop AMIs for use with the NVIDIA Tesla M60 hardware\. Upon installation of the software, you are bound by the terms of the [NVIDIA GRID Cloud End User License Agreement](http://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX)\.

     Multiple versions of the NVIDIA GRID driver are stored in this bucket\. You can see all of the available versions with the following command:

     ```
     [ec2-user ~]$ aws s3 ls --recursive s3://ec2-linux-nvidia-drivers/
     ```

1. Run the self\-install script to install the NVIDIA driver that you downloaded in the previous step\. For example:

   ```
   [ec2-user ~]$ sudo /bin/sh ./NVIDIA-Linux-x86_64*.run
   ```

   When prompted, accept the license agreement and specify the installation options as required \(you can accept the default options\)\.

1. Reboot the instance\.

   ```
   [ec2-user ~]$ sudo reboot
   ```

1. Confirm that the driver is functional\. The response for the following command lists the installed NVIDIA driver version and details about the GPUs\.
**Note**  
This command may take several minutes to run\.

   ```
   [ec2-user ~]$ nvidia-smi -q | head
   ```

1. \[G3 instances only\] To enable NVIDIA GRID Virtual Applications, complete the GRID activation steps in [Activate NVIDIA GRID Virtual Applications on G3 Instances](activate_grid.md) \(NVIDIA GRID Virtual Workstation is enabled by default\)\.

1. Complete the optimization steps in [Optimizing GPU Settings](optimize_gpu.md) to achieve the best performance from your GPU\.

## Using an Alternative NVIDIA Driver<a name="uninstall-provided-nvidia-packages"></a>

Amazon provides AMIs with updated and compatible builds of the NVIDIA kernel drivers for each official kernel upgrade in the AWS Marketplace\. If you decide to use a different NVIDIA driver version than the one that Amazon provides, or decide to use a kernel that's not an official Amazon build, you must uninstall the Amazon\-provided NVIDIA packages from your system to avoid conflicts with the versions of the drivers that you are trying to install\.

Use this command to uninstall Amazon\-provided NVIDIA packages:

```
[ec2-user ~]$ sudo yum erase nvidia cuda
```

The Amazon\-provided CUDA toolkit package has dependencies on the NVIDIA drivers\. Uninstalling the NVIDIA packages erases the CUDA toolkit\. You must reinstall the CUDA toolkit after installing the NVIDIA driver\.