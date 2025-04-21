# Project Report
**Course**: IT 359 - Section 2

**Group**: 12

**Project**: Chisel Tunneling Tool

**Members**: Tara Larrabee, Koby Fletcher


## Project Overview
This project is all about showing how the Chisel tunneling tool can be used in a restricted network. The goal of this lab is to understand and demonstrate different port forwarding methods, like local, remote, and dynamic, using Chisel. Everything is done in a Docker-based environment that simulates a segmented internal network with a firewall in between. 

There are two main files that walk through everything: 
- [`SETUP.md`](SETUP.md): goes over how to build the lab environment from the IT 359 Proxmox Kali VM. 
- [`MINI-LAB.md`](MINI-LAB.md): walks through all the different Chisel tunneling examples step-by-step

## Goals
- Set up a segmented container network with a simulated firewall
- Show how to bypass firewall restrictions using Chisel
- Demonstrate the use of remote and dynamic port forwarding
- Practice pivoting deeper into an internal network using SOCKS5


## Environment Summary
**Host Machine**: Kali Linux (used to simulate the attacker)

**Containers**:
- `pfrouter`: simulates a firewall (like pfSense)
- `ubuntu_target`: internal Ubuntu machine
- `metasploitable`: internal vulnerable system

**Virtual Interface**: `attacker0` manually set up on Kali to connect it to Docker's `attacker_net`

**Firewall**: `iptables` on `pfrouter` to block inbound traffic and force tunneling

> See [`SETUP.md`](SETUP.md) for the full build instructions and diagram. 

## What We Demonstrated Using Chisel
**Local Port Forwarding**
- Tried to connect from attacker to internal service using local port forwarding
- It failed as expected due to firewall restrictions (no inbound access allowed)

**Remote Port Forwarding**
- Created a reverse tunnel from the target to the attacker 
- This showed how we could: 
    - Access Apache from `ubuntu_target` through the tunnel
    - SSH into `ubuntu_target` using the reverse port
    - Set up `samba` and downloaded a file from a public SMB share

**Dynamic Port Forwarding (SOCKS5 + `proxychains`)**
- Access the web server on `metasploitable`
- Scan internal ports using `nmap`
- Pivot into the network using Firefox for visual interaction

## Key Takeaways
- Chisel is a super useful tool for tunneling across restricted networks

- Local port forwarding does not work in real-world firewalled environments unless inbound connections are allowed

- Reverse tunneling and SOCKS5 are powerful for post-exploitation and pivoting

- Docker makes it easy to simulate these scenarios without needing a full physical lab

## Tools Used
- Kali Linux
- Docker + Docker Compose
- Chisel (v1.8.1)
- Apache2, SSH, Samba
- `iptables`
- `proxychains`, `curl`, `nmap`, Firefox

## References 
- https://docs.docker.com/compose/install/

- https://ubuntu.com/core/docs/docker-run 

- https://phoenixnap.com/kb/iptables-linux

- https://builtin.com/software-engineering-perspectives/ssh-port-forwarding#:~:text=Remote%20port%20forwarding%2C%20also%20known,running%20on%20a%20remote%20server. 

- https://github.com/jpillora/chisel/blob/master/README.md#usage

- https://jieliau.medium.com/chisel-tool-for-your-lateral-movement-dd3fb398c696

- https://www.techtarget.com/searchnetworking/definition/Server-Message-Block-Protocol#:~:text=The%20Server%20Message%20Block%20(SMB)%20protocol%20is%20a%20client%2D,SMB%20protocol%20in%20the%201980s. 

- https://github.com/haad/proxychains 

## Final Thoughts
This lab demonstrates how Chisel can be used in real-world like scenarios where the attacker cannot reach the internal network directly. By using reverse port forwarding and dynamic tunneling with SOCKS5, we were able to pivot into the network and reach machines and services that would normally be restricted to external networks. 