**Residential Network Hardening & Advanced Segmentation**
Hardware: TP-Link Archer BE3600 (Wi-Fi 7)

Objective: Transition from a "flat" unmanaged network to a hardened, multi-segment architecture to minimize the attack surface and isolate high-risk IoT/Legacy assets.

**📋 Project Overview**
This project documents the end-to-end security hardening of a residential gateway. By leveraging Wi-Fi 7 management features, I implemented VLAN-style Network Segmentation, Static DHCP Binding, and Layer 2/3 Isolation to shield primary data-processing units from vulnerable IoT peripherals.

Note: All IP addresses shown (192.168.0.x) pertain to a private internal network and are used exclusively for architectural demonstration.

**🏗️ Network Architecture**
The environment is divided into three logical segments to enforce the Principle of Least Privilege:

Main Network: High-trust 5GHz/6GHz segment for primary workstations and authorized mobile devices.

Home IoT: Untrusted 2.4GHz segment for "smart" hardware, cameras, and legacy peripherals.

Guest Network: Temporary, 100% isolated segment for external visitors with zero internal resource access.

**🛠️ Implementation Steps**
**Phase 1: Gateway & Protocol Hardening**
Management Plane Migration: Shifted from cloud-dependent (TP-Link ID) management to Local-Only Admin access. This reduces the external attack surface and eliminates vendor-cloud dependencies.

Cryptographic Standards: Deployed WPA3/WPA2 Mixed Mode, ensuring modern clients utilize the highest encryption standards while maintaining compatibility for legacy hardware (e.g., Alexa, Xbox 360).

**Phase 2: Asset Discovery & Identity Management**
OUI Analysis & "Pull-Testing": Conducted a physical inventory to resolve mystery MAC addresses. Identified a dual-interface TP-Link T2UB Nano (Bluetooth/Wi-Fi) through targeted disconnection.

MAC-to-IP Binding: Implemented DHCP Address Reservations to prevent "IP Drifting" and ensure security policies remain persistently bound to specific hardware IDs.

**Phase 3: Attack Surface Reduction (The "Caging" Strategy)**
I implemented a tiered isolation strategy based on device risk profiles:

IoT & Legacy Isolation: Leveraged Device Isolation to "fence off" Nest Cameras, Ring Doorbells, and legacy consoles (Xbox 360/3DS), preventing them from serving as pivot points.

Printer "Local-Only" Pivot: To resolve discovery issues (mDNS/WSD) caused by strict Layer 2 isolation, the HP Envy 5540 was migrated to the Main segment with Outbound WAN Filtering.

The "Bedtime" Kill-Switch: Utilized Parental Control Access Schedules to impose a permanent "Bedtime" on the printer. This enforces a total WAN blackout—preventing unauthorized "phone-home" telemetry—while maintaining full LAN availability for trusted print services.

**Phase 4: AP Hardening & Multicast Control**
IGMP Snooping & Proxy: Optimized multicast delivery to ensure discovery packets are only routed to authorized subscribers.

Intra-BSS Traffic Blocking: Enabled AP Isolation on the IoT segment to prohibit "East-West" communication, ensuring a compromised smart bulb cannot interact with neighboring IoT assets.

**Persistence & Stability (Address Reservations)**
To ensure security rules and logging remain consistent, I implemented DHCP Address Reservations. This prevents "IP drifting" after power outages:

Nest Camera: Locked to 192.168.0.156

HP Printer: Locked to 192.168.0.129

Alexa Units: Locked to .213 and .214

[Address Reservations] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/Network_clients_verification.png

**🔍 Security Validation & Audit**
To verify the integrity of the segmentation, I conducted Layer 3 ICMP audit tests between segments:

Source: Trusted Workstation (192.168.0.84)

Target: Isolated IoT Asset (192.168.0.214)

Result: Destination Host Unreachable (100% Packet Loss).

Conclusion: Verified that the firewall effectively drops unsolicited inter-segment traffic, successfully neutralizing the risk of lateral movement.

[Confirmation of Ping] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/AP_isolation_ping_confirmation.png

**📊 Security Wins**
✅ **Lateral Movement Neutralized:** A compromise of an IoT peripheral no longer grants visibility into sensitive financial or personal data.

✅ **Spectrum Optimization:** Offloaded low-bandwidth IoT traffic to the 2.4GHz band, preserving 5GHz/6GHz airtime for high-performance tasks.

✅ **Verified Metadata Privacy:** Opted out of third-party client identification to keep internal device lists local to the gateway.

**Troubleshooting: The "Local Admin" Mismatch**
During the initial setup, I encountered a common "Cloud-Lock" issue where the TP-Link Tether app attempted to restrict management to a cloud-bound TP-Link ID, preventing advanced local hardening.

The Challenge
The Tether mobile app "binds" the router to a cloud account, which can obscure advanced security settings and create a dependency on external servers for local network management. When attempting to switch to the Web UI, I faced a "TP-Link ID vs. Local Admin" credential mismatch.

**The Resolution (The "Power User" Bypass)**
To regain full local control and ensure I was accessing the gateway directly, I performed the following:

Gateway Identification: Used PowerShell (Windows 11) with Administrator privileges to verify the exact local gateway address.

PowerShell ipconfig Result: Confirmed the Default Gateway as 192.168.0.1.

Direct Browser Access: Instead of relying on the app's redirected links, I navigated directly to http://192.168.0.1 in a secure browser session.

Credential Sync: I resolved the "mismatch" by signing in with the TP-Link ID credentials established during the Tether setup. This granted me the elevated permissions needed to "unbind" the cloud account and transition to a strictly Local Admin management model.

[PowerShell output of ipconfig] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/ipconfig_redacted.png

🧠 **Lessons Learned**
Discovery vs. Security: Strict isolation often breaks mDNS-dependent services. Security professionals must balance Segmentation with Operational Utility using tools like WAN-filtering.

App vs. Web UI: Residential "Smart Apps" often obscure advanced security settings; direct Web UI access is mandatory for professional-grade hardening.

Inventory Verification is Key: Initially, the "none" and "network device" IDs were confusing. Physically disconnecting the TP-Link T2UB Nano adapter confirmed that "Combo" devices (BT/Wi-Fi) often present as multiple MAC addresses.

🚀** Future Roadmap**
[x] IoT AP Isolation: Prohibited lateral traffic within the 2.4GHz segment.

[x] WAN Caging: Effectively utilized scheduling to disable internet access for high-risk assets.

[ ] Wi-Fi 7 MLO Integration: Transition to Multi-Link Operation once compatible NICs are integrated into primary workstations.
