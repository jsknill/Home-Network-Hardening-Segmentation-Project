**Residential Network Hardening & Advanced Segmentation**

**Hardware: TP-Link Archer BE3600 (Wi-Fi 7)**

**Objective:** Transition from a "flat" unmanaged network on an Apple Wi-fi 5 device, to a hardened, multi-segment architecture to minimize the attack surface and isolate high-risk IoT/Legacy assets.

📋 Project Overview
This project documents the end-to-end security hardening of a residential gateway. By leveraging Wi-Fi 7 management features, I implemented **VLAN-style Network Segmentation, Static DHCP Binding,** and **Layer 2/3 Isolation** to shield primary data-processing units from vulnerable IoT peripherals.

**Note:** All IP addresses shown (192.168.0.x) pertain to a private internal network and are used exclusively for architectural demonstration.

**🏗️ Network Architecture**
The environment is divided into three logical segments to enforce the **Principle of Least Privilege:**

**Main Network:** High-trust 5GHz/6GHz segment for primary workstations and authorized mobile devices.

**Home IoT:** Untrusted 2.4GHz segment for "smart" hardware, cameras, and legacy peripherals.

**Guest Network:** Temporary, 100% isolated segment for external visitors with zero internal resource access.

**🛠️ Implementation Steps**
**Phase 1: Gateway & Protocol Hardening**
**Management Plane Migration:** Shifted from cloud-dependent (TP-Link ID) management to **Local-Only Admin access.** This reduces the external attack surface and eliminates vendor-cloud dependencies.

**Cryptographic Standards:** Deployed **WPA3/WPA2 Mixed Mode,** ensuring modern clients utilize the highest encryption standards while maintaining compatibility for legacy hardware (e.g., Alexa, Xbox 360).

**Phase 2: Asset Discovery & Identity Management**
**OUI Analysis & "Pull-Testing":** Conducted a physical inventory to resolve mystery MAC addresses. Identified a dual-interface TP-Link T2UB Nano (Bluetooth/Wi-Fi) through targeted disconnection.

**MAC-to-IP Binding:** Implemented **DHCP Address Reservations** to prevent "IP Drifting" and ensure security policies remain persistently bound to specific hardware IDs.

**Traffic Analysis & Telemetry (The "Free-Tier" Strategy):** To maintain granular network visibility without recurring subscription costs, I implemented a decentralized monitoring stack:

**Active Scanning (Fing Desktop):** Deployed for **Layer 2/3 device discovery** and vulnerability assessment. This allowed for manual port-scanning and connectivity auditing to verify that internal firewalls are dropping unauthorized inter-segment traffic.

**Host-Layer Visibility:** Deployed **Portmaster** and **GlassWire** on the primary workstation for application-layer auditing.

**Audit Verification:** Used **GlassWire’s usage breakdown** to confirm that traffic to the HP Envy 5540 remains strictly within the LAN/WLAN boundary, verifying that the "Bedtime" WAN-block is active.

**Phase 3: Attack Surface Reduction (The "Caging" Strategy)**
**IoT & Legacy Isolation:** Leveraged Device Isolation to "fence off" Nest Cameras, Ring Doorbells, and legacy consoles (Xbox 360/3DS).

**Printer "Local-Only" Pivot:** To resolve discovery issues (mDNS/WSD) caused by strict Layer 2 isolation, the HP Envy 5540 was migrated to the Main segment with Outbound WAN Filtering.

**The "Bedtime" Kill-Switch:** Utilized Parental Control Access Schedules to impose a permanent "Bedtime" on the printer. This enforces a total WAN blackout—preventing unauthorized "phone-home" telemetry—while maintaining full LAN availability for trusted print services.

**Phase 4: AP Hardening & Multicast Control**
**IGMP Snooping & Proxy:** Optimized multicast delivery to ensure discovery packets are only routed to authorized subscribers.

**Intra-BSS Traffic Blocking:** Enabled AP Isolation on the IoT segment to prohibit "East-West" communication, ensuring a compromised smart bulb cannot interact with neighboring IoT assets.

**Persistence & Stability (Address Reservations)**
To ensure security rules and logging remain consistent, I implemented DHCP Address Reservations. This prevents "IP drifting" after power outages:

**Nest Camera:** Locked to 192.168.0.156

**HP Printer:** Locked to 192.168.0.129

**Alexa Units:** Locked to .213 and .214

[Address Reservations] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/Network_clients_verification.png

**🔍 Security Validation & Audit**
To verify the integrity of the segmentation, I conducted **Layer 4 Service Analysis** and ICMP audit tests using **Fing Desktop.**

**1. ICMP Reachability (Ping)**
**Source:** Trusted Workstation (192.168.0.84)

**Target:** Isolated IoT Assets (Alexa/Echo)

**Result:** Destination Host Unreachable (100% Packet Loss).

**Significance:** Confirms the firewall effectively drops unsolicited inter-segment traffic.

[Confirmation of Ping] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/AP_isolation_ping_confirmation.png

**2. Service Exposure Audit (Port Scan)**
**Asset: HP Envy 5540 (Main Segment):** Failed ICMP Ping (Hardened) but showed Open Ports 80, 443, 631 (IPP), 8080.

**Verification:** Successfully confirmed local print functionality while external WAN access failed.

**Asset: Amazon Alexa (IoT Segment):** Detected Ports 1080, 8888 but failed data handshake.

**Verification:** Confirms AP Isolation is active; devices are "visible" at Layer 2 but "unreachable" at Layer 3.

[Fing Port Scan Results - Printer] Insert_Link_To_Printer_Port_Scan_Pic_Here

[Fing Port Scan Results - Alexa] Insert_Link_To_Alexa_Port_Scan_Pic_Here

**3. Outbound WAN Verification (The Cage Check)**
**Procedure:** Attempted a manual Firmware Update check from the HP Printer's console.

**Result:** Connection Timeout/Failed.

**Significance:** Validates that the "Bedtime" schedule successfully severed the device's ability to communicate with external HP servers.

[Firmware Update Failure Proof] Insert_Link_To_Firmware_Failure_Pic_Here

**📊 Security Wins**
**✅ Lateral Movement Neutralized:** A compromise of an IoT peripheral no longer grants visibility into sensitive financial or personal data.

**✅ Spectrum Optimization:** Offloaded low-bandwidth IoT traffic to the 2.4GHz band.

**✅ Verified Metadata Privacy:** Opted out of third-party client identification services.

**Troubleshooting:** The "Local Admin" Mismatch
During the initial setup, I encountered a common "Cloud-Lock" issue where the TP-Link Tether app attempted to restrict management to a cloud-bound TP-Link ID, preventing advanced local hardening.

**The Challenge**
The Tether mobile app "binds" the router to a cloud account, which can obscure advanced security settings and create a dependency on external servers for local network management.

**The Resolution (The "Power User" Bypass)**
Gateway Identification: Used PowerShell (Windows 11) to verify the exact local gateway address (192.168.0.1).

**Credential Sync:** Signed in via Web UI to "unbind" the cloud account, transitioning to a strictly Local Admin management model.

[PowerShell output of ipconfig] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/ipconfig_redacted.png

🧠 **Lessons Learned**
Discovery vs. Security: Strict isolation often breaks mDNS-dependent services. Security professionals must balance Segmentation with Operational Utility.

App vs. Web UI: Residential "Smart Apps" often obscure advanced security settings; direct Web UI access is mandatory for professional-grade hardening.

Inventory Verification is Key: Identified dual-interface devices (BT/Wi-Fi) that initially appeared as "mystery" nodes.

**🚀 Future Roadmap**
**[x] IoT AP Isolation:** Prohibited lateral traffic within the 2.4GHz segment.

**[x] WAN Caging:** Effectively utilized scheduling to disable internet access for high-risk assets.

**[ ] Wi-Fi 7 MLO Integration:** Transition to Multi-Link Operation once compatible hardware is available.
