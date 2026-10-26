# FortiGate Auto-Block with Honeypot & Python

A hands-on network security lab built in EVE-NG. The goal was to automate threat response: when an attacker touches a Honeypot in the DMZ, a Python script catches the traffic and blocks them on a FortiGate firewall automatically via REST API.

---

## Lab Architecture

Built and tested inside EVE-NG with three distinct network zones:
- **WAN (port1):** Attacker machine (`192.168.227.145`)
- **DMZ (port3):** Honeypot machine (`192.168.50.10`)
- **LAN (port2):** Internal network (`10.10.1.1/24`)

![Topology](screenshots/topology.png)

---

## Baseline Verification

Verified internal network connectivity to the gateway and confirmed internet access via NAT before launching any attacks:

![Baseline Connectivity](screenshots/baseline_test.png)

---

## How It Works

1. FortiGate sends traffic logs via Syslog (UDP port 514).
2. A custom Python script listens on port 514 and parses incoming logs in real time.
3. If traffic targets the Honeypot IP (`192.168.50.10`), the script triggers FortiOS REST API calls:
   - Creates a firewall address object for the attacker IP.
   - Adds the attacker into the `AI_Blocked_Attackers` address group.
4. The firewall policy blocks any further traffic from that IP.

---

## Setup & Verification Steps

### 1. FortiGate Interfaces
Configured the interfaces for WAN, LAN, and DMZ:
![Interfaces](screenshots/interfaces.png)

### 2. Firewall Policies
Policy order is critical here. The deny policy (`AI-Auto-Block`) must sit on top of the DMZ allow policy so matching packets get dropped first:
![Policy Order](screenshots/policy.png)

### 3. API Address Creation
Verified that the script successfully pushed the attacker's IP to the firewall:
![API Address](screenshots/api_address.png)

### 4. Attack Test & Result
Ran a ping test from the attacker machine to the honeypot. As soon as the first packets hit, the dashboard flagged the IP, pushed the block rule, and the ping immediately timed out:
![Proof of Concept](screenshots/proof_of_concept.jpeg)

---

## Issues Faced & Fixes

- **Policy Ordering:** Initially, the block rule was placed below the allow rule, so the ping kept going through. Moving the deny rule to top position fixed this.
- **Active ICMP Sessions:** Modifying the address group did not immediately kill already-open ICMP states. Running a session clear (`diagnose sys session clear`) was required so ongoing streams dropped instantly.
