---
sort: 1
---

# COMPUTING & IT 

## Cloud Computing and VMs

### Setting Up a Linux VM

This section details the steps to setup a __Ubuntu Linode VM__ following the this [guide](https://www.linode.com/docs/guides/set-up-and-secure/).

- Use the cloud manager to create and view the linux instance.
- Before accessing the instance through **PuTTy**, access the instance using the _root_ login via the **web console**. **NOTE** Most cloud services creates an admin account for access which we will create manually in subsequent steps.
- Perform **system updates**: `apt update && apt upgrade`
- Set **local timezone**: `timedatectl set-timezone 'Asia/Calcutta'`
  - To view all timezones: `timedatectl list-timezones`
- Set **hostname**: `hostnamectl set-hostname <hostname>`
- Customize **prompt** in `.bashrc`: `PS1="\u@\h[\W]\$ "`
  - `\u`:username, `\h`:hostname, `\W`: working directory
- Add some essential aliases to .bashrc:

```bash
alias date='date +%D'
alias h='history'
alias vimr='vim -R'
alias rm='rm -i'
```

- Add the above customization to `/etc/skel/.bashrc` for new users.
- Create an **admin/sudo** account:
  - Create the new user: `adduser <admin-username>`
  - Add the new user to the sudo group: `adduser <admin-username> sudo`
  - For other distros eg. CentOS or even Ubuntu:
    - `useradd <adminuser> && passwd <adminuser>`
    - `usermod -aG wheel <adminuser>`
    - Make sure the wheel group is uncommented in `/etc/skel` using the command `#visudo`
      - `%wheel ALL=(ALL) ALL`  **NOTE** `%` is NOT a comment.
- **Install and Configure Dropbox**
  - `sudo apt-get install dropbox`
  - Login to the user you want to use Dropbox in.
  - Download the python script (to say `~/bin`):
    - `wget https://www.dropbox.com/download?dl=packages/dropbox.py`
    - **NOTE** the dropbox package comes with a application name `dropbox` but seems to have some outdated code so downloaded the above python script. 
  - Make sure `~/bin` is in the PATH.
    - eg. in `.bashrc`: `export PATH=/home/user/bin:$PATH` 
  - `dropbox.py update` to update the package?
  - `dropbox.py start`
    - _First Time_: `dropbox status` will show you a link that you browse to enter the Dropbox account credential that you want to link to.
  - Make sure everytime the VM is rebooted, you start dropbox.
  - _NOTE_ If your distro's repo does not have the packages, Check this Dropbox Links: [Installs](https://help.dropbox.com/installs), [On Linux](https://help.dropbox.com/installs/linux-commands#add).

- Follow section [Security Hardening a Linux Server](wiki/compute-it.html#security-hardening-a-linux-server)  for hardening access to the server.
- **Install & Configure AWS CLI**
  - AWS CLI is mainly used for automatic backups to AWS S3.
  - Followed this [AWS User Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html).
  - Prereqs:
    - Setup an IAM account (instead of the root AWS) or use an existing one.
    - Create an access ID key and secret key and store it safely. See [AWS Doc](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-prereqs.html).
  - Before installing, make sure `glibc-tools`, `groff` and `less` are installed.
  - Install it from AWS:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
  - `aws --version` to check the installed package.
  - To update the current isntallation:
    - `sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update`
  - **NOTE** You can install `awscli` package from the Ubuntu repo but lot older version.   
  - **Configure AWS CLI** (See [AWS Doc](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html))
    - `aws configure --profile user1`  **NOTE** For default configuration omit the --profile option.
      - AWS Access Key ID: <Enter the Key ID of your IAM user>
      - AWS Secret Access Key: <Enter it>
      - Default region name: Give the one most used eg. `ap-southeast-1`
      - Default output format: default is `json` but use table for more human readable format.
  - Now you are ready to use the **AWS CLI**
    - You need to create a S3 bucket for use. See [AWS Doc](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html).
      - Example options when creating a bucket:
        - AWS Region: ap-southeast-1
        - Copy settings from another bucket. (Specially if rules related Glacier, you don't have to repeat them here.)
        - Object owner: ACL disable 
        - Block all public access (private)
        - Versioning Enabled
        - Add a tag. (Helps is tracking for eg. cost )
        - Server side encryption enabled with default SSE-S3.
    - `aws s3 ls --profile user1` will list all the S3 buckets.
    - `aws s3 cp --profile user1 <src file> s3://bucket-name`  

- **Additional Packages**
  - To build pacakages from source install the essentials:
    - `sudo apt instal build-essential bison flex libx11-dev libxaw7-dev libtool libreadline-dev libncurses-dev automake autoconf texinfo`
      - If you need the kernel headers: `sudo apt install linux-headers-'uname -r'`
    - `sudo apt install texlive-font-utils ghostscript` : For `epstpdf`,....

### VIRTUAL BOX

**ADDING A WINDOWS 10 SHARED FOLDER IN UBUNTU 22.04**

- After installation and _before rebooting_, make sure the installation ISO image is removed from the VM's storage:
  - `Settings -> Storage` and select the ISO and from `Attributes -> Optical Drive` select _Remove Disk from Virtual Drive`
- After reboot and login, install some essentials:
  - `sudo apt-get update && sudo apt-get install -y build-essential linux-headers-\`uname -r\``
- From VM's VBox Menu select `Devices -> Insert Guest Additions CD image`
- `lsblk` to find the mount point say `/media/cdrom/VBoxLinusGAs`
- Run the installation script:
  - `sudo /media/cdrom/VBoxLinuxGAs/VBoxLinuxAdditions.run`
- Create the folder to mount the share say, `/home/ubuntu/share`
- Add the user to `vboxsf` group (not sure if necessary but doesn't hurt):
  - `sudo adduser ubuntu vboxsf`
- Shutdown the VM: `sudo shutdown -h now`
- Select from VM's Vbox Menu: `Settings -> Shared Folder -> Add New..`:
  - Folder Path: `<Path to Windows 10 Folder>`
  - Folder Name: `vbshare` **NOTE** This will be name of the folder in Ubuntu that will be used to mount.
  - Mount point: `/home/ubuntu/share`
  - _Select_ `Auto-mount`
  - _Select_ `Make Permanent`
- Start the VM and it should be mounted. **If NOT** Follow these steps:
  - Add the following in `/etc/fstab`:
    - `vbshare /home/ubuntu/share vboxsf defaults 0 0`
    - Test it: `sudo mount -a`
  - Add `vboxsf` in `/etc/modules` (Not sure if necessary)
- Resources:
  - [Mounting VirtualBox shared folders on Ubuntu Server 18.04 LTS](https://gist.github.com/estorgio/0c76e29c0439e683caca694f338d4003)
  - [How to access a shared folder in VirtualBox? - askUbuntu](https://askubuntu.com/questions/161759/how-to-access-a-shared-folder-in-virtualbox) 

### git

Tags: #git #github
 
**ADDING A PROJECT TO A GITHUB REPO**

- Create a repo without any README/.gitignore/license
- If you are going to usee SSH (recommended), copy the SSH key to github from the "Settings->SSH/GPG Keys" menu.
- Navigate to the project directory:
  - `git init`
  - `git add .`
  - `git commit -m "init" -a`
  - `git remote add origin <URL from Github->Code>`
  - `git remote -v` to check
  - `git branch -M main` Assuming this is the default branch of your repo.
  - `git push -u origin main` 

**ADDING SSH KEYS github.com**

- github doesn't support https for maintaing repo using API eg. from Linux. Sp most preferre way is to clone and maintain using SSH so need to add SSH keys to the github account.
- Generating 4096-bit RSA key from **Linux**:
  - `ssh-keygen -t rsa -b 4096 -C "email"`
    - _Note_: DSA keys has been dropped by github to improve security. github recommends `ed25119` but is ok with 4096-bit RSA for legacy systems. Decided to use above not to have any issues with legacy systems. See github doc [here](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- To add the keys to github account: click setting from the top-right menu, slect "SSH and GPG keys" and click "New SSK Key" and copy paste from `~/.ssh/id_rsa.pub`

- When adding a SSH key to github if you get an error saying "Key already in use" but you don't see it in the list, Use the following command to see which github account is used in:
  - `ssh -T -ai ~/.ssh/id_rsa git@github.com`

- If the output is "hi username .." then that git user is using the above key.

**USING MULTIPLE GIT ACCOUNTS FROM ONE LINUX ACCOUNT**

- Create different keys for different accounts eg.:
  - `ssh-keygen -t rsa -b 4096 -C "default"` : This will create the private/public pair id_ras/id_rsa.pub
  - `ssh-keygen -t rsa -b 4096 -f git1 -C "github.com/git1"`
  - `ssh-keygen -t rsa -b 4096 -f git2 -C "github.com/git2"`
- Add the public key to the respective git account. Add the default key to the account most used so you will not need to add follow the steps to add the keys to ssh-agent before doing a git push.
- **NOTE** All documentation suggest to create the file `~/.ssh/ssh_config` file to automatically use the respective keys but just couldn't get it to work.
- After loging in and let's say you want to use account git1:
  - `eval "$(ssh-agent -s)"` Starts the ssh-agent.
  - `ssh-add ~/.ssh/git1` Adds the private key for git1
    - `ssh-add -l` will list all the keys in the agent
    - `ssh-add -D` will delete all the keys.
  - Now you can do all the git operation for account git1: `git clone/add/commit/push/etc`
  - Before loging out, kill the ssh-agent process: `pkill -9 -f ssh-agent`
  - Probably a good idea to add it to crontab.

**DEALING WITH LARGE PAK FILES**

If you are dealing with binaries, even pdfs docs etc, then the git repo blows pretty quickly. Especially after deleting all the binaries, the revisions are still in the pack files in `.git/objects/...` . So how do you delete them ? Followed this [link](https://support-acquia.force.com/s/article/360004334093-Removing-large-files-from-Git-without-losing-history#:~:text=Branch%20filtering,particular%20files%20from%20your%20history.)

- `git verify-pack -v .git/objects/pack/pack-{hash}.idx | sort -k 3 -n | tail -n 20` -- lists the 20 largest pak files.
- `git rev-list --objects --all | grep {hash}`  -- This will list the file path for this hash.
- `git filter-branch --index-filter 'git rm --cached --ignore-unmatch ./path/to/resource/*.ext' --tag-name-filter cat -- --all`
- If you know there was an entire diretory (say `docs/public_html`) that has been deleted and you don't need it, you can delete:
  - `git filter-branch --index-filter 'git rm --cached --ignore-unmatch docs/public_html/*' --tag-name-filter cat -- --all`
- `git push origin --force --all`
- `git push origin --force --tags` -- If you want to purge the tags as well.
- Now, the above command will purge the history from the githib repo but the backup will be created in the .git local directory so there will be no space saving in the current working driectory. Still don't know how to clean it up properly. The way I do it now is move the directory and just clone it again.

## SINGLE BOARD COMPUTERS (SBC)

### NAS on RaspberryPi 4

After install and trying to configure services **OpenMediaVault**, decided not to go with it because of restrictive options and problems with features.

**INSTALLING RASPBERRY PI OS LITE**

- Download and start the Raspberry Pi imager.
- Choose the OS type: `Raspberry Pi OS Lite (64 bit)`
- Choose the target SD card.
- From _settings_ preconfigure avaialble options eg. `ssh creds, WiFi creds, hostname, etc.`. This is essential for _headless_ install.
- Login using PuTTy using the preconfigured SSH user and update OS and install essential eg. `vim`

**INSTALLING UFW FIREWALL (OPTIONAL)**
 #ufw #firewall

NOT NEEDED if unit is inside a firewall.

- `sudo apt install ufw`
- Before rebooting make sure you allow SSH: `sudo ufw allow 22`
- Check the status `sudo ufw status`

**MOUNTING EXTERNAL SSD**

- Find the block device name eg. `/dev/sda` using the command `lsblk`
- Create a partition (not necessary but highly recommended) using `fdisk`:
  - `fdisk -l` to list all partitions available.
  - `fdisk /dev/sda`
  - `n` for creating a partition. For creating one Linux partition use the defaults.
  - `p` print the partition for checking.
  - `w` write the partition to disk.
- Now you can see the block device for the new partition eg. `/dev/sda1` using `lsblk`
- Create a `ext4` partition: `sudo mkfs.ext4 /dev/sda1`
- Find the _UUID_ of the partition: `blkid /dev/sda1`
- Create a mount point: `sudo mkdir /media/wd220`
- Create a fstab entry in `/etc/fstab`:
  - `UUID=5abdf860-950b-40b8-8799-49f6ce70044c /media/wd220 ext4 defaults,auto,users,rw,nofail 0 0` 
  - **FIXME** document the options.
- `sudo mount -a` 

**CREATING A SAMBA SHARE**

Could not get a Public share to work. Write permission error from Windows. After trying out lot of ways, following seem to work for a Private share.

- Create the share directory: `sudo mkdir /media/wd220/Music`
- Change owner, group and permission to the user that will be a samba user say smbuser:
  - `sudo chown smbuser /media/wd220/Music`
  - `sudo chgrp users /media/wd220/Music`
  - `sudo chmod 2775 /media/wd220/Music` The 2 in the begining makes the folder sticky so users in the group "users" can write to the directory with their ownership.
- Install _samba_ : `sudo apt-get install samba smb-client cifs-utils`
- Add the Windows group (eg. WORKGROUP) to the global option in `/etc/samba/smb.conf`
- Add the share folder to `/etc/samba/smb.conf`:

```bash
[Music]
   comment = Public Music Folder
   path = /media/wd220/Music
   read only = no
   guest ok = no
   valid users = smbuser
```

- `sudo smbpasswd -a smbuser` Add the user as a Samba user. You are going to use this credential when accessing the folder from Windows.
- Allow the SMB ports 139,445 in the firewall (if ufw enabled):

```bash
sudo ufw allow 139
sudo ufw allow 445
```

- `sudo systemctl restart smbd` Restart the SMB daemon.
- Now try accessing from Windows `Win+R` and entering `\\[IP/hostname]/Music`
- Some useful links: [devconnected](https://devconnected.com/how-to-install-samba-on-debian-10-buster/#:~:text=In%20order%20for%20Samba%20to,on%20ports%20139%20and%20445.) | [ComputingForGeeks](https://computingforgeeks.com/how-to-configure-samba-share-on-debian/?expand_article=1) | [ServerSpace](https://serverspace.io/support/help/configuring-samba-on-debian/)

**LOGITECH MEDIA SERVER (LMS) on RaspberryPiOS**
 #LMS #lms #squeezebox

This tutorial documents the steps in installing **Logitech Media Server** for organizing Music files. Also, this installation was done on the Raspberry Pi OS that is already running OpenMediaVault (OMV) . So all filesystems are managed through OMV.

Followed the LMS part of [Harald Kreuzer's Blog](https://www.haraldkreuzer.net/en/news/installing-logitech-media-server-raspberry-pi-4b-5-inch-display). **NOTE** There is section in the blog about disabling the swap to increase the longetivity of the SD card. Worth looking into it.

Important steps are listed below:

- `sudo apt-get update && sudo apt-get upgrade -y` followed by `sudo reboot`
- **Install LMS Server**:

```
sudo apt-get install libsox-fmt-all libflac-dev libfaad2 
sudo apt-get install libio-socket-ssl-perl 
sudo apt-get install libcrypt-openssl-bignum-perl 
sudo apt-get install libcrypt-openssl-random-perl 
sudo apt-get install libcrypt-openssl-rsa-perl 
wget https://downloads.slimdevices.com/LogitechMediaServer_v8.3.1/logitechmediaserver_8.3.1_arm.deb 
sudo dpkg -i logitechmediaserver_8.3.1_arm.deb
``` 
**NOTE** Check the latest LMS distro available.

- `sudo ufw allow 9000`
- The server will now be accessible at `http://<IP>:9000/`
  - **NOTE** you do not need to create the logitech account. You can skipt.
- **FIXME** Create the appropriate directory structure in OMV and configure that here.
  - You can check these links ([BegineersGuide](https://wiki.slimdevices.com/index.php/Beginners_Guide_To_Organising.html), [Survey](http://www.hydrogenaudio.org/forums/index.php?showtopic=32726), [Organize4DJ-MP3Tags](https://homedjstudio.com/organize-music-library/)) to see some popular ways to organize folders for music.
  - One popular choice is: `/<library>/<artist>/<album>/<tracks>`
  - Organizing Classical Music is different (See [BegineersGuideToClassical](https://wiki.slimdevices.com/index.php/BeginnersGuideToClassical.html). A simple _tagging_ (Tagged during ripping) reco from the guide:
    - the folder strategy I am using now is: `/<lib>/<composer>/<CDalbum>/<tracks>`
    - For Various artiests: `/<lib>/<VariousArtist>/<CDalbum>/<tracks>`
    - **Tag** reco:
      - _Album_ tag for _Work_ eg. "Beethoven Symphony no. 5 - Karajan"
      - _Artist_ tag for _Composer_ eg. "Beethoven"
      - _Title_ tag for _Movement_ eg. "Beethoven Symp 5 - 3- Allegro"
- You can remove all unnecessary plugins. 
- It's worth installing the `material` plugin which is a responsive plugin so the server will be accessible at `https://<IP>:9000/material` even from a mobile.

**AWS CLI**

- Check this [section](https://mixignal.github.io/wiki/compute-it.html?highlight=aws%20cli#setting-up-a-linux-vm) for installing aws cli tools.
- **Note**, download the ARM version instead of the x86 version: 
  - `curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"`

**Setting crontab for syncing from AWS S3**
- `00 03 * * 1 aws s3 sync --delete --profile srout s3://linode-vm01-bak /media/wd220/aws-S3/srout/linode-vm01-bak > /home/srout/logs/s3sync-linodevm01.log 2>&1`

**OpenMediaVault**

After install and trying to configure services, decided not to go with it because of restrictive options and problems with features. Following links are good guides if you decide to:

- [Installing OpenMediaVault to a Raspbeery Pi](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwizmoe02KGAAxXFavUHHUvVAVgQFnoECA4QAQ&url=https%3A%2F%2Fpimylifeup.com%2Fraspberry-pi-openmediavault%2F&usg=AOvVaw1Iw1MJa1bGIa_1SJTSwJWb&opi=89978449)
- [How to Install OpenMediaVault on a Raspberry Pi](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwizmoe02KGAAxXFavUHHUvVAVgQFnoECA0QAQ&url=https%3A%2F%2Fwww.makeuseof.com%2Finstall-openmediavault-raspberry-pi%2F&usg=AOvVaw2--kLBPwz6WF0Yf27PES_x&opi=89978449)


## Security

### Security Hardening a Linux Server

- Some good guides for securing your server/VM:
  - This section details the steps to setup a __Ubuntu Linode VM__ following the this [guide](https://www.linode.com/docs/guides/set-up-and-secure/).
  - [Thread from Linode community](https://www.linode.com/community/questions/467/ive-noticed-some-suspicious-activity-on-my-linode-what-do-i-do): on tracking suspious activity including running anti-virus and antimalware.
  - [Fail2Ban Tutorial from Linode](https://www.linode.com/docs/guides/using-fail2ban-to-secure-your-server-a-tutorial/)

- **Harden SSH Access** by adding authentication via private/public key pair and disable password access.
  - For **PuTTy**, use **PuTTygen** to generate private/public key pair.
  - use 4096-bit RSA or ECDSA to generate the key pair.
  - Save the private key in safe location and add it to the PuTTy session: `Connection -> SSH -> Auth -> Private key file for authentication`
  - Add the public key to the Linux VM instance: `~/.ssh/authorized_keys`
  - Andd now when loging in for that particular user, you will not require to use the password.
- **SSH Daemon Options** in `/etc/ssh/sshd_config`:
  - Disable _root_ login via SSH: `PermitRootLogin no`
  - Disable _password auth_: `PasswordAuthentication no`
  - If using only IPv4 then: `AddressFamily inet`
  - _Restart_ ssh daemon: `sudo systemctl restart sshd`
- Use **Fail2Ban** to secure `sshd` service (and other services as well):
  - [Linode Tutorial on Fail2Ban](https://www.linode.com/docs/guides/using-fail2ban-to-secure-your-server-a-tutorial/) to install and configure fail2ban.
  - **Install** (without sendmail and for CentOS make sure `epel-release` is installed and systems are up to date:
    - **Ubuntu** (22.04): `#apt install fail2ban`
    - **CentOS** (7): `#yum install fail2ban`
  - Make sure `ssh` is enabled through firewall.
  - Make local copies of:
    - `/etc/fail2ban/fail2ban.conf -> /etc/fail2ban/fail2ban.local` : Default settings should be good enough for the start.
    - `/etc/fail2ban/jail.conf -> /etc/fail2ban/jail.local`
  - Configure `jail.local`:
    - Uncomment ignore ip such that local host is not filtered and add any static IPs you regularly log from:
```
ignoreip = 127.0.0.1/8 ::1 <Any other Public IPs>
```
    - If `sendmail` is not installed, change `mta` to regular `mail` (Not sure if you really need this):
```
mta = mail`
```
    - Enable the `sshd` jail:
```
[sshd]
enabled = true
```
  - For CentOS, change `backend` from `auto` to `systemd`:
```
backend = systemd
```
  - Start and enable the service so it starts at boot:
    - `#systemctl enable fail2ban`
    - `#systemctl start fail2ban`
    - `#systemctl status fail2ban` to check the status if it started or any error.
  - You can also the check the status through the client app: 
    - `#fail2ban-client status` which will show the active jails like this:
```
Status
|- Number of jail:      1
`- Jail list:   sshd
```
  - And you can see the details of jail:
    - `#fail2ban-client status sshd` which will show the details of the jail `sshd`:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 4
|  |- Total failed:     62
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     8
   `- Banned IP list:
```
  - **NOTE** the log file `/var/log/auth.log` is specified in `/etc/fail2ban/paths-common.conf`
  - You can use `journalctl` to browse through the logs.
   
- **Configure Firewall**. The default application in Ubuntu `ufw` (Uncomplicated Firewall) is disabled. Follow the [Linode Tutorial](https://www.linode.com/docs/guides/configure-firewall-with-ufw/) to install and setup the firewall. Basic setup steps:
  - Allow SSH connections: `sudo ufw allow ssh`
  - Use the default rules:
    - `sudo ufw default allow outgoing`
    - `sudo ufw default deny incoming`
  - Enable it: `sudo ufw enable`
  - Check the status: `sudo ufw status`

### System Log Files

**Using `journactl` Command**

System logs are a vital resource for administrators and developers looking to monitor system performance, troubleshoot issues, and understand the events taking place within the system. 
Check this  [Linux Journal article](https://www.linuxjournal.com/content/mastering-journalctl-command-comprehensive-guide), to learn about this command. Here are some useful ways to use `journalctl`:

- `$ journalctl -r` : new entries first.
- `$ journalctl -n 15` : view last 15 entries.
- `$ journalctl -p 3` : priority level 3. Note, most sshd entries are in this level.
- `$ journalctl -u fail2ban.service` : logs of a particular service.
- `$ journalctl --list-boots`: Logs from previous boots.
  - `$ journalctl -b -1`: Logs from a particular boot.
- `$ journalctl -f` : real-time monitoring. 
- `$ journactl --since yesterday`
- `$ journalctl --disk-usage` : Shows the disk usage of the journal log files.
  - To clean of the journal logs to a limited **size**:
    - `$ journalctl --rotate` to move active logs to archive and 
    - `$ journalctl --vaccum-size=250M` will delete past logs to limit size to `250Mbytes`.
  - To make it automatic, edit `/etc/systemd/journald.conf`
```
SystemMaxUse = 250M
```
  - restart service:
```bash
sudo systemctl restart systemd-journald
```
  - To clean of the journal logs to a limited **time**:
    - `$ journalctl --rotate` to move active logs to archive and 
    - `$ journalctl --vaccum-time=1d` will delete past logs from begining to given time.

### CLI Password Vault pass

**NOTE** This installation is done at **USER LEVEL**. So make sure `pass` is installed eg. `sudo apt install pass`

- `gpg --full-generate-key` skip this step if already installed.
  - select the kind eg. `RSA`
  - keysize eg. `4096`
  - validity eg. `0` (forever)
  - Enter user info

- `gpg --list-secret-keys --keyid-format LONG` will generate something like this:

```
sec   4096R/AAAA2222CCCC4444 2021-03-18 [expires: 2023-03-18] uid         John Doe <jdoe@example.com>
```

- `pass init 'AAAA2222CCCC4444'` to initiate pass using the GPG key ID.
  - **IMPORTANT** Make the passphrase strong and keep it in a safe place.

- `pass generate -c Internet/github.com 21` to generate a 21-char-long password and copy it to clipboard.

- `pass insert -em banking/citi` will insert a multiline (m) info and echo (e) it back. Press `Ctrl-D` when done entering.

- `pass show banking/citi` displays the password.

- **Setting up for Syncing with Git Repo**:
  - Create a git repo in github.com (can be something else too)
  - `pass git init` will initialize pass DB fir git repo sync.
  - `pass git remote add origin git@github.com:johndoe/pass-store.git`
    - Make sure the proper authentication is in place.
  - `pass git push -u --all`

- **Cloning the git pass**
  - From the server export the keys:
    - `gpg --export-secret-keys > privkey.asc`
    - `gpg --export > pubkey.asc`
    - Make sure you store in a safe place.
  - Import on the new client:
    - `gpg --import pubkey.asc`
    - `gpg --allow-secret-key-import --import privkey.asc`
  - Clone the git repo from the server (FIXME: How to clone from github?)
    - Setup SSH keys between client-server if it's setup that way.
    - `git clone user@server:~/.password-store`
    - Now you can use the client pass but push from client has issues. So best to use it only for pull. 

## Networking

### Peer-to-Peer VPN using tinc

**SETUP on CentOS 7**

- Mostly followed this [blog](https://www.digitalocean.com/community/tutorials/how-to-install-tinc-and-set-up-a-basic-vpn-on-ubuntu-18-04) on Digital Ocean. **Note**: The blog is for Ubuntu 18.04
- The setup was done on two linux machines: VM on cloud (`docosvm01`) and a server behind a firewall (`vlsisrv02`), both running CentOS 7.
- Install `tinc` on both the machines: `sudo yum install tinc`
  - Make sure the `epel` repo is enabled.
- The setup's goal is to have `docosvm01` listen to connect requests from other clients eg. `vlsisrv02`. Primary reason being the VM has public address which avoids any NAT problems. The clients can be behind a NAT without any issues.
- Create the configuration directory on both the machines (`tincvpn1` is name for this VPN conn):
  - `sudo mkdir -p /etc/tinc/tincvpn1/hosts` 
- Create the config file `/etc/tinc/tincvpn1/tinc.conf` with the following content:

```bash
Name = docosvm01
AddressFamily = ipv4
Interface = tun0
```
**IMPORTANT NOTE**: The interface Name (`docosvm01`) has to **different** from the `hostname`.

- Create a host config file (name same as above `Name`) `/etc/tinc/tincvpn1/hosts/docosvm01` with the following content:

```bash
Address = <Public IP of the VM>
Subnet = 10.0.0.1/32
```

- `sudo tincd -n tincvpn1 -K4096` : create the public/private pair keys
  - Choose the default options. The public key will get autimatically appended to the host file.
- Create the link-up script `/etc/tinc/tincvpn1/tinc-up` :

```bash
#!/bin/sh
ip link set $INTERFACE up
ip addr add 10.0.0.1/32 dev $INTERFACE
ip route add 10.0.0.0/24 dev $INTERFACE
```

- Create the link-down script `/etc/tinc/tincvpn1/tinc-down` :

```bash
#!/bin/sh
ip route del 10.0.0.0/24 dev $INTERFACE
ip addr del 10.0.0.1/32 dev $INTERFACE
ip link set $INTERFACE down
```

- `sudo chmod 755 /etc/tinc/tincvpn1/tinc-*`
- Open the port `655` (not sure which so opened both tcp/udp):

```bash
sudo firewall-cmd --add-port=655/udp --permanent
sudo firewall-cmd --add-port=655/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-port
```

For **Ubuntu** you can use `ufw`: `sudo ufw allow 655/tcp ; sudo ufw reload`


- **Confiuring the Client** `vlsisrv02`:
- `sudo mkdir -p /etc/tinc/tincvpn1/hosts`
- `/etc/tinc/tincvpn1/tinc.conf`:

```bash
Name = vlsisrv02
AddressFamily = ipv4
Interface = tun0
ConnectTo = docosvm01
```

- `/etc/tinc/tincvpn1/hosts/vlsisrv02`:

```bash
Subnet = 10.0.0.2/32
```

- `sudo tincd -n tincvpn1 -K4096` : Create key pairs.

- `/etc/tinc/tincvpn1/tinc-up`:

```bash
#!/bin/sh
ip link set $INTERFACE up
ip addr add 10.0.0.2/32 dev $INTERFACE
ip route add 10.0.0.0/24 dev $INTERFACE
```

- `/etc/tinc/tincvpn1/tinc-down`:

```bash
#!/bin/sh
ip route del 10.0.0.0/24 dev $INTERFACE
ip addr del 10.0.0.2/32 dev $INTERFACE
ip link set $INTERFACE down
```

- `sudo chmod 755 /etc/tinc/tincvpn1/tinc-*`
- Open the ports `655`:

```bash
sudo firewall-cmd --add-port=655/udp --permanent
sudo firewall-cmd --add-port=655/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-port
```

- **Distributing the Keys**
- If the SSH keys are used between the servers, make sure the public keys are appropriately added in `~/.ssh/authorized_keys`
- Using `scp`, copy:
  - `docosvm01:/etc/tinc/tincvpn1/hosts/docosvm01` to `vlsisrv02:/etc/tinc/tincvpn1/hosts/` 
  - `vlsisrv02:/etc/tinc/tincvpn1/hosts/vlsisrv02` to `docosvm01:/etc/tinc/tincvpn1/hosts/` 
  - **Note**: Although the blog suggests to change the public IP of `docosvm01` to the VPN subnet IP, you don't have to.

- **Testing the setup**:
  - `sudo tincd -n netname -D -d3` on both the servers will start the daemon in foreground (`-D`) in verbose debug mode (`-d3`).
  - If it runs successfully you can `ping 10.0.0.2` from `docosvm01`
- `sudo systemctl start tinc@tincvpn1` will start the daemon in the background.
- `sudo systemctl enable tinc@tincvpn1` will start the daemon at startup. **FIXME** presently not starting at boot.
  

**ADDING ANOTHER TINC SERVER**

- If you want to add another _server_ say `docosvm02` for peer-to-peer access to the same client `vlsisrv02`:
- Setup `docosvm02` just like `docosvm01` above.
- Add 'ConnectTo' to the `vlsisrv02:/etc/tinc/tincvpn1/tinc.conf` to include `docosvm02` as well:

```bash
Name = vlsisrv02
AddressFamily = ipv4
Interface = tun0
ConnectTo = docosvm01
ConnectTo = docosvm02
```

- Copy the keys: `docosvm02:/etc/tinc/tincvpn1/hosts/docosvm02` to `vlsisrv02:/etc/tinc/tincvpn1/hosts/` 

- Restart `tinc` on `vlsisrv02`: `sudo systemctl restart tinc@tincvpn1`
- If all goes well, that's it!

### PPTP VPN client 

**SETUP on CentOS 7**

- Followed this [blog](https://zlthinker.github.io/Setup-VPN-on-CentOS) to setup the VPN
- Install PPTP: `sudo yum install pptp pptp-setup`
- Configuration: `sudo pptpsetup –create bmt-229 –server [server address] –username [username] –password [pwd] –encrypt`
- This command will create a file named `bmt-229` under `/etc/ppp/peers/` with server info written inside.
- This command will also write your username and password into `/etc/ppp/chap-secrets`
- Register the ppp_mppe kernel module: `sudo modprobe ppp_mppe`
- Register the nf_conntrack_pptp kernel module: `sudo modprobe nf_conntrack_pptp`

**USER GUIDE**
    
- Connect to VPN PPTP: `sudo pppd call config`
- It will establish PPTP VPN connection. You can type command `ip a | grep ppp` to find the connection name (e.g. `ppp0`). No return indicates connection failure.
- If any error, you can look into `/var/log/messages` for log info
- Check IP routing table info: `route -n`
- Add Network Segment to current connection: 
  - `route add -net 192.168.11.0 netmask 255.255.255.0 dev ppp0`
- You can now ping the destination to check the access
- Disconnect the VPN: `sudo killall pppd`

### PuTTY

**EXPORTING and IMPORTING a SESSION**
- In windows search and start the *Registry Editor* progam `regedit.exe`
- Navigate to `HKEY_CURRENT_USER -> Software -> SimonTatham -> PuTTY -> Sessions`
- Right Click on the session name and export it.
- To import, `File -> Import`

**DELETING A KEY FROM A SESSION**
- In windows search and start the *Registry Editor* progam `regedit.exe`
- Navigate to `HKEY_CURRENT_USER -> Software -> SimonTatham -> PuTTY -> SshHostKeys`
- Select the appropriate key and delete it.


## WebSite/Wiki

### Jekyll: Static Page on GitHub

[#jekyll](https://jekyllrb.com) | #github | [#jekyll-rtd-theme](https://github.com/rundocs/jekyll-rtd-theme) | #website | #static | [#jekyll-rtd-userguide](https://jekyll-themes.com/jekyll-rtd/) 

This section shows you how to create a static web page using [Jekyll](https://jekyllrb.com) (and a Jekyll theme) and host it on github.

**PREPARING LINUX FOR JEKYLL**
  * First we need install the prereqs on a Linux workstation. Following is for the `Ubuntu` on `AWS`:
  * `sudo apt-get install ruby-full build-essential zlib1g-dev`
  * In order to load `gem` locally, add the following in `.bashrc`
    * `export GEM_HOME=$HOME/gems`
    * `export PATH=$HOME/gems/bin:$PATH`
  * `gem install jekyll bundler`

**CONFIGURING JEKYLL**
  * Site-wise configuration are done using `_config.yml`
  * See https://jekyll-rtd-theme.rundocs.io/ for config options.
  * **IMPORTANT** Option `baseurl` when testing a site that doesn't sit at the root of the server domain. See this [blog](https://byparker.com/blog/2014/clearing-up-confusion-around-baseurl/#:~:text=Set%20baseurl%20in%20your%20_config,baseurl%20to%20in%20your%20_config) for more detail on it.
  * Someone changed this to `/silicon-vlsi.github.io` and all urls had duplicate domain eg `https://silicon-vlsi.github.io/silicon-vlsi.github.io/content/projects.html` and thus breaking the links.
  * Removed the `baseurl` and `url` as well since hosting on github automatiacally takes care of it. I think. It works so far.

**USING A JEKYLL TEMPLATE IN GITHUB**
  * Login to your github account eg. `silicon-vlsi`
  * Navigate to the template repo (eg. [#jekyll-rtd-theme](https://github.com/rundocs/jekyll-rtd-theme) and click `Fork`
  * Rename (from the repo's settings) the copied repo to the following format:
    * `<username>.github.io`
    * eg. `silicon-vlsi.github.io`
  * Give it few minutes to publish it and browse to `http://silicon-vlsi.github.io` to see the website!

**USING JEKYLL TO MAINTAIN THE SITE**
  * Clone the repo to your prepared Linux workstation:
    * `git clone https://github.com/silcion-vlsi/silicon-vlsi.github.io`
  * Change directory `cd` to `silicon-vlsi.github.io` and edit `_config.yml` change the info.
  * For the first time after clone, to get the dependencies:`bundle install`
    * `bundle update` FIXME Document this
  * Build the site again after the changes:`bundle exec jekyll build`
  * `git commit --all [--allow-empty] -m "comment"` FIXME: Document when we need `--allow-empty`
  * `git push`

**CONTENT MANAGEMENT**

The directory structure (USR tag indicates changes made by the user and SYS typically should be left untouched and synced with the original repo):

```bash
.
├── README.md              : USR: Content for the landing page
├── _config.yml            : USR: Site-wide configuration
├── _includes              : SYS: All includes: common codes, etc
├── _layouts               : SYS: site layout
├── _sass                  : SYS: ??
├── _site                  : SYS: Compiled html site here
├── assets                 : SYS: CSS themes etc.
├── content                : USR: Main site content goes here.
│   ├── README.md
│   ├── Resources
│   ├── people.md
│   ├── projects.md
│   └── training.md
└── wiki                   : USR: The second content page
    ├── README.md
    ├── doc1
    ├── doc2
    └── quickref.md

```

**SYNCING THE LOCAL FORK WITH ORIGINAL UPSTREAM REPO**
FIXME Refer a proper documentation for this and put some more detail in this documentation. 

  * Related github docs: [Config a remote for fork](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/configuring-a-remote-for-a-fork), [Syncing a fork](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/syncing-a-fork)
  * **Onetime** config remote upstream repo with the fork:
    * List the current configured remote repository for your fork.`git remote -v`
    * Specify the remote upstream repository that will be synced with the fork:`git remote add upstream  https://github.com/rundocs/jekyll-rtd-theme.git`
    * Verify: `git remote -v`
  * Syncing the fork withe upstream repo:
    * Fetch the branches and their respective commits from the upstream repository. Commits to BRANCHNAME will be stored in the local branch upstream/BRANCHNAME: `git fetch upstream`
    * Check out your fork's local default branch - in this case, we use `develop` FIXME need more clarity on this one:`git checkout develop(?)`
    * Merge the changes from the upstream default branch - in this case, `upstream/develop` - into your local default branch. This brings your fork's default branch into sync with the upstream repository, without losing your local changes:`git merge upstream/develop`
    * Push the changes to the fork:`git push`


### LOGOS

  * [What Does the Color of Your Logo Say About Your Business? (Infographic)](http://www.entrepreneur.com/article/232401)


### Creating favicon

  * Generate a **16x16** image (Gimp, Inkscape, etc) eg. favicon.png
  * Convert it to a ppm or pnm format eg: `$ pngtopnm favicon.png > favicon.pnm `
    * **NOTE** If you have more than 256 colors, you'll get an error. You can quantize it to 256 using `$ pnmquant 256 favicon.pnm > temp.pnm; mv temp.pnm favicon.pnm`
  * Convert using the the utility `ppmtowinicon` : `$ ppmtowinicon -output favicon.ico favicon.pnm`

## Media

### Audio-Video

**YouTube DOWNLOAD**

- `sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp`
- `sudo chmod a+rx /usr/local/bin/yt-dlp`
- `sudo yt-dlp -U` : To update
- **NOTE** The `yt-dlp` and `youtube-dl` packages in Ubuntu 22.04 was broken as of 2023/03/10

**EXTRACTING AUDIO FROM MP4**

- Install `ffmpeg` : `sudo apt install ffmpeg`
- To extract: `ffmpeg -i <file.mp4> out.mp3`
- See script `~/bin/yt-extract`

## Open-Source

**Resources**

- [Redhat: What is open source software?](https://www.redhat.com/en/topics/open-source/what-is-open-source-software)
- [opensource.com:Which open source software license should I use?](https://opensource.com/law/13/1/which-open-source-software-license-should-i-use?extIdCarryOver=true&sc_cid=701f2000001OH6fAAG)

### Licensing

- No copy left: MIT, new BSD
- Some copy left with patenting & licensing 
- All copy left: GPLs 
