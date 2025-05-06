## **Brief Explanation**

![cano2](https://github.com/user-attachments/assets/d9cda332-e786-4a54-a363-c235bf172981)

### **What is CANO?**

CANO, or Close Access Network Operations, is a method for gaining access to closed wireless networks by exploiting the WPA2 authentication process. It involves using a Raspberry Pi equipped with two Alpha Wi-Fi cards, one to monitor and another to transmit.

Raspberry Pi connects to a server via a cellular modem using WireGuard VPN, allowing an operator to manage the device. CANO encompasses both the technical tools deployed on the Raspberry Pi and the setup used by the end user to control and interact with it.

# PI Setup and Tool Guide

## Contents

1. [Pi Setup](#Pi-Setup)
2. [Drivers for Alpha Cards](#Set-Drivers-for-Alpha-Cards)
3. [Kismet Setup](#Install-Kismet)
4. [Wireguard](#Setting-Up-Wireguard)
5. [QMI Setup](#QMI-Setup)
6. [PPP Setup](#PPP-Setup)
7. [MAC Changer](#MAC-Changer)

## Pi Setup

### Image the Raspberry Pi

1. **Prepare the SD Card**:
    - Insert the microSD card into the SD adapter and plug it into your computer
    - Open the Raspberry Pi Imager application
    - Select Raspberry Pi OS Bullseye 64-bit and write to the SD card
    - Enable SSH with credentials `pi::password`

2. **Insert the SD Card and Power Up**:
    - Insert the flashed SD card into the Raspberry Pi
    - Connect the Raspberry Pi to the GL.iNet router via Ethernet
    - Power on the Raspberry Pi

### Configure the Raspberry Pi

1. **SSH into the Raspberry Pi**:
    - Use the IP address found from the router:
      <div style="margin-top: 20px;">
     
      ```bash
      ssh pi@<raspberry_pi_ip>
      ```
    -  Credentials: `pi::password`

2. **Run Initial Configuration**:
    - Run `sudo apt update && sudo apt upgrade -y`
    - Run `sudo raspi-config` and make the following changes:
        - Change hostname to `<AGREED UPON HOST NAME>`.
        - Enable SSH.
        - Set locale to `en_US.UTF-8`.
        - Set timezone to local.
        - Set WLAN country to local.
        - Expand the filesystem.
        - Turn ONNNN predictable network interface names.
    - Reboot the Raspberry Pi.

4. **Set Up SSH Keys**:
    - Generate SSH keys on your laptop:
      <div style="margin-top: 20px;">
       
      ```bash
      ssh-keygen -b 4096
      ssh-copy-id pi@<raspberry_pi_ip>
      ```
    - Test SSH login without a password:
      <div style="margin-top: 20px;">
       
      ```bash
      ssh pi@<raspberry_pi_ip>
      ```

5. **Harden SSH Configuration**:
    - Modify `sshd_config`:
      <div style="margin-top: 20px;">
       
      ```bash
      sudo nano /etc/ssh/sshd_config
      ```
    - Set the following:
      <div style="margin-top: 20px;">
       
      ```plaintext
      AddressFamily inet
      PermitRootLogin no
      PasswordAuthentication no
      ```
    - Restart the SSH daemon:
      <div style="margin-top: 20px;">
       
      ```bash
      sudo systemctl restart sshd
      ```


## Set Drivers for Alpha Cards

#### Make sure kernel headers are up-to-date

```bash
sudo apt-get install raspberrypi-kernel-headers
sudo reboot
```

#### Copy and Paste the Command

```bash
git clone https://github.com/lwfinger/rtw88.git
cd rtw88
make
sudo make install
```

## Install Kismet

#### Copy and Paste the Command

###### Bullseye

```bash
wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key --quiet | gpg --dearmor | sudo tee /usr/share/keyrings/kismet-archive-keyring.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/kismet-archive-keyring.gpg] https://www.kismetwireless.net/repos/apt/release/bullseye bullseye main' | sudo tee /etc/apt/sources.list.d/kismet.list >/dev/null
sudo apt update
sudo apt install kismet
```

###### Bookworm

```bash
wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key --quiet | gpg --dearmor | sudo tee /usr/share/keyrings/kismet-archive-keyring.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/kismet-archive-keyring.gpg] https://www.kismetwireless.net/repos/apt/release/bookworm bookworm main' | sudo tee /etc/apt/sources.list.d/kismet.list >/dev/null
sudo apt update
sudo apt install kismet
```

## Setting Up WireGuard

### Configure WireGuard on AWS

1. **SSH into the AWS server**:
   - SSH:
     <div style="margin-top: 20px;">
      
     ```bash
     ssh -i ~/.ssh/AWS8.pem ubuntu@<aws_server_ip>
     ```
2. **Install WireGuard**:
   - Install:
     <div style="margin-top: 20px;">
      
     ```bash
     sudo apt install wireguard jq resolvconf
     # if this command errors, make sure the aws has been updated
     # sudo apt update && sudo apt upgrade -y
     ```

3. **Download and Run Manager Script**:
   - Download config script for WireGuard:
     <div style="margin-top: 20px;">
     
     ```bash
     wget https://raw.githubusercontent.com/complexorganizations/wireguard-manager/main/wireguard-manager.sh
     sudo mv wireguard-manager.sh /etc/wireguard/wireguard-manager.sh
     ```

   - Run the script :
     <div style="margin-top: 20px;">
      
     ```bash
     sudo bash /etc/wireguard/wireguard-manager.sh
     ```
     - Note: Press 1 for everything except the name
     - You WILL need to add a custom rule on the AWS to route the wireguard traffic ```UDP 51820```

### Configure WireGuard for Peers

1. **On AWS create Wireguard peer**:
   - **Run the Manager Script**:
     <div style="margin-top: 20px;">

     ```bash
     # create a client config if you havennt already
     sudo bash /etc/wireguard/wireguard-manager.sh
     ```
     - Note: Option 5

   - **Chown file and move to home**
     <div style="margin-top: 20px;">

     ```bash
     cp /etc/wireguard/clients/<generated_conf> ~
     chown <current_use> <generated_conf>
     ```

2. **Set up WireGuard on peer**:
   - Install WireGuard on Peer:
     <div style="margin-top: 20px;">
     
     ```bash
     sudo apt install wireguard jq resolvconf
     ```

   - Copy config to WireGuard install directory:
     <div style="margin-top: 20px;">
     
     ```bash
     sudo scp <local>@<config file> pi@ip
     mv <generated_conf> /etc/wireguard/<generated_copy>
     ```

   - Run Command to use file:
     <div style="margin-top: 20px;">
     
     ```bash
     sudo wg-quick up /etc/wireguard/<conf_name.conf>

     # bring config down
     sudo wg-quick down /etc/wireguard/<conf_name.conf>
     ```

   - You can edit your .bashrc file to quick up and down quickly
     <div style="margin-top: 20px;">

     ```bash
     # you can write it anywhere in your .bashrc
     alias wgup='wg-quick up /etc/wireguard/<conf_name.conf>'
     alias wgdown='wg-quick down /etc/wireguard/<conf_name.conf>'
     ```

   - Make sure you source the file
     <div style="margin-top: 20px;">
     
     ```bash
     source ~/.bashrc
     # or
     . ~/.bashrc
     ```

## QMI Setup

#### Before running, make sure pi is updated
```bash
sudo apt update && sudo apt upgrade -y
```
#### Download this script, run it to install QMI tools
```bash
# BEFORE RUNINNING THE SCRIPT: make sure modem cable is disconnected

wget https://raw.githubusercontent.com/sixfab/Sixfab_QMI_Installer/main/qmi_install.sh
sudo chmod +x qmi_install.sh
sudo ./qmi_install.sh
```
###### You will need to reboot the pi after install
#### Make sure the HAT cable is plugged in, go to installed files directory
```bash
# change to that directory
cd /opt/qmi_files/quectel-CM

# connect to the internet
sudo ./quectel-CM -s <apn>

# NOTE: the command doesnt "end" so keep it up to keep the connection.

# TROUBLESHOOT
ifconfig wwan0
ping -I <ribbon_antenna_interface> -c 5 8.8.8.8
```
#### We need this to reconnect automatically on reboot
```bash
# download auto setup script
wget https://raw.githubusercontent.com/sixfab/Sixfab_QMI_Installer/main/install_auto_connect.sh

# run it
sudo chmod +x install_auto_connect.sh
sudo ./install_auto_connect.sh
# type APN
```
#### Check to see if service is running
```bash
sudo systemctl status qmi_reconnect.service


# TROUBLESHOOT:

# chown the file if needed at /etc/systemd/system/qmi_reconnect.service

# start service
stemctl start qmi_reconnect.service

# stop service
stemctl stop qmi_reconnect.service

# restart
sudo systemctl restart qmi_reconnect.service

```
#### If you want to disable from starting up
```bash 
sudo systemctl disable qmi_reconnect.service

# OTHER TROUBLESHOOT 
# https://docs.sixfab.com/page/qmi-interface-internet-connection-setup-using-sixfab-shield-hat
# https://docs.sixfab.com/page/setting-up-a-data-connection-over-qmi-interface-using-libqmi
```
## PPP Setup

#### Before running, make sure the pi is up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

#### Download the PPP installer:
```bash
git clone https://github.com/sixfab/Sixfab_PPP_Installer.git
cd SixFab*
```

#### Run the installer
```bash
sudo bash ./ppp_install_standalone.sh

###
#
# Choose the following
#
# 1. Option 6: 3G/4G 
# 2. <your_apn>
# 3. No username/password
# 4. ttyUSB3
# 5. Yes to autoconnect
#
###
```

###### Ensure antennas are attached

#### Reboot the pi, if not already
```bash
sudo reboot
```

#### Check to see new interfaces
```bash
ip -br a
# or
iwconfig
```
#### If wireguard is setup correctly, you are in the network with the pi

## MAC Changer

#### Install macchanger
```bash
sudo apt install macchanger
# Select no on popup
```
#### Bring the interface down before attempting to change MAC
```bash
sudo ifconfig <interface> down
```
#### To change to a specific MAC, use ```-m``` option
```bash
sudo macchanger -m <mac> <interface>
```
#### If you want a random MAC, use ```-r``` option
```bash
sudo macchanger -r <interface>
```
#### You can also change to vendor specific, use ```-l``` to list all vendors
```bash
macchanger -l
```
#### Or to list from a specific vendor, use ```--list=<VENODR>```
```bash
macchanger --list=<vendor>
```
