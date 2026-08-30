# Proxmox `nic0` Network Disconnect Troubleshooting

* Document is AI-Generated from user prompts. Not all steps taken *

## Purpose

This document records the troubleshooting steps taken so far for intermittent network connectivity loss on the Proxmox host.

**Status:** Troubleshooting in progress  
**Current conclusion:** No confirmed solution yet.

---

## 1. Problem Observed

The Proxmox host intermittently loses network/Internet connectivity.

The observed recovery method has been to physically unplug the Ethernet cable from the server and plug it back in.

The Proxmox host uses its motherboard's onboard Ethernet port for the current network connection.

---

## 2. Initial Kernel Log Evidence

The following kernel messages were found around the time of a network disconnect:

```text
Aug 29 17:43:46 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
                               TDH                  <22>
                               TDT                  <5d>
                               next_to_use          <5d>
                               next_to_clean        <21>
                               buffer_info[next_to_clean]:
                               time_stamp           <10403416b>
                               next_to_watch        <22>
                               jiffies              <104034741>
                               next_to_watch.status <0>
                               MAC Status             <40080083>
                               PHY Status             <796d>
                               PHY 1000BASE-T Status  <3800>
                               PHY Extended Status    <3000>
                               PCI Status             <10>

Aug 29 17:43:48 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
```

The same `Detected Hardware Unit Hang` message continued at:

```text
17:43:46
17:43:48
17:43:50
17:43:52
```

### Interpretation

The messages indicate that the Intel Ethernet device associated with `nic0` experienced a hardware/driver transmit hang.

This was treated as the primary lead for the network disconnect investigation.

---

## 3. Identified Network Hardware

The following command was run:

```bash
lspci -nnk | grep -A3 -i ethernet
```

Output:

```text
00:1f.6 Ethernet controller [0200]: Intel Corporation Ethernet Connection (7) I219-V [8086:15bc] (rev 10)
        DeviceName: Onboard - Ethernet
        Subsystem: ASUSTeK Computer Inc. Device [1043:8672]
        Kernel driver in use: e1000e
        Kernel modules: e1000e
```

### Current NIC

| Item | Value |
|---|---|
| Interface | `nic0` |
| Controller | Intel Ethernet Connection (7) I219-V |
| PCI address | `00:1f.6` |
| Driver | `e1000e` |
| Location | Onboard Ethernet |
| Manufacturer/system | ASUS motherboard |

The TP-Link TX201 PCIe network adapter is **not currently installed**.

---

## 4. Confirmed Proxmox Network Configuration

The following file was checked:

```bash
cat /etc/network/interfaces
```

Current configuration:

```text
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.101/24
        gateway 192.168.1.254
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

source /etc/network/interfaces.d/*
```

### Network relationship

```text
nic0
  │
  └── vmbr0
        │
        ├── Proxmox host
        ├── VMs
        └── LXCs
```

Therefore, `vmbr0` is currently dependent on the physical `nic0` interface.

---

## 5. Checked NIC Link Status

The following command was run:

```bash
ethtool nic0
```

Relevant output:

```text
Speed: 1000Mb/s
Duplex: Full
Auto-negotiation: on
Link detected: yes
```

The NIC supports:

```text
10baseT
100baseT
1000baseT
```

The link partner also advertised 1 Gbps operation.

At the time of checking, the physical Ethernet link was functioning normally.

---

## 6. Checked NIC Offloading Features

The following command was run:

```bash
ethtool -k nic0
```

Initially, several offloading features were enabled, including:

```text
rx-checksumming: on
tx-checksumming: on
scatter-gather: on
tcp-segmentation-offload: on
generic-segmentation-offload: on
generic-receive-offload: on
rx-vlan-offload: on
tx-vlan-offload: on
```

These settings were recorded before making troubleshooting changes.

---

## 7. Checked Energy Efficient Ethernet (EEE)

The following command was run:

```bash
ethtool --show-eee nic0
```

Output included:

```text
EEE settings for nic0:
        EEE status: enabled - inactive
        Tx LPI: 17 (us)
        Supported EEE link modes: 100baseT/Full
                                   1000baseT/Full
        Advertised EEE link modes: 100baseT/Full
                                   1000baseT/Full
        Link partner advertised EEE link modes: Not reported
```

EEE was identified as one possible area to test.

---

## 8. EEE Was Disabled

The following command was used:

```bash
ethtool --set-eee nic0 eee off
```

EEE was then checked again.

### Result

Disabling EEE **did not resolve the intermittent network disconnect problem**.

Therefore, EEE is currently not considered a confirmed solution.

---

## 9. Kernel Logs Were Narrowed to the Failure Window

The following command was used:

```bash
journalctl -k --since "17:35" --until "17:50" | grep -Ei "nic0|e1000e|link|network"
```

The relevant results were:

```text
Aug 29 17:43:46 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
Aug 29 17:43:48 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
Aug 29 17:43:50 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
Aug 29 17:43:52 pve1 kernel: e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang:
```

This reinforced that the `e1000e`/`nic0` hardware hang occurs during the connectivity issue.

---

## 10. Current Troubleshooting Change: NIC Offloading Disabled

After EEE did not resolve the problem, a broader NIC offloading test was performed.

The following command was run:

```bash
ethtool -K nic0 gso off tso off rxvlan off txvlan off gro off tx off rx off sg off
```

This temporarily disables several NIC offloading features, including:

- GSO
- TSO
- GRO
- RX VLAN offloading
- TX VLAN offloading
- RX checksum offloading
- TX checksum offloading
- Scatter-gather

### Current status

This change is currently being used as a **troubleshooting test**, not a confirmed permanent configuration.

The server should be monitored to determine whether another:

```text
e1000e ... nic0: Detected Hardware Unit Hang
```

occurs.

---

## 11. Current Configuration Status

At this point, the troubleshooting state is:

```text
Proxmox
   │
   └── vmbr0
         │
         └── nic0
               │
               └── Intel I219-V
                     │
                     └── e1000e driver
```

Current tests:

| Test | Result |
|---|---|
| Check kernel logs | Hardware hang confirmed |
| Identify NIC | Intel I219-V |
| Check link status | 1 Gbps / Full Duplex / Link detected |
| Check EEE | Enabled initially |
| Disable EEE | Did not resolve problem |
| Disable broad NIC offloading | Currently being tested |
| TP-Link TX201 | Not installed |

---

## 12. Current Status — No Confirmed Solution

**This troubleshooting process is not complete.**

The current offloading configuration is only being tested to determine whether NIC hardware offloading is related to the `e1000e` hardware hang.

No permanent solution has been selected yet.

If the hardware hang continues, future investigation may include:

1. Checking the installed Proxmox/kernel version.
2. Checking the `e1000e` driver version.
3. Checking BIOS/UEFI and motherboard firmware.
4. Testing a different Ethernet cable.
5. Testing a different switch port.
6. Investigating Intel I219-V/e1000e-specific issues.
7. Installing and testing the TP-Link TX201 as an alternative physical NIC.
8. Comparing stability between the onboard Intel NIC and the TX201.

---

## Useful Commands

### Check NIC hardware

```bash
lspci -nnk | grep -A3 -i ethernet
```

### Check interface status

```bash
ip -br link
```

### Check NIC information

```bash
ethtool nic0
```

### Check driver

```bash
ethtool -i nic0
```

### Check NIC features

```bash
ethtool -k nic0
```

### Check EEE

```bash
ethtool --show-eee nic0
```

### Monitor relevant kernel messages

```bash
journalctl -k -f | grep -Ei "e1000e|nic0|hang|link"
```

### Search current boot for NIC errors

```bash
journalctl -k | grep -Ei "e1000e|nic0|hang|link"
```

### Check Proxmox version

```bash
pveversion -v
```
