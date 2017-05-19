# AWS CONFIGURATION

1. [Create User](#create-user)
1. [Create Security Group](#create-security-group)
1. [Update Security Group](#update-security-group)
1. [Create Key Pair](#create-key-pair)
1. [Setup Client](#setup-client)
   * [On Windows](#on-windows)
   * [On Mac](#on-mac)
1. [Launch Development Instance](#launch-development-instance)
1. [Verify](#verify)
1. [Create Elastic IP](#create-elastic-ip)
1. [Configure Host Workstation](#configure-host-workstation)
   * [Login](#login)
   * [Review Cloud-init Output](#revie-cloud-init-output)
   * [Set Password For `centos` User](#set-password-for-centos-user)
   * [Set VNC Password](#set-vnc-password)
   * [Reload VNC Server](#reload-vnc-server)
   * [Set Up Git](#set-up-git)
   * [Login To Dev Instance Using RDP](#login-to-dev-instance-using-rdp)
   * [Configure dotfiles](#configure-dotfiles)
   * [Upgrade Installed Packages](#upgrade-installed-packages)
   * [Fix Frozen DEV Instance](#fix-frozen-dev-instance)

## Create User

If a user does not already exist for you, get an administrator to create one
for you with associated IAM policies that allow you to launch EC2 instances

## Create Security Group

In AWS console, go to EC2 console

Click on Security Groups

If 'Intern Dev Workstation Security Group' exists, then skip to next section

* Press 'Create Security Group' button
  * Security group name: 'Intern Dev SG'
  * Description: 'Intern Dev SG'
  * VPC: <Leave default>
  * Security group rules:
    * Inbound:
      * SSH - TCP -   22 - Custom IP (<CIDR Range for your laptop>)
      * RDP - TCP - 3389 - Custom IP (<CIDR Range for your laptop>)
    * Outbound:
      * <Leave default allowing All traffic>

*Note: if you are behind a router, the inbound will CIDR will have to be that of your router.*

## Update Security Group

If the Security Group does exist, add the CIDR for your laptop


* Select 'Intern Dev Security Group' button
  * Add Security group rules (do NOT delete any others!)
    * Inbound:
      * SSH - TCP -   22 - Custom IP (<CIDR Range for your laptop>)
      * RDP - TCP - 3389 - Custom IP (<CIDR Range for your laptop>)
    * Outbound:
      * <Leave default allowing All traffic>


## Create Key Pair

1. In AWS console, go to EC2 console
2. Click on Key Pairs
3. If a key pair already exists for you, then skip to next section
4. Create a new key pair with your name (e.g., <yourname>_keypair)
5. The private key (e.g., <yourname>_keypair.pem) will automatically download to your workstation

## Setup Client

### On Windows

1. Open PuTTYgen (e.g., C:\Program Files (x86)\PuTTY\puttygen.exe)
1. Load your private key PEM file (HINT: Change file viewer to show All Files)
1. Enter and confirm Key passphrase (optional)
1. Under Type of key to generate, select SSH-2 RSA
1. Click on Save private key and save private key file (e.g., <yourname>_keypair.ppk)
1. Exit PuTTYgen
1. Open Pageant (e.g., "C:\Program Files (x86)\PuTTY\pageant.exe")
1. Click on Add Key
1. Select private key file (e.g., <yourname>_keypair.ppk)
1. Close Pageant (it is still running in background)
1. To run Pageant when you login to Windows: 
   * Create Pageant shortcut in Windows Startup folder 
   * Change shortcut target to include private key, like the following:
     * ` "C:\Program Files (x86)\PuTTY\pageant.exe" C:\Users\Administrator\credentials\<yourname>_keypair.ppk`

For more details, see:
  http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html

### On Mac

1. Copy your key to ~/.ssh
1. chmod chmod the key to 600

#### To ssh to the instance:

```sh
ssh -i ~/.ssh/<keyname> centos@<aws hostname>
```

Or, set up your ssh [config](https://linux.die.net/man/5/ssh_config) file.

```
Host ec2dev
  Hostname <hostname>
  User centos
  IdentityFile /Users/<user>/.ssh/<user>.pem
  ForwardX11 yes
  ForwardX11Trusted yes

```

#### Via safari

Open Safari

(haven't gotten above to work yet) From what I have read it is supposed to works

in the url type:

```
vnc://<ec2 instance hostname>
```


For more details, see:
  http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html

---

## Launch Development Instance

1. Launch new EC2 instance to serve as CentOS 7 development workstation
1. Click "Launch Instance" from the EC2 console
1. Select "AWS Marketplace"
1. Search "CentOS 7"
1. Choose "CentOS 7 (x86_64) with Updates HVM"
1. Select "t2.medium"
1. Select "Next Configure Instance Details"
1. Keep Defaults except:
   * Subnet: "subnet-818b73c9"
   * Auto-assign Public IP: "Enabled"
   * Advanced Details:
      * Copy Contents of [userdata.yml](userdata.yml) into User Data or use the userdata file uploader
   * Configure Instance Details
1. Select "Add Storage"
   * Add a new Volume with the following:
      * Volume Type: EBS
      * Device: /dev/sdb  - this is important.  Userdata will look for this device.
      * Size: 20GB
      * Volume Type: General Purpose SSD
      * Delete On Termincation: Check
1. Select "Add Tags"
   * Add Tags
      * Name: `<your name>` Dev
      * Application: Intern Dev
      * Schedule: Daily
1. Select "Configure Security Groups"
   * Check 'Select an exiting security group'
   * Select 'Intern Dev SG'
1. Select 'Review and Launch'
1. Review your entries
1. Select 'Launch'
1. Select your key pair


### Verify

After the instance launches, ssh to it:

#### MAC

```
ssh -vv -i <key> centos@<hostname>
```

Tail /var/log/cloud-init-output.log and make sure that it completes successfully

## Create Elastic IP

####  (Optional)

1.  In AWS console, go to EC2 console
1.  Click on Elastic IPs
1.  Press "Allocate New Address" button
1.  Select new Elastic IP
1.  Choose Actions - Associate Address
1.  Click in the Instance field, select the EC2 instance that you just launched, and then press "Associate" button

---

## Configure Host Workstation

Once dev workstation is up and running (i.e., green for Instance State and Status Checks),
you can login and verify the server built correctly.

### Login

Using PuTTY, login to dev workstation as 'centos' user if you are on windows

### Review Cloud-init Output

cloud-init (i.e., user-data) execution is logged to /var/log/cloud-init-output.log
sudo more /var/log/cloud-init-output.log

### Set Password For `centos` User

```
sudo -i
passwd centos
<enter new password>
<verify new password>
exit
```

### Set VNC Password

```
vncpasswd
<enter same password>
<verify same password>
```

### Reload VNC Server

```
sudo systemctl daemon-reload
sudo systemctl start vncserver@:1.service
sudo systemctl enable vncserver@:1.service
```

### Set Up Git

```
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
git config --global color.ui auto
git config --global push.default simple
git config --global user.name <YOUR_FIRST_NAME>
git config --global user.email <YOUR_EMAIL>
```

### Login To Dev Instance Using RDP

Open Windows Remote Desktop Connection (mstsc.exe)

Press 'Show Options' to expand settings

Keep the default settings, except the following:


1. General
   * Computer: Public DNS for EC2 instance
   * Username: centos
1. Display 
   * Colors: True Color (24 bit)
1. Experience 
   * Connection speed: LAN (10 Mbps or higher)
   * Font smoothing: checked
1. Click 'Save As' to save this configuration to you Desktop 
1. Press 'Connect' button

If warning is displayed about identity of remote computer, then check
"Don't ask me again for connections to this computer" and press "Yes" button.
Login as 'centos' user with password you set for 'centos' user
(Xfce only) When welcomed to first start of panel, choose "Use default config"

---

### Configure Window Manager

Xfce

(1) Google Chrome

1. Right-click on Desktop and select "Create Launcher..."
1. In Name field, enter Chrome
1. As you type, system will pop-up a recommendation for Google Chrome
1. Select recommended option for Google Chrome
1. Press "Create" button
1. From the desktop, click on newly created Google Chrome launcher
1. When "Untrusted application launcher" appears, press "Mark Executable" button
1. When prompted, leave box checked to "Make Google Chrome the default browser" and
1. press "OK" button.
1. Close Google Chrome
1. Right-click on Google Chrome icon on desktop and choose
1. 'Open With "Create Launcher on the panel"' option.
1. From "Add New Item", select "Panel 1" and press "Add" button 
    - This adds icon to top-right corner of screen
1. From "Add New Item", select "Panel 2" and press "Add" button 
    - This adds icon to bottom-middle of screen

(2) Mozilla Firefox

1. Right-click on Desktop and select "Create Launcher..."
1. In Name field, enter Firefox
1. As you type, system will pop-up a recommendation for Firefox
1. Select recommended option for Firefox
1. Press "Create" button
1. From the desktop, click on newly created Firefox Web Browser launcher
1. When "Untrusted application launcher" appears, press "Mark Executable" button
1. For "Import settings and data", select "Don't import anything" and press "Next"
1. Close Firefox
1. Right-click on Firefox Web Browser icon on desktop and choose
1. 'Open With "Create Launcher on the panel"' option.
1. From "Add New Item", select "Panel 1" and press "Add" button
    - This adds icon to top-right corner of screen
1. From "Add New Item", select "Panel 2" and press "Add" button 
    - This adds icon to bottom-middle of screen
 

(3) Atom


1. Right-click on Desktop and select "Create Launcher..."
1. In Name field, enter Atom
1. As you type, system will pop-up a recommendation for Atom
1. Select recommended option for Atom
1. Press "Create" button
1. From the desktop, click on newly created Atom launcher
1. When "Untrusted application launcher" appears, press "Mark Executable" button
1. Close Atom
1. Right-click on Atom icon on desktop and choose
1. 'Open With "Create Launcher on the panel"' option.
1. From "Add New Item", select "Panel 1" and press "Add" button
     - This adds icon to top-right corner of screen
1. From "Add New Item", select "Panel 2" and press "Add" button 
     - This adds icon to bottom-middle of screen


(4) Clean Up Panels

From panel at bottom-middle of screen, right-click on generic Web Browser icon
and select "Remove" option.


Mate

(1) Google Chrome


1. In top left-corner, click on Applications and expand Internet
1. Right-click on Google Chrome and select "Add this launcher to panel"
1. In top left-corner, click on Applications and expand Internet
1. Right-click on Google Chrome and select "Add this launcher to desktop"
1. From the desktop, click on newly created Google Chrome launcher
1. When prompted, leave box checked to "Make Google Chrome the default browser" and press "OK" button.
1. Close Google Chrome


(2) Mozilla Firefox


1. In top left-corner, click on Applications and expand Internet
1. Right-click on Firefox Web Browser and select "Add this launcher to desktop"
1. From the desktop, click on newly created Firefox Web Browser launcher
1. For "Import settings and data", select "Don't import anything" and press "Next"
1. Close Firefox


(3) Atom


1. In top left-corner, click on Applications and expand Programming
1. Right-click on Atom and select "Add this launcher to panel"
1. In top left-corner, click on Applications and expand Programming
1. Right-click on Atom and select "Add this launcher to desktop"
1. From the desktop, click on newly created Atom launcher
1. Close Atom


### Configure dotFiles

Clone the dotFiles repo and follow instructions in readme.txt.


### Upgrade Installed Packages

```sh
  sudo yum -y update
  #  Leave out the -y option so you have an opportunity to decide if you want to install updates or not
  sudo yum clean all
```

### Fix Frozen DEV Instance

If RDP session ever freezes (most likely due to no available memory),
SSH to dev workstation (using PuTTY) and stop VNC server and xrdp processes

```sh
sudo systemctl stop vncserver@:1.service
sudo systemctl stop xrdp
```
