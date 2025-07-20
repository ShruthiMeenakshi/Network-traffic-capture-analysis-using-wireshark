# 🌐 Wireshark Local Network Traffic Analysis by Kali Linux

## Understanding LAN Communications Through a Packet Capture

![Wireshark Local Network Analysis Screenshot](<wireshark in local network.png>)

This repository provides a detailed analysis of a Wireshark packet capture taken on a local area network (LAN). It aims to illustrate common network protocols in action, offering insights into how devices communicate, discover each other, and obtain network configurations within a typical home or lab environment.

Whether you're new to network analysis, studying for certifications, or just curious about what's happening on your network, this capture serves as a practical example.

---

## 🚀 Key Takeaways from This Analysis

* **Core LAN Protocols:** Understand the real-world usage of DHCP, ARP, NBNS, BROWSER, and MDNS.
* **Packet Structure:** See how different protocol layers (Ethernet, IP, UDP/TCP, Application) are encapsulated within a single packet.
* **Device Discovery:** Observe how devices dynamically find each other and establish communication without manual configuration.
* **Troubleshooting Insights:** Learn to identify common network negotiation patterns.

---

## 🔍 Deep Dive into the Capture (Screenshot Breakdown)

The provided screenshot (`wireshark in local network.jpg`) showcases three primary Wireshark panes:

1.  **Packet List Pane (Top):** Provides a high-level summary of each captured frame, including its number, timestamp, source, destination, protocol, length, and a brief informational description.
2.  **Packet Details Pane (Middle):** Offers an in-depth, layered view of a selected packet's contents, breaking down the headers and data of each protocol (e.g., Ethernet, IP, UDP, and application-specific layers).
3.  **Packet Bytes Pane (Bottom):** Displays the raw hexadecimal and ASCII representation of the selected packet's data, useful for low-level inspection.

---

## 📡 Protocols Explained

Let's dissect the key network protocols identified in the "Protocol" column of the Packet List:

### 1. **DHCP (Dynamic Host Configuration Protocol)**
* **Purpose:** Enables devices (clients) to automatically obtain IP addresses and other network configuration parameters (subnet mask, default gateway, DNS server) from a DHCP server. This is fundamental for modern network setup.
* **Observation:** You'll see `DHCP ACK` (server confirming IP assignment) and `DHCP REQUEST` (client asking for an IP).
* **Example from Capture:** `59 DHCP ACK - Transaction ID 0xc2371f00` indicates a successful IP lease.

### 2. **ARP (Address Resolution Protocol)**
* **Purpose:** A crucial Layer 2 (Data Link) protocol used to map an IP address (Layer 3) to a physical MAC address on the local network segment. Devices use ARP to find the MAC address of a local destination before sending an IP packet.
* **Observation:** You'll see `ARP` requests like `Who has 10.0.2.37? Tell 10.0.2.15` (a device asking for an IP's MAC) and corresponding `ARP` replies.
* **Example from Capture:** `60 10.0.2.3 is at 08:00:27:00:ac:0e` is an ARP reply, providing the MAC address for `10.0.2.3`.

### 3. **NBNS (NetBIOS Name Service)**
* **Purpose:** A legacy Microsoft protocol used for name resolution on local networks, primarily for older Windows machines. It resolves human-readable NetBIOS names (e.g., computer names) to IP addresses.
* **Observation:** The capture shows "Registration NB" (a computer registering its name and IP) and "NBNS Query" (a computer asking for another computer's IP by its NetBIOS name).
* **Example from Capture:** `110 Registration NB <01><02> WORKGROUP <02><01>` shows a device registering its presence within the "WORKGROUP" domain.

### 4. **BROWSER (Computer Browser Service)**
* **Purpose:** Another legacy Microsoft networking protocol responsible for maintaining a dynamic list of active computers and shared resources on a Windows network. There's typically a "Master Browser" that aggregates and distributes this list.
* **Observation:** "Local Master Announcement" entries are visible, where systems like `METASPLOITABLE` (a common vulnerable VM in cybersecurity labs) announce their role as the Master Browser, or just their presence.
* **Example from Capture:** `286 Local Master Announcement METASPLOITABLE, Workstation...` shows `METASPLOITABLE` advertising its network roles and capabilities.

### 5. **MDNS (Multicast DNS)**
* **Purpose:** A modern protocol (part of Bonjour, Avahi, etc.) that allows devices on a local network to discover services and resolve hostnames to IP addresses without needing a central DNS server. Common in IoT, smart devices, and Apple environments.
* **Observation:** The presence of MDNS packets suggests that modern, zero-configuration networking-capable devices are active on the network.

---

## 🔬 In-Depth Packet Analysis (Example Packet)

Let's examine the selected packet (Packet No. 39 in the list summary: `39 337.669051402 PcsSystemte_00:ac:.. ARP 42 Who has 10.0.2.3? Tell 10.0.2.15`) as detailed in the bottom two panes of the screenshot.

*(**Note:** The details pane shown in the screenshot actually corresponds to an **NBNS query**, not the highlighted ARP packet. We will explain the details pane's content accurately.)*

### 1. **Frame Layer (OSI Layer 1/2 - Physical/Data Link)**
* **`Frame 9: 92 bytes on wire (736 bits), 92 bytes captured (736 bits) on interface eth0, id 0`**
    * **Meaning:** This describes the raw data block as captured by Wireshark. It indicates the total size of the frame as transmitted on the network (`on wire`) and the portion saved by Wireshark (`captured`). `eth0` is the network interface used for the capture.

### 2. **Ethernet II Layer (OSI Layer 2 - Data Link)**
* **`Ethernet II, Src: PcsSystemte_00:ac:0e (08:00:27:00:ac:0e), Dst: Broadcast (ff:ff:ff:ff:ff:ff)`**
    * **Meaning:** This is the Ethernet header.
        * **Source MAC Address (`Src`):** `08:00:27:00:ac:0e`. `PcsSystemte` indicates the manufacturer (likely VMware or VirtualBox, as `08:00:27` is a common OUI for virtual network adapters).
        * **Destination MAC Address (`Dst`):** `ff:ff:ff:ff:ff:ff`. This is the **Ethernet Broadcast** address, meaning the frame is intended for all devices on the local network segment.

### 3. **Internet Protocol Version 4 (OSI Layer 3 - Network)**
* **`Internet Protocol Version 4, Src: 10.0.2.4, Dst: 20.0.2.255`**
    * **Meaning:** This is the IP header, defining the logical source and destination.
        * **Source IP Address:** `10.0.2.4`.
        * **Destination IP Address:** `20.0.2.255`. This appears to be a network broadcast address for the `20.0.2.0/24` subnet.

### 4. **User Datagram Protocol (OSI Layer 4 - Transport)**
* **`User Datagram Protocol, Src Port: 137, Dst Port: 137`**
    * **Meaning:** This is the UDP header. UDP is a connectionless transport protocol, often used for services that prioritize speed over guaranteed delivery.
        * **Source Port:** `137`.
        * **Destination Port:** `137`. Port 137 is specifically registered for **NetBIOS Name Service (NBNS)**. This confirms that the data within this packet is an NBNS message.

### 5. **NetBIOS Name Service (OSI Layer 7 - Application)**
* **`NetBIOS Name Service`**
    * **Meaning:** This is the highest-level protocol, the actual application data.
    * **Key Fields:**
        * `Transaction ID: 0x5f27`
        * `Flags: 0x0110, Opcode: Name query, Recursion desired, Broadcast` - These flags indicate that this is a query for a NetBIOS name, requesting that the lookup be handled recursively, and sent as a broadcast.
        * `Queries`: Contains details about what name is being queried.
            * `<01><02> WORKGROUP<01> type NB, class IN` - The query is specifically for the "WORKGROUP" name.
            * `Name: WORKGROUP<01> (Local Master Browser)` - This indicates the query is searching for the *Local Master Browser* responsible for the "WORKGROUP" domain.

---

## 🛠️ How This Capture Was Obtained

This capture was likely obtained using Wireshark on a Kali Linux virtual machine, potentially within a lab environment containing other virtual machines or systems configured for network interaction (e.g., Metasploitable, Windows VMs).

---

## 📚 Further Learning

To deepen your understanding, consider exploring:

* **Wireshark Documentation:** The official Wireshark user guide.
* **OSI Model:** Understanding the 7 layers of the Open Systems Interconnection model.
* **TCP/IP Model:** The practical model for internetworking.
* **Network+ Certification:** Covers many of these fundamental networking concepts.
* **Specific Protocol RFCs:** For in-depth technical specifications of protocols like DHCP (RFC 2131), ARP (RFC 826), and NBNS (various Microsoft specifications).

---

## 🤝 Contribution

Feel free to open an issue or pull request if you have any questions, spot an error, or would like to contribute further analysis!

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE) (or choose your preferred license).

---
