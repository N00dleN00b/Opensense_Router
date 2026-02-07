# OPNsense Router Setup Documentation

## Scenario
The previous network admin rage quit and destroyed the router on the way out, and so we must set up an OPNsense router connecting our team's private LAN to the "WAN", create a VM on our private LAN that successfully gets an IP from DHCP and can reach the internet, and document your network topology and steps taken to set up the network. This guide documents the process of setting up an OPNsense router to bridge a public network and internal WAN to a LAN. 

## Network Architecture

### Network Information
- **Team**: Team 9
- **WAN IP**: 10.31.0.109/24
- **WAN Gateway**: 10.31.0.9
- **LAN Network**: 10.31.9.1/24
- **DHCP Range**: 10.31.9.100 - 10.31.9.199
- **Proxmox Host**: pve402

### Network Topology
- **WAN**: Shared upstream connection (vnet: shared-wan)
- **LAN**: Blue team network (internal company network)

## OPNsense Router VM Creation

### VM Configuration
1. Navigate to Proxmox web interface on pve402
2. Click **Create VM** with the following settings:
   - **VM ID**: 486309XXX (4863 being the class number, and 09 being team 09)
   - **Name**: OpenSense-router09
   - **Resource Pool**: team09
   - **ISO Image**: OPNsense 26.1-dvd (latest version)

3. **Hardware Settings**:
   - **Storage**: SCSI controller (default)
   - **Enable**: Qemu agent
   - **CPU**: 2 cores
   - **RAM**: 2 GB

4. **Network Device 1 (WAN)**:
   - **Bridge**: shared-wan
   - **Model**: VirtIO (paravirtualized)
   - **MTU**: 1
   - **VLAN Tag**: LEAVE EMPTY
   - **Firewall**: Unchecked

5. **Do not** check "Start after created"

6. **Add Network Device 2 (LAN)**:
   - Go to Hardware → Add → Network Device
   - **Bridge**: blue-team-network
   - **Model**: VirtIO (paravirtualized)
   - **MTU**: 1

### Network Interface Order
- **vtnet0** (first MAC address): WAN interface
- **vtnet1** (second MAC address): LAN interface

## OPNsense Installation

### Initial Boot
1. Start the VM
2. Select **Other** as the OS type (OPNsense is not Linux)

### Installation Process
1. **Login credentials**:
   - Username: `installer`
   - Password: `opnsense`

2. **Installation steps**:
   - Press Enter twice to confirm installation
   - Select **Install (ZFS)**
   - Choose **Stripe** for single virtual hard drive
   - Press Space to select both options (the "star" *)
   - Confirm with **Yes** to start installation

3. **Post-installation**:
   - Remove installation media: Hardware → CD/DVD Drive → "Do not use any media"
   - The system will auto-boot into OPNsense
   - **New login credentials**:
     - Username: `root`
     - Password: `opnsense`

## Network Interface Configuration

### Assign Interfaces 
1. Enter option **1** at the menu
2. Configure LAGG: Press Enter (skip)
3. **WAN interface**: vtnet0 (verify the MAC address)
4. **LAN interface**: vtnet1
5. **OPT interfaces**: None
6. Proceed: **Y**

### Configure WAN Interface
1. Enter option **2**
2. Select WAN interface
3. Configure IPv4 via DHCP: **No**
4. **IPv4 Address**: 10.31.0.109
5. **Subnet mask**: 24
6. **Upstream gateway**: 10.31.0.9
7. **DNS server**: 1.1.1.1
8. Configure IPv6 via DHCP: **Y**
9. Revert to HTTP: **N**
10. Generate certificate: **N**
11. Restore web GUI access: **N**

### Configure LAN Interface 
1. Enter option **2** again
2. Select LAN interface
3. Configure IPv4 via DHCP: **No**
4. **IPv4 Address**: 10.31.9.1
5. **Subnet mask**: 24
6. **Upstream gateway**: LEAVE EMPTY
7. **Enable DHCP**: Y
8. **DHCP start address**: 10.31.9.100
9. **DHCP end address**: 10.31.9.199
10. Configure web GUI protocol: **No**
11. Generate self-signed certificate: **Yes**
12. Restore web GUI default access: **Y**

## Testing and Verification

### Create Test VM
1. Created Kali VM
2. **VM Settings**:
   - Resource pool: team09 (pve402) 
   - Connect to: blue-team-network


# Testing & Implementation

# Check IP address assignment
ip a

INSERT IMAGE


# Test gateway connectivity
ping 1.1.1.1


INSERT IMAGE

# Test Web GUI (Firefox)
1. Open Firefox and navigate to: `10.31.9.1`
3. Select "Advanced" and accept security warning 
4. **Login credentials**:
   - Username: `root`
   - Password: `opnsense`

