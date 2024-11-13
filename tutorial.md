![OpenWrt Logo](https://upload.wikimedia.org/wikipedia/commons/8/84/OpenWrt_Logo.svg)

# Step 1: Add OS to CT Templates

1. Get the OpenWrt RootFS Template
   - Visit the [OpenWrt repository](https://images.linuxcontainers.org/images/openwrt/23.05/amd64/default/20241109_11%3A57/) for the RootFS file. 
Copy the link to the rootfs.tar.xz file.

2. Download Template in Proxmox
   - In Proxmox, go to your Node > Local > CT Templates.
   - Click Download from URL, paste the URL, and confirm.

3. Rename and Verify the Template
   - Rename the template to openwrt.
   - Add a checksum for security. Use SHA-256:
     
        ```bash
        sha256sum openwrt.tar.xz
        ```
   - Go back to the OpenWrt repository, find the checksum, and compare it with the result.

4. Confirm Download Completion
   - Once downloaded, confirm the template appears in your CT templates.

# Step 2: Install OpenWrt in a Container (LXC)

1. Create the LXC Container
   - Go to Proxmox, choose Create CT, and fill in the following settings:

        ```bash
        VM ID: 202
        OS Template: Select the OpenWrt template from CT templates.
        Architecture: amd64
        Hostname: openwrt
        Root Disk: 20GB
        Memory: 1GB
        CPU Cores: 2
        OS Type: unmanaged
        Privileged: No
        ```

2. Boot the Container
   - Open your node’s shell and start the container. Confirm it boots up and loads without issues.

# Step 3: Set Up Network Interfaces

1. Add Network Interfaces
   - In the Proxmox container settings, go to Network.
   - Add two interfaces:
      - eth0: Configure for DHCP or set a static IP.
      - eth1: Configure as needed for LAN or additional networking.

2. Configure Network on OpenWrt
   - Go to the console of the OpenWrt container.
   - Edit the network configuration file:
        ```bash
        vim /etc/config/network
        ```
   - Set up the LAN bridge configuration:
        ```bash
        config interface 'lan'
            option type 'bridge'
            option ifname 'eth0'
            option proto 'static'
            option netmask '255.255.255.0'
            option ipaddr '192.168.1.126'  # Replace with your LXC's IP
        ```

3. Save and Reboot the Container
   - After editing, save the file:
        ```bash
        :wq
        ```
   - Go to Proxmox, select your OpenWrt container, and click Reboot to apply changes.
  
# Step 4: Access the OpenWrt Web UI

1. Open the Web UI
   - After the container restarts, open a browser and navigate to:
   ```
   http://<Your-LXC-IP> (e.g., http://192.168.1.126).
   ```
   - If you see a security warning due to a self-signed certificate, proceed with caution.

2. Login to OpenWrt
   - You should now see the OpenWrt login page. Log in and start configuring OpenWrt as desired.

