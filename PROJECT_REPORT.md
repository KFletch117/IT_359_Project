# Project Report
**Course**: IT 359 - Section 2

**Group**: 12

**Project**: Chisel Tunneling Tool

**Members**: Tara Larrabee, Koby Fletcher


## Project Overview
Inside of an organization, systems and employees typically need access external sources on the Internet to gather the information they need to get their job done. It's common for an organization to allow the company firewall to accept outbound connections to the ports that access these services, like port 80 (HTTP) and port 443 (HTTPS). It's also very common practice to block inbound requests coming from most ports, like ports 80 and 443, to internal machines because it exposes these machines, which is a severe risk to the private network. As an employee, when you are working on an internal machine inside of the company's private network, accessing an external website on the Internet means your device initiates an outbound connection to the webserver's IP address on port 443. The company's NAT device inside of the firewall will rewrite your source IP address and port using a translation table to allow return traffic to flow back through the same session. 

This trust that was established using outbound connections can be an opportunity for abuse. Chisel becomes useful in this situation. Chisel is a tunneling tool that creates a fast TCP/UDP tunnel over HTTP/HTTPS and leverages this to hide communication channels when the compromised machine establishes outbound connections. Instead of the internal machine sending a reqest out to a webserver, it sends it to the external machine running the Chisel Server, which is usually on port 80 or port 443. The communication succeeds in moving through the firewall to disguise itself as legitimate web activity. So, from the perspective of the company, it looks like the employee is simply browsing the web.

Our goal for this project is to show how Chisel is used to bypass a restricted network through several different scenarios and by using different methods to bypass the firewall. This project covers how to use Chisel by setting up a segmented container network with a simulated firewall. This setup is done so that we can define our own machines and firewall rules within a small, virtual environment to safely demonstrate the use of Chisel. By creating our own restricted firewall, we can accurately demonstrate the use of Chisel while also putting emphasis behind the critical function of Chisel, that is, bypassing the firewall. 


## Environment Summary
To simulate a real-world network setup, we used Docker to set up a virtual, containerized network. Each device made in the network as containers are images of a specified machine with limited space and capability, which is fine for our setup. The internal network we made is managed by a firewall, called by pfrouter and it is configured to allow all outbound connections but deny all inbound connections. This means an external machine, will not be able to establish a connection to the machines inside of the internal container network. 

### Network Breakdown:

**Host Machine**: Kali Linux (used to simulate the attacker)

**Containers**:
- `pfrouter`: simulates a firewall (like pfSense)
- `ubuntu_target`: internal Ubuntu machine
- `metasploitable`: internal vulnerable system

**Virtual Interface**: `attacker0` manually set up on Kali to connect it to Docker's `attacker_net`

**Firewall**: `iptables` on `pfrouter` to block inbound traffic and force tunneling

> See [`SETUP.md`](SETUP.md) for the full build instructions and diagram. 

## What We Demonstrated Using Chisel
For Chisel to bypass a firewall, it uses port forwarding. There are three types of port forwarding that Chisel supports: local, remote, and dynamic port forwarding. 
### **Local Port Forwarding**
Local port forwarding is a technique where the external machine opens a local port from their own machine. This redirects traffic through a tunnel to a specified port on a service running on internal machine. Chisel supports local port forwarding but it is expected to fail when used against an internal network with firewall restrictions, dropping all inbound access. In our mini-lab, we demonstrate this by trying to connect the attacker to an internal service using local port forwarding to show it failing in our scenario. 

### **Remote Port Forwarding**
Remote port forwarding is the opposite of local port forwarding.The internal machine creates tunnel that allows the external machine to open a port on their own machine. So traffic is sent to the port on the external machine and then forwarded through the tunnel to the service running on the internal machine. This is where Chisel thrives. We demonstrated the use of a reverse tunnel from the target to the attacker. We show how we could access Apache from `ubuntu_target`, SSH into `ubuntu_target`, and set up `samba` to download a file from a public SMB share as the attacker. 

### **Dynamic Port Forwarding (SOCKS5 + `proxychains`)**
Dynamic port forwarding is a technique that uses a SOCKS5 proxy on the external machine which allows traffic from multiple services to be tunneled through the internal machine. Chisel utilizes the reverse tunneling with a proxy to pivot deeper into the internal network by gaining access to services on an adjacent machine to the one that is compromised. To demonstrate this we tried to access services on `metasploitable` by access its own web server hosted on itself, by scanning its internal ports using `nmap`, and by providing a more visual representation by gaining access to Firefox on it as well. 

## Key Takeaways
Chisel is a useful tool for tunneling across restricted networks. It is fast, lightweight, and provides a stealthy technique for attackers to gain access to an internal network. Another important note is that local port forwarding does not work in real-world firewalled environments unless inbound connections are allowed. On the other hand, reverse tunneling and SOCKS5 are powerful for post-exploitation and pivoting. Lastly, Docker makes it easy to simulate these scenarios without needing a full physical lab. 

## Final Thoughts
Over the course of this project, there were many successes and also many problems to troubleshoot. To highlight a few, the things that worked were setting up and using the docker environment, configuring the firewall correctly to block external machines, getting reverse and dynamic port forwarding to work, and making everything compatible. Some of the problems included losing Internet access completely from the Kali machine so that none of the containers had internet access, the metasploitable container not being able to download more recent installations/packages, and losing all network configurations when exiting a container.

On top of learning the "ins and outs" of the Chisel tool, in general, we learned more about what tunneling is and how it can be used to exploit a compromised machine. We also learned what containers are, how they work, and understood how useful they are in hacking activities like this. Setting up the environment to use Chisel gave us a better understanding of using a firewall and configuring the right devices to communicate with each other while denying everything we did not want to communicate to our internal network. Some of the services and understanding proxies was another learning curve to this project. Overall, the amount of information we gain was substantial and supplemental to the class itself.

In conclusion, we demonstrated how Chisel can be used in real-world like scenarios where the attacker cannot reach the internal network directly. By using reverse port forwarding and dynamic tunneling with SOCKS5, we were able to successfully pivot into the network and reach machines and services that would normally be restricted to external networks. 


Please review these documents:
- [`SETUP.md`](SETUP.md): goes over how to build the lab environment from the IT 359 Proxmox Kali VM. 
- [`MINI-LAB.md`](MINI-LAB.md): walks through all the different Chisel tunneling examples step-by-step


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

