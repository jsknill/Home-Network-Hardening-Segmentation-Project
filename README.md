**Hardware: TP-Link Archer BE3600 (Wi-Fi 7 Router)**

**Objective:** Transition from a "flat" unmanaged home network to a hardened, segmented architecture to reduce attack surface and isolate high-risk IoT/Legacy devices.

📋 Project Overview
This project documents the end-to-end security hardening of a residential network. By leveraging a Wi-Fi 7 router's advanced management features, I implemented Network Segmentation, MAC-to-IP Binding, and Asset Identification to protect primary data-processing units from vulnerable IoT peripherals.

**"Note: All IP addresses shown are part of a private internal network (192.168.0.x) and are used for documentation and architectural demonstration purposes only."**

🏗️ Network Architecture

The network is divided into three distinct logical segments (VLANs/SSIDs) to enforce the Principle of Least Privilege:

**Main Network**: Trusted,High-speed 5GHz/6GHz band for primary PCs and mobile devices.
**Home IoT**: Untrusted, Isolated 2.4GHz band for "smart" devices, cameras, and legacy consoles.
**Guest Network**: Minimal,Temporary access for visitors; 100% isolated from internal assets.

🛠️ **Implementation Steps**
1. **Baseline Hardening**
Local Admin Migration: Shifted from cloud-based (TP-Link ID) management to Local Admin access via the Web UI (192.168.0.1) to reduce external dependencies and exposure.

Protocol Security: Implemented WPA2/WPA3 Personal mixed mode to maintain compatibility with legacy hardware (Xbox 360, Alexa) while providing maximum encryption for modern clients.

2. **Asset Discovery & Documentation**
One of the most critical phases was performing a full "Pull-Test" and OUI lookup to identify mystery devices.

Discovery: Identified two unknown MAC addresses (08-A6-BC and D2-9C-AF).

Validation: Through physical disconnection testing, confirmed these were the Wi-Fi and Bluetooth components of a TP-Link Archer T2UB Nano USB adapter.

Labeling: Renamed all clients in the Network Map to ensure immediate recognition of unauthorized lateral movement.

3. **Attack Surface Reduction (Isolation)**
Using Device Isolation, I "fenced off" high-risk devices that are notoriously poorly patched or have known vulnerabilities:

IoT Devices: Moved Nest Cameras, Ring Doorbell, and Amazon Alexa units to the IoT segment.

Legacy Consoles: Isolated Xbox 360 and Nintendo 3DS units to prevent them from becoming an entry point into the main data network.

Printers: Moved the HP Envy 5540 to IoT to prevent printer-based exploit pivots.

4. **Persistence & Stability (Address Reservations)**
To ensure security rules and logging remain consistent, I implemented DHCP Address Reservations. This prevents "IP drifting" after power outages:

Nest Camera: Locked to 192.168.0.156

HP Printer: Locked to 192.168.0.129

Alexa Units: Locked to .213 and .214

![Address Reservations] 

📊 **Security Wins**

✅ **Zero Lateral Movement**: A compromise of an IoT device (e.g., the Ring Doorbell) no longer grants access to personal financial data on the main PCs.

✅ **Reduced Interference**: Optimized the 5GHz band by offloading low-bandwidth IoT traffic to the 2.4GHz IoT segment.

✅ **Verified Metadata Privacy**: Opted out of third-party client identification services to keep device lists private to the local gateway.

5. **Troubleshooting: The "Local Admin" Mismatch**
During the initial setup, I encountered a common "Cloud-Lock" issue where the TP-Link Tether app attempted to restrict management to a cloud-bound TP-Link ID, preventing advanced local hardening.

The Challenge

The Tether mobile app "binds" the router to a cloud account, which can obscure advanced security settings and create a dependency on external servers for local network management. When attempting to switch to the Web UI, I faced a "TP-Link ID vs. Local Admin" credential mismatch.

The Resolution (The "Power User" Bypass)

To regain full local control and ensure I was accessing the gateway directly, I performed the following:

Gateway Identification:
Used PowerShell (Windows 11) with Administrator privileges to verify the exact local gateway address.

PowerShell
ipconfig
Result: Confirmed the Default Gateway as 192.168.0.1.

Direct Browser Access:
Instead of relying on the app's redirected links, I navigated directly to http://192.168.0.1 in a secure browser session.

Credential Sync:
I resolved the "mismatch" by signing in with the TP-Link ID credentials established during the Tether setup. This granted me the elevated permissions needed to "unbind" the cloud account and transition to a strictly Local Admin management model, which is a key step in reducing the router's external attack surface.

**[PowerShell output of ipconfig] https://github.com/jsknill/Home-Network-Hardening-Segmentation-Project/blob/main/ipconfig_redacted.png**

**Why this is a "Technical Win"**
By using ipconfig to find the gateway, I bypassed the "easy" setup path (which favors vendor data collection) in favor of a "hardened" setup path (which favors user privacy and local control). This ensured that my Device Isolation and Address Reservation settings were configured on the router itself, not just a cloud-cached version of it.

## 🧠 Lessons Learned
- **Inventory Verification is Key:** Initially, the "none" and "network device" IDs were confusing. Physically disconnecting the TP-Link T2UB Nano adapter confirmed that "Combo" devices (BT/Wi-Fi) often present as multiple MAC addresses.
- **App vs. Web UI:** While mobile apps are convenient for "Day 1" setup, the Web UI is mandatory for "Day 2" hardening. Cloud-binding via the Tether app can obscure critical security controls.
- **Hardware Limitations:** Discovered that while the router is Wi-Fi 7 (802.11be) capable, client-side hardware (Intel AC 3168) is the current bottleneck, emphasizing the importance of a phased upgrade path.

  
🚀 **Future Roadmap**

[ ] Implement AP Isolation within the IoT segment to prevent "east-west" traffic between smart devices.

[ ] Configure Access Schedules to disable internet access for legacy consoles during overnight hours.

[ ] Transition to Wi-Fi 7 MLO (Multi-Link Operation) once compatible hardware is integrated into the primary workstation.
