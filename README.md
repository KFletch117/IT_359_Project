# IT 359 - Chisel Tunneling 
Welcome to our IT 359 Final Project! 

Our final project required us to choose a unique offensive security topic useful for ethical hacking that was not covered in class. The class covers the tools and techniques used in penetration testing, so we chose the tool Chisel. 

Chisel is a tunneling tool used for firewall evasion, secure communication, and network pivoting. This project demonstrates how an attacker can use Chisel to bypass internal segmentation, tunnel services over HTTP, and pivot deeper into a restricted environment using reverse tunneling and SOCKS5 proxies.

## Project Features
- Segmented internal Docker network with a firewall
- Reverse and Local Port Forwarding with their expected outcomes
- Dynamic SOCKS5 proxy tunneling using proxychains

## Team Members
**Tara Larrabee**
- Wrote and constructed [`MINI-LAB.md`](MINI-LAB.md) and [`SETUP.md`](SETUP.md) walkthroughs
- Implemented the attacker-side setup and proxychains pivoting on the Kali machine
- Demonstrated Chisel tunneling techniques (reverse, local, and dynamic)
- Created and tested all exercises and scenario walkthroughs
- Ensured attacker-side and target-side configurations worked as intended

**Koby Fletcher**
- Built and tested the segmented Docker network with internal and external routing
- Designed **ubuntu_target** and **metasploitable** routing to simulate internal segmentation
- Designed and implemented the firewall on **pfrouter** using `iptables`
- Validated breakage points and firewall bypass behavior throughout the lab
- Assisted in creating lab design, learning objectives and lab questions

## Project Contents
- [`SETUP.md`](SETUP.md) – Environment setup and walkthrough for container configuration
- [`MINI-LAB.md`](MINI-LAB.md) – Mini-lab guide on how to use Chisel with real-world scenarios
- [`PROJECT_REPORT.md`](PROJECT_REPORT.md) – Final summary of the project and findings

## Images
Additional images for the project included `images` folder.

## Disclaimer
This project is for educational purposes only. All demonstrations are performed in a controlled environment to simulate real-world attack scenarios for ethical hacking. 