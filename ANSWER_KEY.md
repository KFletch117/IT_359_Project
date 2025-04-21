# Answer Key

1. Chisel is a tunneling tool that lets you move through a firewalled environments by creating tunnels over HTTP. It works by setting up a client and a server and once the tunnel is established, it forwards the traffic between the two machines. You can forward single ports or run a SOCKS5 proxy to pivot across a network. Chisel is fast and great for offensive testing when you are dealing with a restricted environment. 

2. Local port forwarding tries to connect inbound from the attacker to an internal service, meaning the attacker opens a port locally and forwards it to a target's service. In our lab, pfrouter blocks all inbound traffic so this fails as expected.

    Remote port forwarding works because the target (ubuntu_target) reachesout (outbound) to the attacker, which is allowed by the firewalll, and creates a reverse tunnel. This lets the attacker access the internal service through the tunnel, without needing a direct route in. That is why reverse mode is highlighted in our lab. 

3. Dynamic port forwarding creates a SOCKS5 proxy that routes traffic through a compromised target. It basically turns the compromised machine into a pivot point for scanning or accessing other internal systems or resources. You only set up one tunnel, but you can interact with multiple hosts and services through it. 

   In terms of syntax, it is more like remote port forwarding because the target client is still the one making the outbound connection to the attacker and binding a port there (`R:1080:socks`). 


4. SOCKS5 is important because it lets us dynamically route multiple types of traffic (e.g., HTTP, SSH) through one proxy. With Chisel's built-in SOCKS5 support, we can proxy tools like `nmap`, `curl`, and Firefox through the tunnel using `proxychains`. 

   This gives us the ability to scan internal networks, access hidden services, or even open up internal web apps without ever exposing ourselves or needing direct access to the network. It is perfect for digging deeper and pivoting to other targets once you are inside. 

5. Chisel is powerful because it assists attackers in bypassing firewalls that block inbound connections, accessing internal services through a reverse tunnel, pivoting into segmented networks with SOCKS5, and exfiltrating data or exploring hidden services. This is all done over HTTP, which blends in with normal web traffic. 


