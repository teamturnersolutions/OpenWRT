![OpenWrt Logo](https://upload.wikimedia.org/wikipedia/commons/8/84/OpenWrt_Logo.svg)

### **OpenWRT in a Proxmox Container

---

#### **1. Basic Architecture Overview**
- **OpenWRT as a Router in a CT**:
  - OpenWRT running in a Proxmox container can act as a virtual router for your network.
  - It can handle **DHCP, NAT, VLANs, VPNs, and firewall rules**, just like a physical router.
  - The container leverages the Proxmox host’s virtualized networking.

- **Proxmox Role**:
  - Proxmox provides the virtualized hardware and network interfaces that OpenWRT uses.
  - The Proxmox host itself does not manage routing unless explicitly configured.

---

#### **2. Network Interfaces in OpenWRT**
- **Bridged Mode**:
  - Attach the OpenWRT container to a Proxmox bridge (e.g., `vmbr0`) to connect it to the physical network.
  - This is essential if OpenWRT needs to route traffic between the LAN and the WAN.

- **VLAN Support**:
  - OpenWRT fully supports VLAN tagging (`802.1q`), which is critical for segmenting networks.
  - You can create virtual interfaces in OpenWRT for each VLAN.

- **MACVLAN Passthrough** (Optional):
  - If you want the container to bypass the host and directly communicate with the network, you can use MACVLAN.
  - This avoids NAT at the Proxmox host level.

---

#### **3. Host and CT Communication**
- By default, Proxmox and OpenWRT communicate over the bridge (`vmbrX`) if attached.
- **Routing within the host**:
  - If OpenWRT is the main router, it will handle all traffic, including traffic originating from the Proxmox host.
  - Static routes on the Proxmox host might be required for proper inter-subnet communication.

---

#### **4. Container Configuration in Proxmox**
- **Container Privileges**:
  - Use **privileged containers** if you need full hardware access (e.g., for VLANs or Wi-Fi passthrough).
  - Unprivileged containers work for most routing use cases but may have limitations with direct hardware access.

- **Network Interface Assignment**:
  - Assign virtual NICs to the OpenWRT CT corresponding to the WAN and LAN.
  - Example: `eth0` for WAN and `eth1` for LAN.

- **Resource Allocation**:
  - Allocate sufficient CPU and memory to the container, especially if handling VPNs or multiple VLANs. Example: 1-2 cores and 512 MB RAM minimum.

---

#### **5. Routing and NAT**
- **WAN Connection**:
  - The OpenWRT CT can connect to the internet via the Proxmox host’s WAN bridge.
  - The WAN interface in OpenWRT should be configured to obtain an IP dynamically or use a static IP if necessary.

- **LAN Configuration**:
  - Configure a LAN interface to serve IP addresses via DHCP to clients.
  - OpenWRT can also manage multiple subnets.

- **NAT and Port Forwarding**:
  - NAT is enabled by default in OpenWRT. Port forwarding rules can be created to allow external access to services behind the router.

---

#### **6. Firewall and Security**
- **Firewall Rules**:
  - OpenWRT uses `iptables` or `nftables` to manage traffic.
  - Ensure rules are configured to allow Proxmox host-to-OpenWRT communication if required.

- **CT Security**:
  - Avoid exposing privileged containers to the internet without hardening.
  - Consider using unprivileged containers with controlled access.

- **Sysctl Settings**:
  - Enable IP forwarding in the container if acting as a router:
    ```sh
    echo 1 > /proc/sys/net/ipv4/ip_forward
    ```

---

#### **7. Performance Considerations**
- **CT Overhead**:
  - Containers are lightweight compared to VMs, so they can route traffic efficiently without significant resource demands.

- **Bandwidth Handling**:
  - OpenWRT in a CT can handle gigabit speeds as long as the Proxmox node has sufficient resources.

- **VPN Performance**:
  - OpenWRT in a CT may handle VPN tasks (e.g., WireGuard or OpenVPN) well, but encryption may be CPU-intensive. Allocate additional cores if needed.

---

#### **8. VLAN Management**
- Configure VLANs on the Proxmox host’s bridge and pass them to the OpenWRT container.
  - Example Proxmox `/etc/network/interfaces` entry:
    ```plaintext
    auto vmbr0
    iface vmbr0 inet static
        address 192.168.1.2/24
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
    ```
  - Use OpenWRT’s web interface (`LuCI`) or CLI to manage VLANs.

---

#### **9. Failover and Redundancy**
- **High Availability (HA)**:
  - OpenWRT in a CT can be part of an HA setup if Proxmox is configured for HA.
- **Backups**:
  - Regularly back up the OpenWRT container to ensure quick recovery.

---

#### **10. Troubleshooting**
- **No Internet on OpenWRT**:
  - Check the Proxmox bridge configuration.
  - Ensure the WAN interface in OpenWRT is properly configured.

- **LAN Devices Not Getting IPs**:
  - Verify DHCP is enabled on the LAN interface in OpenWRT.

- **Proxmox Host Communication**:
  - Add static routes or use a management interface for host-to-OpenWRT communication.


### WiFi in OpenWRT

1. **Check Wi-Fi on the host**:
   - Ensure that Wi-Fi is enabled on your host device.
   - If you are using Proxmox, confirm that the Wi-Fi interface is functional and identified by the host (e.g., `iwconfig` or `nmcli`).

2. **Passthrough Wi-Fi to the container**:
   - If your OpenWRT container needs to manage Wi-Fi, you'll need to expose the Wi-Fi interface to the container. 
   - **For privileged containers**:
     - Use `lxc.mount.entry` in your Proxmox container's configuration file to bind the Wi-Fi interface.
     - Alternatively, add the Wi-Fi interface to the container's network configuration.
   - **For unprivileged containers**:
     - You may need to switch to a privileged container for Wi-Fi passthrough, as access to hardware devices is generally restricted in unprivileged containers.

3. **Manually create the `wireless` config file**:
   - If the Wi-Fi hardware is present and detected but the config file is still missing, you can manually create `/etc/config/wireless` and populate it based on OpenWRT's defaults or examples from the documentation.

4. **Restart OpenWRT services**:
   - After making changes, restart the OpenWRT network service:
     ```sh
     /etc/init.d/network restart
     ```

---

### To Check Wi-Fi Detection in the Container:
Run the following command in the OpenWRT container:
```sh
iw dev
```
If no Wi-Fi interface is listed, it confirms that the container does not have access to the Wi-Fi hardware.
