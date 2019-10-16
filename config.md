Here's what I did to get Hashcat to run on an Amazon EC2 c4.8xlarge instance running Ubuntu 16.04. I don't claim that this is the best way, or even the correct way, to install Hashcat. But it seems to have worked for me. 

1) Create an AWS account, log in, configure a c4.8xlarge instance with an Ubuntu 16.04 image, create/download the `.pem` file, create a security group to allow access via ssh, and spin up the box.

2) Give the `.pem` file the correct permissions and ssh into the instance (replacing mycert.pem and the instance url with your actual pem file and instance url).
<br/> $ `chmod 400 mycert.pem`
<br/> $ `ssh -i mycert.pem ubuntu@ec2-10-200-30-400.us-east-2.compute.amazonaws.com`

3) Update apt and install OpenCL
<br/>$ `sudo apt update`
<br/>$ `sudo apt upgrade`
<br/>$ `sudo apt install ocl-icd-libopencl1`
<br/>$ `sudo apt install opencl-headers`
<br/>$ `sudo apt install clinfo`
<br/>$ `sudo apt install ocl-icd-opencl-dev`
<br/>$ `wget "http://registrationcenter-download.intel.com/akdlm/irc_nas/12513/opencl_runtime_16.1.2_x64_rh_6.4.0.37.tgz"`
<br/>$ `sudo apt-get install -y rpm alien libnuma1`
<br/>$ `tar -xvf opencl_runtime_16.1.2_x64_rh_6.4.0.37.tgz`
<br/>$ `cd opencl_runtime_16.1.2_x64_rh_6.4.0.37/rpm/`
<br/>$ `fakeroot alien --to-deb opencl-1.2-base-6.4.0.37-1.x86_64.rpm`
<br/>$ `fakeroot alien --to-deb opencl-1.2-intel-cpu-6.4.0.37-1.x86_64.rpm`
<br/>$ `sudo dpkg -i opencl-1.2-base_6.4.0.37-2_amd64.deb`
<br/>$ `sudo dpkg -i opencl-1.2-intel-cpu_6.4.0.37-2_amd64.deb`
<br/>$ `sudo touch /etc/ld.so.conf.d/intelOpenCL.conf`
<br/>$ `sudo vim /etc/ld.so.conf.d/intelOpenCL.conf` 
<br/>paste the following into `intelOpenCL.conf` and save the file: _/opt/intel/opencl-1.2-6.4.0.37/lib64/clinfo_
<br/>$ `sudo mkdir -p /etc/OpenCL/vendors`
<br/>$ `sudo ln /opt/intel/opencl-1.2-6.4.0.37/etc/intel64.icd /etc/OpenCL/vendors/intel64.icd`
<br/>$ `sudo ldconfig`
<br/>$ `clinfo`
<br/>$ `cd /home/ubuntu`
<br/>$ `wget "https://codeload.github.com/hpc12/tools/tar.gz/master"`
<br/>$ `tar xzvf master`
<br/>$ `cd tools-master`
<br/>$ `make`
<br/>$ `./print-devices`

4) Clone and build hashcat
<br/>$ `cd /home/ubuntu`
<br/>$ `sudo apt install cmake build-essential`
<br/>$ `sudo apt install checkinstall git`
<br/>$ `git clone https://github.com/hashcat/hashcat.git`
<br/>$ `cd hashcat`
<br/>$ `git submodule update --init`
<br/>$ `make`

5) Install screen so you can start a cracking session and return to it later.
<br/>$ `sudo apt-get install openssh-server openssh-client screen`
<br/>$ `exit`

6) Move the seed file to the remote instance (assuming the seed file is on your desktop). <br/>I used ethereum2john.py to create my seed from my backup wallet.
<br/>$ `scp -i ~/Desktop/mycert.pem ~/Desktop/myseed.txt  ubuntu@ec2-10-20-30-400.us-east-2.compute.amazonaws.com:~/hashcat`

Here are a couple of sources to help you get started actually using hashcat: 
<br/>• https://hashcat.net/wiki/doku.php?id=mask_attack
<br/>• https://www.4armed.com/blog/perform-mask-attack-hashcat/

**Don't forget to spin down your instances when not in use. Good Luck!**