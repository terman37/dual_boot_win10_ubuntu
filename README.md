# My Dual boot Win10 / ubuntu 20.04

Notes on how to setup proper dual boot Win/Linux

on my Asus Zenbook UX534FT

( ! sound not working for now - except in HDMI ) / **fixed** by script 2020-07-28

### Prepare Windows

- Disable bitlocker

- Shrink partition to make some space available ( > 50Go) - unformated

- Download ubuntu iso from https://ubuntu.com/download/desktop

- Create bootable usb disk using balenaEtcher

- Disable Fast Startup

  power options / Change settings that are currently unavailable / uncheck FastStartup

### Modify BIOS (UEFI) settings

- disable fastboot
- disable secure boot

### Boot from Usb key

- Select install ubuntu
- follow setup:
  - select language (English)
  - select keyboard layout (French - French (AZERY)) - check it's ok.
  - select normal installation - use nvidia drivers...
  - select install along Windows (it will detect and install it on empty partition created earlier)
  - define user / password / laptop name...
  - continue...
- it will reboot (remove usb stick)

### Boot in Ubuntu

#### Auto install script:

All steps below have been automated using script. 

download install.sh, allow execution and run it

```bash
./install.sh
```

#### Settings Setup:

- disable Screenpad display

  - settings / devices...

- set ubuntu to use LocalTime to avoid time difference in windows when rebooting:

  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```

- check graphic card used
  
  - settings / details 
  
  - and/or
  
    ```bash
    nvidia-smi
    ```

#### Automount Windows Partition

- create a folder ( ex: in /home/user/Windows )
- find windows partition UUID:

```bash
ls -l /dev/disk/by-partlabel/
ls -l /dev/disk/by-uuid/
```

- edit fstab

```bash
sudo gedit /etc/fstab
```

- add line in fstab (it will be mounted after reboot)

  ```
  UUID=<uuid found before> <path_to_folder_created> ntfs-3g defaults 0 0
  ```

- force mounting now:

  ```bash
  sudo mount -a
  ```

#### Update

```bash
sudo apt update
sudo apt upgrade
```

#### Hide icon of mounted drives from the dock

- install **dconf editor**
  - from store
- launch it and go to 
  - org / gnome / shell / extensions / dash-to-dock
- turn off **show-mounts** option

#### Fix sound issue (fixed in latest ubuntu release)

- create a file /etc/rc.local

  ```bash
  sudo nano /etc/rc.local
  ```

- copy script in file

  ```bash
  #!/bin/bash
  
  hda-verb /dev/snd/hwC0D0 0x20 0x500 0xf
  hda-verb /dev/snd/hwC0D0 0x20 0x477 0x74
  exit 0
  ```

- setup permissions / make it executable

  ```bash
  sudo chmod +x /etc/rc.local
  ```

### Install Softwares:

#### Remove unused softwares (ex: Games) to cleanup a bit

> #### install **Chrome** - No switch to firefox

- download and install

  ```bash
  wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
  sudo apt install ./google-chrome*.deb
  ```

#### Install KeepassXC

```bash
sudo add-apt-repository ppa:phoerious/keepassxc
sudo apt update
sudo apt install keepassxc
```

#### Install DisplayLink drivers

- https://support.displaylink.com/knowledgebase/articles/684649

#### Customize boot screen

- install **Grub Customizer**
- modify order for booting (ex: windows first ...)
- select image etc...

#### Install **Git**

```bash
sudo apt install git
```

#### Install **Github desktop**

- download latest **deb** release from https://github.com/shiftkey/desktop/releases
- install it

```bash
sudo apt install ./GithubDesktop*.deb
```

- Launch and configure

> #### install **MiniConda** - Switch to python and venv

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

```bash
sh Miniconda3-latest-Linux-x86_64.sh
```

- install Jupyter notebook (in base env)

  ```bash
  conda install jupyter
  ```

- install kernels in select env

  ```bash
  conda install ipykernel
  ```

#### Install (update) python

- install specific verison (ex 3.7)

  - download from python Gzipped tarball https://www.python.org/downloads/source/ or 

    ```bash
    wget https://www.python.org/ftp/python/3.7.8/Python-3.7.8.tgz
    ```

  - uncompress

    ```bash
    sudo tar xzf Python-3.7.8.tgz
    ```

  - Install required lib

    ```
    sudo apt-get install build-essential libsqlite3-dev sqlite3 bzip2 libbz2-dev \
    zlib1g-dev libssl-dev openssl libgdbm-dev libgdbm-compat-dev liblzma-dev libreadline-dev \
    libncursesw5-dev libffi-dev uuid-dev
    ```

  - compile (make alt install to keep existing configsudo)

    ```bash
    cd Python-3.7.8
    sudosudo ./configure --enable-optimizations
    sudo make altinstall
    ```
    
  - check

    ```bash
    python3.7 -V
    ```

  - check install path

    ```bash
    which python3
    which python3.7
    ```

- install pip / venv

  ```bash
  sudo apt install python3-pip
  sudo apt install python3-venv
  ```

- Use:

  - python3 for default python3 install
  - python3.7 for specific version
  - **avoid** python: for python 2.7
  - once in a virtual env: use python and pip

- Example

  - python 3 (default version)

    ```bash
    python3 -m venv venv/test3
    source venv/test3/bin/activate
    pip list
    ```

    returns: /usr/bin/python3

  - python 3 (default version)

    ```bash
    python3.7 -m venv venv/test37
    source venv/test37/bin/activate
    pip list
    ```

    returns: /usr/local/bin/python3.7

#### Install **Typora**

```bash
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt update
sudo apt install typora
```

#### Install **Sublime**

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo add-apt-repository "deb https://download.sublimetext.com/ apt/stable/"
sudo apt update
sudo apt install sublime-text
```

#### Install **VSCode**

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg

sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/

sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install code
```

#### Install Docker

- docker

  - install

  ```bash
  sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  sudo apt update
  sudo apt install docker-ce docker-ce-cli containerd.io
  ```

  - check

  ```bash
  sudo systemctl status docker
  ```

- post install / permissions as non root user

  ```bash
  sudo usermod -aG docker ${USER}
  ```

- docker compose (1.28.2) official doc https://docs.docker.com/compose/install/
  ```bash
  sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  
  sudo chmod +x /usr/local/bin/docker-compose
  ```

#### Install DBeaver

```bash
sudo add-apt-repository ppa:serge-rider/dbeaver-ce
sudo apt-get update
sudo apt-get install dbeaver-ce
```

#### Install Gimp

```bash
sudo apt install gimp
```





#### Install Pentaho pdi

- remove old version of java if needed

  ```
  sudo apt remove openjdk-11-jre-headless openjdk-11-jre openjdk-11-jdk-headless openjdk-11-jdk
  ```

- install jdk and jre

  ```
  sudo apt install openjdk-8-jdk
  sudo update-alternatives --config java
  ```

- Download/extract

- launch in pentaho folder

  ```
  sh spoon.sh
  ```

  - tip *"make it executable"*

    > right-click on spoon.sh : properties : permissions : Allow executing file as program
    >
    > in file explorer menu : preferences : behavior : executable text files : run them (or ask what to do)

    





### Shell (byobu + zsh)

##### install byobu / zsh / powerline-fonts

```bash
sudo apt install -y byobu zsh fonts-powerline
```

##### install oh-my-zsh

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

##### change theme 

```bash
nano ~/.zshrc
# change theme line to agnoster
ZSH_THEME="agnoster"
```

##### config byobu

```bash
touch ~/.byobu/.tmux.conf
echo "set -g mouse on" >> ~/.byobu/.tmux.conf
echo "set -g mouse-utf8 on" >> ~/.byobu/.tmux.conf
echo "set -g default-shell /usr/bin/zsh" >> ~/.byobu/.tmux.conf
echo "set -g default-command /usr/bin/zsh" >> ~/.byobu/.tmux.conf
```

##### byobu as default in terminal

- in terminal preferences, create a profile "byobu"
- in command tab, select run a custom command instead of my shell
- custom command: /usr/bin/byobu



