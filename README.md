Abstract
As organizations harden their traditional email perimeters, threat actors are rapidly pivoting to mobile-centric attack vectors. Highlighting findings from the India Cyber Threat Report 2025, this project investigates the mechanics of "Quishing" (QR Phishing) combined with Cloud Storage Abuse. The project successfully demonstrates an end-to-end attack simulation and the subsequent development of a hybrid network forensics tool capable of intercepting mobile traffic, parsing PCAP data for intermediary command-and-control (C2) tunnels, and actively tracing obfuscated URLs to uncover malicious payloads hosted on trusted cloud platforms.
1. Introduction
The modern threat landscape has seen a significant increase in attacks designed to bypass Secure Email Gateways (SEGs). By embedding malicious links within QR codes, attackers force the victim to transition the attack from a protected corporate network to an unprotected personal mobile device. Furthermore, to evade domain reputation filters, attackers are increasingly hosting their malware (e.g., malicious .apk files) on legitimate cloud services like Google Drive or Dropbox. This project explores the forensic footprint of such an attack and develops a custom detection methodology.
2. Project Objectives
•	To construct a functional Quishing simulation utilizing URL shorteners, tunneling services, and cloud-hosted payloads.
•	To establish a methodology for intercepting and capturing network traffic (.pcapng) from mobile devices on isolated networks.
•	To develop an automated forensic analysis tool capable of detecting evasive routing and classifying cloud-storage abuse.




3. Architecture & Methodology
The project was executed in three distinct phases, representing both the "Red Team" (Attack) and "Blue Team" (Defense) perspectives.
 
Phase 1: Attack Simulation (The Trap)
A malicious infrastructure was established to mimic real-world threat actor tactics.
•	Payload Hosting: A simulated malware file (malware_test.apk) was hosted on Google Drive.
•	Intermediary Server: A Python Flask server was deployed on Google Colab to act as a redirector, logging victim interaction.
•	Tunneling & Obfuscation: pyngrok was used to expose the internal server to the public internet, and the resulting URL was heavily obfuscated using a URL shortener (bit.ly).
•	Delivery: A QR code was generated, pointing to the obfuscated link.
Phase 2: Traffic Interception (The Wiretap)
Because the victim device is a mobile phone, direct packet capture is complex.
•	A laptop was configured as an intermediary Wi-Fi Hotspot.
•	The target mobile device connected to this controlled access point.
•	Wireshark was utilized to capture the live network traffic (quishing_evidence.pcapng) as the victim scanned the QR code and was routed to the payload.

Phase 3: Forensic Analysis (The Analyzer)
A custom Python-based forensic tool ("Quish-Hunter") was developed to automate the investigation of the captured evidence. The tool operates in two modes:
•	Passive Analysis: Utilizes the scapy library to parse the .pcapng file, specifically hunting for DNS queries (UDP Port 53) directed at known URL shorteners or tunneling services (e.g., ngrok).
•	Active Tracing: Utilizes the requests library to emulate a mobile User-Agent, actively following the HTTP 301/302 redirect chain of the suspicious link to uncover the final hidden destination.
4. Results & Findings
The execution of the Quish-Hunter tool against the captured network evidence yielded the following successful detections:
1. Passive DNS Detection: The tool successfully parsed 2,646 packets and identified the intermediary tunneling infrastructure, proving the mobile device communicated with an external relay.
[!] ALERT: SUSPICIOUS ACTIVITY DETECTED! Evidence found of communication with: -> acrotic-ascertainably-angie.ngrok-free.dev
2. Active Redirect Tracing: Upon manual input of the initial shortened URL (https://bit.ly/3NkV9Lg), the tool successfully mapped the redirect chain, bypassing the obfuscation.
Hop 1: https://bit.ly/3NkV9Lg [Status: 301] FINAL DESTINATION: https://drive.google.com/file/d/...
3. Threat Classification: The heuristic analysis engine correctly flagged the final destination, validating the threat intelligence hypothesis that cloud services are being abused.
[!] RISK: Destination is a Cloud Storage Provider (Often used to bypass firewalls). [!] Verdict: SUSPICIOUS. Proceed with caution.


4. Conclusion
This project proves that relying solely on static IP blacklists or standard email filters is insufficient against modern Quishing campaigns. By combining passive packet sniffing with active URL resolution, defenders can successfully unmask obfuscated attack chains. The methodology developed here serves as a viable proof-of-concept for investigating mobile-first threats and cloud service abuse.


