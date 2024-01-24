# Portable Kali for Raspberry Pi

## Installation Steps

### 1. Install Kali Linux
- Begin by installing Kali Linux on your Raspberry Pi's SD card using the raspi-imager tool available from the official Raspberry Pi website.

### 2. Configuration Files
- After flashing the SD card, connect it to your computer.
- Create the following files in the "boot" directory of the SD card:
    ```
    touch ssh
    touch wpa_supplicant.conf
    code -n wpa_supplicant.conf
    ```
- In the "wpa_supplicant.conf" file, add the following lines, replacing "SSID" and "password" with your Wi-Fi credentials:
    ```
    country=US
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    ap_scan=1

    update_config=1
    network={
        ssid="SSID"
        psk="password"
        priority=1
    }

    network={
        ssid="SSID2"
        psk="password2"
        priority=2 # Higher numbers indicate higher priority
    }
    ```

### 3. Initial Setup
- Insert the SD card into your Raspberry Pi and power it on.
- Use a keyboard, HDMI cable, and monitor to log in.
- Log in using the following default credentials:
    - Username: `kali`
    - Password: `kali`
- To change the default password, run the command `passwd` and follow the prompts on the terminal.

## Setting Up `vncserver`

### 1. Update and Install
- After logging in as the `kali` user, execute the following commands:
    ```bash
    sudo apt update -y
    sudo apt upgrade -y
    sudo apt install autocutsel
    ```

### 2. Configure Autologin
- Modify the "lightdm.conf" file as follows:
    ```bash
    sudo nano /etc/lightdm/lightdm.conf
    ```
    Make the following changes and save the file:
    ```bash
    autologin-user=kali
    autologin-user-timeout=0
    ```

### 3. Set VNC Password
- Set a password for your VNC server with this command:
    ```bash
    vncpasswd
    ```
    Follow the on-screen instructions. If you don't specify a file location, it will be stored in `~/.vnc/passwd`.

### 4. Configure `xstartup`
- Create the `xstartup` file:
    ```bash
    sudo touch ~/.vnc/xstartup
    sudo nano ~/.vnc/xstartup
    ```
    Add the following lines to the file:
    ```bash
    #!/bin/sh
    unset SESSION_MANAGER
    unset DBUS_SESSION_BUS_ADDRESS
    startxfce4 &
    ```
    The following part helps with the grey screen - [Link](https://forums.raspberrypi.com/viewtopic.php?t=216725)
    ```bash
    #!/bin/sh unset SESSION_MANAGER 
    unset DBUS_SESSION_BUS_ADDRESS 
    startxfce4 & 
    [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup 
    [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources 
    xsetroot -solid grey 
    vncconfig -iconic &
    ```

    Save and exit, then change the file's permissions:
    ```bash
    chmod +x ~/.vnc/xstartup
    ```

### 5. Reboot
- Reboot your Raspberry Pi:
    ```bash
    sudo reboot
    ```

### 6. Start VNC Server
- After the reboot, run the following command to start the VNC server:
    ```bash
    vncserver
    ```

### 7. Verify VNC Server Port
- Ensure that the VNC server is running on the default port `5901`:
    ```bash
    netstat -tuna
    ```

### 8. Connect via VNC
- Use a VNC viewer client on another machine to connect to the Raspberry Pi's VNC server using the password you set up earlier with `vncpasswd`.

### 9. VNCServer `systemctl`` service 
- Create `vncserver.service` file for the `systemctl` to start at the startup.
    ```bash
    cd /etc/systemd/system
    sudo touch vncserver.service
    sudo nano vncserver.service
    ```
- Add following lines to the file
    ```bash
    [Unit]
    Description=VNC remote desktop server
    After=sshd.service

    [Service]
    Type=dbus
    ExecStart=/usr/bin/vncserver
    User=kali
    Type=forking

    [Install]
    WantedBy=multi-user.target
    ```
- Enable the VNCserver service to start at the startup
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl start vncserver
    sudo systemctl enable vncserver 
    ```

## Creating a Wi-Fi Access Point

### 1. Requirements
- To set up a Wi-Fi access point, you'll need a USB Wi-Fi adapter.

### 2. Install the USB Wi-Fi adapter driver
- `lsusb` - to identify the driver and the chipset type.
    ```bash
    Bus 001 Device 003: ID 2357:012d TP-Link Archer T3U [Realtek RTL8812BU]
    ```
- Following instructions are specifically for the TP-Link AC1300 ([Archer AC1300 Mini Wireless MU-MIMO USB Adapter](https://www.tp-link.com/us/home-networking/usb-adapter/archer-t3u/))  USB Wi-Fi adapter
    ```bash
    sudo apt update
    which dkms || sudo apt install dkms
    git clone https://github.com/morrownr/88x2bu-20210702.git
    cd 88x2bu-20210702
    sudo ./install-driver.sh
    ```
-  Check driver existance
    ```bash
    find /lib/modules/`uname -r`/ -name "88x2bu.ko"
    ```
- Follow the instructions for kali linux on following links -
    - [https://docs.alfa.com.tw/Support/Linux/RTL8812BU/](https://docs.alfa.com.tw/Support/Linux/RTL8812BU/)
    - [https://github.com/morrownr/88x2bu-20210702](https://github.com/morrownr/88x2bu-20210702)
    - [https://www.manuel-bauer.net/blog/install-driver-for-rtl8812bu-wifi-dongle-on-a-raspberry-pi](https://www.manuel-bauer.net/blog/install-driver-for-rtl8812bu-wifi-dongle-on-a-raspberry-pi)

- Warning: If you decide to do a distro upgrade, which will likely install a new version of kernel such as 5.15 to 6.1, you need to update this driver with the newest available code and then run the removal script before performing the disto upgrade. Use the following commands in the driver directory:Before the kernel update, you need to update this driver.
    ```bash
    git pull
    ```
     Note: Do not reboot before running the below command so that the driver stays active until after your distro upgrade is complete.
    ```bash
    sudo sh remove-driver.sh
    ```
    Note: The following command will reinstall the updated driver after you are finished with the distro upgrade and reboot.
    ```bash
    sudo ./install-driver.sh
    ```

### 3. Update and Install
- Run the following commands on your Raspberry Pi to update and install the necessary software:
    ```bash
    sudo apt update -y
    sudo apt upgrade -y
    sudo apt install hostapd dnsmasq
    ```

### 4. Configure `hostapd`
- Modify the "hostapd.conf" file as follows:
    ```bash
    sudo nano /etc/hostapd/hostapd.conf
    ```
    Add the following lines to it, ensuring you set your preferred Wi-Fi SSID and password:
    ```bash
    # Use the name of your secondary WiFi adapter
    interface=wlan1
    # Set your desired hidden SSID
    ssid=YOURSSID
    hw_mode=g
    # You can change this to any channel you prefer
    channel=11
    macaddr_acl=0
    auth_algs=1
    # Change this option to `1` if you want a hidden Wi-Fi
    ignore_broadcast_ssid=0
    wpa=2
    # Set your desired password
    wpa_passphrase=YOUR_PASSWORD
    wpa_key_mgmt=WPA-PSK
    wpa_pairwise=CCMP
    rsn_pairwise=CCMP
    ```
    Save and close the file, then apply the changes:
    ```bash
    sudo hostapd /etc/hostapd/hostapd.conf
    ```

### 5. Configure DHCP
- Modify the "dnsmasq.conf" file:
    ```bash
    sudo nano /etc/dnsmasq.conf
    ```
    Add the following lines to it:
    ```bash
    interface=wlan1
    dhcp-range=192.168.1.2,192.168.1.20,255.255.255.0,24h
    dhcp-option=3,192.168.1.1
    dhcp-option=6,192.168.1.1
    server=8.8.8.8
    log-queries
    log-dhcp
    listen-address=127.0.0.1
    ```

### 6. Enable IP Forwarding
- Modify the "sysctl.conf" file:
    ```bash
    sudo nano /etc/sysctl.conf
    ```
    Update the file to include the following line:
    ```bash
    net.ipv4.ip_forward=1
    ```

### 7. Modify Network Interfaces
- Modify the "interfaces" file:
    ```bash
    sudo nano /etc/network/interfaces
    ```
    Update it as follows:
    ```bash
    allow-hotplug wlan1
    iface wlan1 inet static
        address 192.168.1.1
        netmask 255.255.255.0
    ```

### 8. Configure IPTABLES
- Enable IP masquerading for routing:
    ```bash
    sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
    ```
    Confirm that the changes are applied:
    ```bash
    sudo iptables -t nat -L
    ```
    Save the changes to a file:
    ```bash
    sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
    ```

### 9. Make IPTABLES Changes Persistent
- To ensure that the IPTABLES changes persist after a reboot, create a script:
    ```bash
    sudo nano /etc/network/if-pre-up.d/iptables
    ```
    Modify the file as follows:
    ```bash
    /usr/sbin/iptables-restore < /etc/iptables.ipv4.nat
    ```
    Change the permissions of the script:
    ```bash
    sudo chmod +x /etc/network/if-pre-up.d/iptables
    ```

### 10. Start Services
- Stop all running services, restart them, and enable them:
    ```bash
    sudo service hostapd stop
    sudo systemctl unmask hostapd
    sudo systemctl daemon-reload
    sudo systemctl start hostapd
    sudo service dnsmasq start
    sudo service hostapd start
    sudo systemctl enable dnsmasq
    sudo systemctl enable hostapd
    ```

### 11. Reboot
- Reboot your Raspberry Pi:
    ```bash
    sudo reboot
    ```

### 12. Connect to Your Wi-Fi Access Point
- After the reboot, you should see your newly created Wi-Fi access point, named "YOURSSID." Connect to it using the password you set, "YOUR_PASSWORD."

## Portable Kali Device
- To ensure your Raspberry Pi 4 has sufficient power, use a power bank that can provide a minimum of 5V and 3A.
- Connect to the VNC server using the IP address specified in the "interfaces" file, which is `192.168.1.1:5901`.
- Consider 3D printing a caddy-like structure to securely hold your Raspberry Pi, power bank, and antenna, making it easily accessible in your backpack for on-the-go use.

- Download the STL file here: [Portable Hacker Box - Kali Raspberry Pi 4 Caddy](https://www.printables.com/model/573492-portable-hacker-box-kali-raspberry-pi-4-caddy).

Enjoy using your portable Kali device!