# AWS CONFIGURATION

1. [Create User](#create_user)
1. [Create Security Group](#create security group)
1. [Update Security Group](#update security group)

## CREATE USER

If a user does not already exist for you, get an administrator to create one
for you with associated IAM policies that allow you to launch EC2 instances

## CREATE SECURITY GROUP

In AWS console, go to EC2 console

Click on Security Groups

If 'Intern Dev Workstation Security Group' exists, then skip to next section

```
Press 'Create Security Group' button
  Security group name: 'Intern Dev SG'
  Description: 'Intern Dev SG' 
  VPC: <Leave default>
  Security group rules:
    Inbound:
      SSH - TCP -   22 - Custom IP (<CIDR Range for your laptop>)
      RDP - TCP - 3389 - Custom IP (<CIDR Range for your laptop>)
    Outbound:
      <Leave default allowing All traffic>
```

*Note: if you are behind a router, the inbound will CIDR will have to be that of your router.*

## UPDATE SECURITY GROUP

If the Security Group does exist, add the CIDR for your laptop

```
Select 'Intern Dev Security Group' button
  Add Security group rules (do NOT delete any others!)
    Inbound:
      SSH - TCP -   22 - Custom IP (<CIDR Range for your laptop>)
      RDP - TCP - 3389 - Custom IP (<CIDR Range for your laptop>)
    Outbound:
      <Leave default allowing All traffic>
```

## CREATE KEY PAIR

In AWS console, go to EC2 console
Click on Key Pairs
If a key pair already exists for you, then skip to next section
Create a new key pair with your name (e.g., mfrank_keypair)
The private key (e.g., mfrank_keypair.pem) will automatically download to your workstation

### On Windows

Open PuTTYgen (e.g., C:\Program Files (x86)\PuTTY\puttygen.exe)
Load your private key PEM file (HINT: Change file viewer to show All Files)
Enter and confirm Key passphrase (optional)
Under Type of key to generate, select SSH-2 RSA
Click on Save private key and save private key file (e.g., mfrank_keypair.ppk)
Exit PuTTYgen
Open Pageant (e.g., "C:\Program Files (x86)\PuTTY\pageant.exe")
Click on Add Key
Select private key file (e.g., mfrank_keypair.ppk)
Close Pageant (it is still running in background)
To run Pageant when you login to Windows:
  Create Pageant shortcut in Windows Startup folder
  Change shortcut target to include private key, like the following:
    "C:\Program Files (x86)\PuTTY\pageant.exe" C:\Users\Administrator\credentials\mfrank_keypair.ppk

For more details, see:
  http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html

### On Mac 

Copy your key to ~/.ssh 
chmod it to 600
to ssh to the instance:

```sh
ssh -i ~/.ssh/<keyname> centos@<aws hostname>
```

Or, set up your ssh [config](https://linux.die.net/man/5/ssh_config) file.

For more details, see:
  http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html

===============

## LAUNCH DEVELOPMENT WORKSTATION

Launch new EC2 instance to serve as CentOS 7 development workstation
Click "Launch Instance" from the EC2 console
Select "AWS Marketplace"
Search "CentOS 7"
Choose "CentOS 7 (x86_64) with Updates HVM"
Select "t2.medium"
Select "Next Configure Instance Details"
Keep Defaults except:
  Subnet: "subnet-59ddd902"
  Auto-assign Public IP: "Enabled"
  Advanced Details:
    Copy Contents of intern_dev_user_data.sh
  Configure Instance Details
Select "Add Storage"
  Add a new Volume with the following:
    Volume Type: EBS
    Device: /dev/sdb  - this is important.  Userdata will look for this device.
    Size: 20GB
    Volume Type: General Purpose SSD
    Delete On Termincation: Check
Select "Add Tags"
  Add Tags
    Name: <your user name> Dev
    Application: Intern Dev
    Schedule: Daily
Select "Configure Security Groups"
  Check 'Select an exiting security group'
  Select 'Intern Dev SG'
Select 'Review and Launch'
Review your entries
Select 'Launch'
Select your key pair

### VERIFY

After the instance launches, ssh to it:

#### MAC

ssh -vv -i <key> centos@<hostname>

CREATE ELASTIC IP
  In AWS console, go to EC2 console
  Click on Elastic IPs
  Press "Allocate New Address" button
  Select new Elastic IP
  Choose Actions - Associate Address
  Click in the Instance field, select the EC2 instance that you just launched,
  and then press "Associate" button

===============

CONFIGURE DEV WORKSTATION

Once dev workstation is up and running (i.e., green for Instance State and Status Checks),
you can login and verify the server built correctly.

LOGIN
Using PuTTY, login to dev workstation as 'centos' user

REVIEW CLOUD-INIT OUTPUT
cloud-init (i.e., user-data) execution is logged to /var/log/cloud-init-output.log
sudo more /var/log/cloud-init-output.log

SET PASSWORD FOR 'centos' USER
sudo -i
passwd centos
<enter new password>
<verify new password>
exit

SET VNC PASSWORD FOR 'centos' USER
vncpasswd
<enter same password>
<verify same password>

RELOAD VNC SERVER
sudo systemctl daemon-reload
sudo systemctl start vncserver@:1.service
sudo systemctl enable vncserver@:1.service

SET UP GIT
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
git config --global color.ui auto
git config --global push.default simple
git config --global user.name <YOUR_FIRST_NAME>
git config --global user.email <YOUR_EMAIL>

===============

LOGIN TO DEV WORKSTATION USING RDP

Open Windows Remote Desktop Connection (mstsc.exe)
Press 'Show Options' to expand settings
Keep the default settings, except the following:
  General
    Computer: Public DNS for EC2 instance
    Username: centos
  Display
    Colors: True Color (24 bit)
  Experience
    Connection speed: LAN (10 Mbps or higher)
    Font smoothing: checked
Click 'Save As' to save this configuration to you Desktop
Press 'Connect' button
If warning is displayed about identity of remote computer, then check
"Don't ask me again for connections to this computer" and press "Yes" button.
Login as 'centos' user with password you set for 'centos' user
(Xfce only) When welcomed to first start of panel, choose "Use default config"

===============

CONFIGURE WINDOW MANAGER

Xfce

(1) Google Chrome
Right-click on Desktop and select "Create Launcher..."
In Name field, enter Chrome
As you type, system will pop-up a recommendation for Google Chrome
Select recommended option for Google Chrome
Press "Create" button
From the desktop, click on newly created Google Chrome launcher
When "Untrusted application launcher" appears, press "Mark Executable" button
When prompted, leave box checked to "Make Google Chrome the default browser" and
press "OK" button.
Close Google Chrome
Right-click on Google Chrome icon on desktop and choose
'Open With "Create Launcher on the panel"' option.
From "Add New Item", select "Panel 1" and press "Add" button
  - This adds icon to top-right corner of screen
From "Add New Item", select "Panel 2" and press "Add" button
- This adds icon to bottom-middle of screen

(2) Mozilla Firefox
Right-click on Desktop and select "Create Launcher..."
In Name field, enter Firefox
As you type, system will pop-up a recommendation for Firefox
Select recommended option for Firefox
Press "Create" button
From the desktop, click on newly created Firefox Web Browser launcher
When "Untrusted application launcher" appears, press "Mark Executable" button
For "Import settings and data", select "Don't import anything" and press "Next"
Close Firefox
Right-click on Firefox Web Browser icon on desktop and choose
'Open With "Create Launcher on the panel"' option.
From "Add New Item", select "Panel 1" and press "Add" button
  - This adds icon to top-right corner of screen
From "Add New Item", select "Panel 2" and press "Add" button
  - This adds icon to bottom-middle of screen

(3) Atom
Right-click on Desktop and select "Create Launcher..."
In Name field, enter Atom
As you type, system will pop-up a recommendation for Atom
Select recommended option for Atom
Press "Create" button
From the desktop, click on newly created Atom launcher
When "Untrusted application launcher" appears, press "Mark Executable" button
Close Atom
Right-click on Atom icon on desktop and choose
'Open With "Create Launcher on the panel"' option.
From "Add New Item", select "Panel 1" and press "Add" button
  - This adds icon to top-right corner of screen
From "Add New Item", select "Panel 2" and press "Add" button
- This adds icon to bottom-middle of screen

(4) Clean Up Panels
From panel at bottom-middle of screen, right-click on generic Web Browser icon
and select "Remove" option.

Mate

(1) Google Chrome
In top left-corner, click on Applications and expand Internet
Right-click on Google Chrome and select "Add this launcher to panel"
In top left-corner, click on Applications and expand Internet
Right-click on Google Chrome and select "Add this launcher to desktop"
From the desktop, click on newly created Google Chrome launcher
When prompted, leave box checked to "Make Google Chrome the default browser"
and press "OK" button.
Close Google Chrome

(2) Mozilla Firefox
In top left-corner, click on Applications and expand Internet
Right-click on Firefox Web Browser and select "Add this launcher to desktop"
From the desktop, click on newly created Firefox Web Browser launcher
For "Import settings and data", select "Don't import anything" and press "Next"
Close Firefox

(3) Atom
In top left-corner, click on Applications and expand Programming
Right-click on Atom and select "Add this launcher to panel"
In top left-corner, click on Applications and expand Programming
Right-click on Atom and select "Add this launcher to desktop"
From the desktop, click on newly created Atom launcher
Close Atom

===============

CONFIGURE DOT FILES

Clone the dotFiles repo and follow instructions in readme.txt.

===============

UPGRADE INSTALLED PACKAGES
  sudo yum -y update
    Leave out the -y option so you have an opportunity to decide if you want to install updates or not
  sudo yum clean all

===============

FIX FROZEN DEV WORKSTATION

If RDP session ever freezes (most likely due to no available memory),
SSH to dev workstation (using PuTTY) and stop VNC server and xrdp processes
sudo systemctl stop vncserver@:1.service
sudo systemctl stop xrdp

===============

REBUILD DEV WORKSTATION FROM IMAGE

If a development workstation needs to be rebuilt from an Amazon Machine Image
(AMI), go to EC2 console, click on AMIs, select the AMI name that you want to
use for a new server, and then choose Actions - Launch.  Follow the same steps
above for launching a new development workstation, including connecting it to an
Elastic IP address.

Once the new server is up, you will probably need to set the password for the
'centos' user.  Follow the steps above for setting the password (twice), once
using the passwd command and once using the vncpasswd command.
