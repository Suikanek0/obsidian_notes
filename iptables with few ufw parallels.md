created this note so my lazy ass wouldn't depend on ufw while managing my whole home network + port forwarding through debian machine... need to learn ts so i wont be prey for russian cyka blyat haxors!!!!
## basic commands

```
ufw allow {port} 
```
**which allows specific port to go through server is parallel to**
```
iptables -A input -p tcp --dport {port} -j accept 
```
**or** *allowing from specific ip*
```
iptables -A input -p tcp -s {ip} --dport {port} -j accept
```
________________________________________________________________________

```
ufw status
```
**which shows applied rules it the same as:** 
```
iptables -L
```

command to show all nat and input/output rules **AS COMMANDS** very good for learning and memorizing commands

```
iptables -S && iptables -t nat -S
```

or organized neatly by number lines
```
iptables -L -n -v --line-numbers && iptables -L -t nat -n --line-numbers
```
### Managing default traffic behaviour

***types:INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING
action: DROP, ACCEPT, QUEUE, RETURN

```
iptables -P {type} {action}
```
# Recommended defaults

*by default drop every packet that isn't allowed*
```
iptables -P INPUT DROP
```
*by default allow traffic to go out to world*
```
iptables -P OUTPUT ACCEPT
```
*by default disallow forwarding packets*
```
iptables -P FORWARD DROP
```

*allow localhost*
```
iptables -A INPUT -i lo -j ACCEPT
```

*hard to explain in shortcut, but essentially allows incoming (INPUT) packets that belong to already-established or related connections. for example allows return/ongoing traffic for an https connection AFTER i allowed port 443 in firewall rules*
```
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

*same logic for packets that are meant to be forwarded*

```
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```


# Forwarding

for forwarding we need to handle prerouting, forwarding, and postrouting, so essentially we need 2 commands for handling prerouting + postrouting + rule for server to actually accept it (if we got default FORWARD DROP policy) 

*prerouting*
```
iptables -t nat -A PREROUTING -p {protocol type} --dport {port} -j DNAT --to-destination {ipaddress:port}
```

*postrouting*
```
iptables -t nat -A POSTROUTING -p {protocol type} -d {ipaddress} --dport {port} -j MASQUERADE
```
*finally rule so it could be allowed without getting dropped*
```
iptables -A FORWARD -p {protocol type} -d {ipaddress} --dport {port} -j ACCEPT
```

**heres a proper example how would i set port forwarding to hmm i dunno port 2137 on tcp that comes from 192.168.100.100**
```
iptables -t nat -A prerouting -p tcp --dport 2137 -j DNAT --to-destination 192.168.100.100:2137

iptables -t nat -A postrouting -p tcp -d 192.168.100.100 --dport 2137 -j MASQUERADE

iptables -A FORWARD -p tcp -d 192.168.100.100 --dport 2137 -j ACCEPT
```
that should be all and port forwarding should successfully work with this configuration and previous defaults!!


#### *little side note to for these 3 commands to properly understand what they do*
lets break the first command down, -t nat is changing default filter table to nat one, -A prerouting is defining that we are gonna manipulate packet before routing decision is made, now -p is just defining protocol for given port, -dport is self explanatory as its just port that is gonna be forwarded, -j DNAT is the destination translation of address, and --to-destination is self-explanatory as well it is choosing to which address its gonna be forwarded.
before dnat
**SOURCE:  1.2.3.4:50000**
**DESTINATION: your.public.ip:2137**
and after dnat it translates like this:
**SOURCE:  1.2.3.4:50000**
**DESTINATION: 192.168.100.100:2137**


second command: -t nat is explained above, -A postrouting, now this rule says that it should be targeting packets on which routing decision has been made and that want to leave the server, -p is protocol as explained above, -d is destination ip address, --dport is destination port, and -j MASQUERADE is translation layer
on this step masquerade translates it like this:
**SOURCE: your.server.LAN.ip:some-port** 
**DESTINATION: 192.168.100.100:2137**

finally last rule which is self explanatory it just allows packets to be forwarded from this ip address and port

![[Pasted image 20260827052100.png]]