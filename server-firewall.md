## Roadmap to success?!/ToDo-List
- [ ] setup working tunnel between clients on one VM per server and server on VPS
	- [x] WG-Server-VPS working
	- [ ] WG-Client-Fileserver working
	- [x] WG-Client-Gameserver working
- [ ] get traffic from VM's (10.0.30.0/24 & 10.0.40.0/24) routed to VPS 
	- [x] Route traffic between WG Server and WG Clients
	- [ ] Login/Auth/Landing page on VPS to limit access
	- [x] setup ip routes on VPS and MASQUERADE 
		- [x] `post-up ip route add 10.0.30.0/24 via 10.0.20.30`
		- [x] `post-up ip route add 10.0.40.0/24 via 10.0.20.40`
		- [x] `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE` (default by wireguard)
	- [ ] ping possible (on vps: `ping 10.0.30.160`)
		- [ ] VM traffic from/to WG-Client-Fileserver correct
		- [ ] VM traffic from/to WG-Client-Gameserver correct
- [ ] setup Nginx Reverse Proxy on VPS
	- [ ] working on WG-Server-VPS 
	- [ ] working on WG-Client-Fileserver
	- [ ] working on WG-Client-Gameserver

![Network Layout](./Networklayout_detailed.svg "Network Layout")
## pre-configuration
- install iptables: `sudo apt install iptables iputils-ping`
- enable IP-forwarding: `echo 1 > /proc/sys/net/ipv4/ip_forward`
- get current config: `iptables -filter -L`
## Wireguard Setup
### WG-Server:
- `sudo apt install wireguard`
- `sudo sysctl -w net.ipv4.ip_forward=1`
- `ip link show` -> \<public-interface\> 
- `wg genkey | tee privatekey | wg pubkey > publickey`
- `sudo nano /etc/wireguard/wg0.conf`
	```
	[Interface]
	PrivateKey=<server-private-key>
	Address=<server-ip-address>/<subnet>
	SaveConfig=true
	PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o <public-interface> -j MASQUERADE;
	PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o <public-interface> -j MASQUERADE;
	ListenPort = 51820
	```
- (setup client)
- `sudo wg set wg0 peer <client-public-key> allowed-ips <client-ip-address>/32`
- `wg-quick up wg0`
- `ping <client-ip-address>` -> working Tunnel
- `ping 8.8.8.8` -> forwarding works
### WG-Client-Fileserver:
- `sudo apt install wireguard`
- `sudo sysctl -w net.ipv4.ip_forward=1`
- `wg genkey | tee privatekey | wg pubkey > publickey`
- `sudo nano /etc/wireguard/wg0.conf`
	```
	[Interface]
	PrivateKey = <client-private-key>
	Address = <client-ip-address>/<subnet>
	SaveConfig = true
	
	[Peer]
	PublicKey = <server-public-key>
	Endpoint = <server-public-ip-address>:51820
	AllowedIPs = 0.0.0.0/0
	PersistentKeepalive=25
	```
- `wg-quick up wg0`
- `ping <server-ip-address>` -> working Tunnel
- `ping 8.8.8.8` -> forwarding works
### Troubleshooting
[![](https://mermaid.ink/img/pako:eNpdks1ugzAQhF9l5UsvARKpJ6RWakJ-Lj21N6gqyzZgATY1dlGU5N27wKpVasnWzux3mLV9YcJKxVJWtnYUNXce3rPCAK6XvNemAmnVYB48jNY1H0tnmwvuIemdFclwHhKjfKL770c8PkvrRu4kkVk-BGkBIeFbiEZANJ7Q-A992hC8W-CxIr3Pxyr6Clo0EHq01-Qf8sHrtgVjl1SYkjrHXLgz1Sei_uWn2SCKnmFL48ziChvch3trjZuuI5st6h9msaPcBGvjlSu5UOkUNo5j9E73CEaup0u9wp5mnBtEnWZxXARbsU65jmuJr3OZvIL5WnWqYCmWkrumYIW5IceDt29nI1jqXVAr5myoapaWvB1QhV5yrzLNK8e7X1dJ7a17XR5__gO3H7Vunws?type=png)](https://mermaid.live/edit#pako:eNpdks1ugzAQhF9l5UsvARKpJ6RWakJ-Lj21N6gqyzZgATY1dlGU5N27wKpVasnWzux3mLV9YcJKxVJWtnYUNXce3rPCAK6XvNemAmnVYB48jNY1H0tnmwvuIemdFclwHhKjfKL770c8PkvrRu4kkVk-BGkBIeFbiEZANJ7Q-A992hC8W-CxIr3Pxyr6Clo0EHq01-Qf8sHrtgVjl1SYkjrHXLgz1Sei_uWn2SCKnmFL48ziChvch3trjZuuI5st6h9msaPcBGvjlSu5UOkUNo5j9E73CEaup0u9wp5mnBtEnWZxXARbsU65jmuJr3OZvIL5WnWqYCmWkrumYIW5IceDt29nI1jqXVAr5myoapaWvB1QhV5yrzLNK8e7X1dJ7a17XR5__gO3H7Vunws)
## nginx Reverse Proxy
| Public IP/Ports          | <-> | I/F  | local IP  | I/F | <-> | I/F | local IP  | I/F     | <-> | I/F | local IP        |
|--------------------------|:---:|------|-----------|-----|:---:|-----|-----------|---------|:---:|-----|-----------------|
| 152.89.105.252:9000-9099 | <-> | ens3 | 10.0.20.1 | wg0 | <-> | wg0 | 10.0.20.2 | enp6s18 | <-> | ??? | 10.0.40.XXX:YYY |
| 152.89.105.252:9100-9199 | <-> | ens3 | 10.0.20.1 | wg0 | <-> | wg0 | 10.0.20.3 | enp6s18 | <-> | ??? | 10.0.30.XXX:YYY |

## mistakes:
times locked out: 4
system fuck ups: 3 

## Proxy Configuration
### VM Level Layout:
#### VPS:
| Service   | Incoming Interface | Outgoing Interface |
|-----------|--------------------|--------------------|
| MGMT      | ens3               | ens3               |
| Webserver | ens3               | ens3               |
| VPN       | ens3               | wg0                |
|           | wg0                | ens3               |
#### WGCF:
| Service | Incoming Interface | Outgoing Interface |
|---------|--------------------|--------------------|
| MGMT    | enp6s18            | enp6s18            |
| VPN in  | enp6s18            | wg0                |
| VPN out | wg0                | enp6s18            |
#### WGCG:
| Service | Incoming Interface | Outgoing Interface |
|---------|--------------------|--------------------|
| MGMT    | enp6s18            | enp6s18            |
| VPN in  | enp6s18            | wg0                |
| VPN out | wg0                | enp6s18            |
#### VM:
| Service | Incoming Interface | Outgoing Interface |
|---------|--------------------|--------------------|
| Any     | enp6s18            | enp6s18            |
 
### Routing Tables:
#### VPS (on VM level):
##### VPN:
| Service   | Source CN   | S_IF | SOURCE_IP | Destination CN | DESTINATION_IP | D_IF | PROTOCOL | S_PORT | D_PORT | Chain       |
|-----------|-------------|------|-----------|----------------|----------------|------|----------|--------|--------|-------------|
| VPN       | User        | ens3 | 0.0.0.0/0 | VPS            |                | wg0  | UDP      |        | 51820  | FORWARD     |
| VPN       | VPS         | wg0  |           | User           | 0.0.0.0/0      | ens3 | UDP      | 51820  |        | FORWARD     |
| VPN       | VPS         | wg0  |           | WG-Client-F    | 10.0.20.2      | wg0  | UDP      | 51820  | 36906  | POSTROUTING |
| VPN       | WG-Client-F | wg0  | 10.0.20.2 | VPS            |                | wg0  | UDP      | 36906  | 51820  | POSTROUTING |
##### VPS:
| Service   | Source CN   | S_IF | SOURCE_IP | Destination CN | DESTINATION_IP | D_IF | PROTOCOL | S_PORT | D_PORT | Chain       |
|-----------|-------------|------|-----------|----------------|----------------|------|----------|--------|--------|-------------|
| MGMT      | MGMT        | ens3 | 0.0.0.0/0 | VPS            |                | ens3 | TCP      | 22     | 22     | INPUT       |
| Webserver | User        | ens3 | 0.0.0.0/0 | VPS            |                | ens3 | TCP      |        | 80     | INPUT       |
| Webserver | User        | ens3 | 0.0.0.0/0 | VPS            |                | ens3 | TCP      |        | 443    | INPUT       |
| DNS       | CF-DNS      | ens3 | 8.8.8.8   | VPS            |                | ens3 | UDP      |        | 53     | INPUT       |
| MGMT      | VPS         | ens3 |           | MGMT           | 0.0.0.0/0      | ens3 | TCP      | 22     | 22     | OUTPUT      |
| Webserver | VPS         | ens3 |           | User           | 0.0.0.0/0      | ens3 | TCP      | 80     |        | OUTPUT      |
| Webserver | VPS         | ens3 |           | User           | 0.0.0.0/0      | ens3 | TCP      | 443    |        | OUTPUT      |
| DNS       | VPS         | ens3 |           | CF-DNS         | 8.8.8.8        | ens3 | UDP      | 53     |        | OUTPUT      |

#### Wireguard Client on Fileserver (on VM level): 
| Service | Source CN   | S_IF    | SOURCE_IP      | Destination CN | DESTINATION_IP | D_IF    | PROTOCOL | S_PORT | D_PORT | Chain       |
| --------| ----------- |---------| -------------- | -------------- | -------------- |---------| -------- | ------ | ------ | ----------- |
| MGMT    | MGMT        | enp6s18 | 10.0.1.0/24    | WG-Client-F    |                | enp6s18 | TCP      |        |        | INPUT       |
| VPN     | WG-Client-F | wg0     |                | FSVM           | 10.0.40.0/24   | enp6s18 | TCP      |        |        | FORWARD     |
| VPN     | FSVM        | enp6s18 | 10.0.40.0/24   | WG-Client-F    |                | wg0     | TCP      |        |        | FORWARD     |
| MGMT    | WG-Client-F | enp6s18 |                | MGMT           | 10.0.1.0/24    | enp6s18 | TCP      |        |        | OUTPUT      |
| VPN     | WG-Client-F | wg0     |                | VPS            | 152.89.105.252 | wg0     | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | VPS         | wg0     | 152.89.105.252 | WG-Client-F    |                | wg0     | UDP      | 51820  | 51820  | POSTROUTING |

#### Wireguard Client on Gameserver (on VM level):
| Service | Source CN   | S_IF    | SOURCE_IP      | Destination CN | DESTINATION_IP | D_IF    | PROTOCOL | S_PORT | D_PORT | Chain       |
|---------|-------------|---------|----------------|----------------|----------------|---------|----------|--------|--------|-------------|
| MGMT    | MGMT        | enp6s18 | 10.0.1.0/24    | WG-Client-G    |                | enp6s18 | TCP      |        |        | INPUT       |
| VPN     | WG-Client-G | wg0     |                | GSVM           | 10.0.30.0/24   | enp6s18 | TCP      |        |        | FORWARD     |
| VPN     | GSVM        | enp6s18 | 10.0.30.0/24   | WG-Client-G    |                | enp6s18 | TCP      |        |        | FORWARD     |
| MGMT    | WG-Client-G | enp6s18 |                | MGMT           | 10.0.1.0/24    | enp6s18 | TCP      |        |        | OUTPUT      |
| VPN     | VPS         | wg0     | 152.89.105.252 | WG-Client-G    |                | wg0     | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | WG-Client-G | wg0     |                | VPS            | 152.89.105.252 | wg0     | UDP      | 51820  | 51820  | POSTROUTING |


#### VMs on Fileserver (in PVE GUI):
| Service | Source CN   | SOURCE_IP   | Destination CN | DESTINATION_IP | PROTOCOL | S_PORT | D_PORT | Chain  |
| --------| ----------- | ----------- | -------------- | -------------- | -------- | ------ | ------ | ------ |
| VPN     | FSVM        |             | WG-Client-F    | 10.0.40.153    | TCP      |        |        | OUTPUT |
| MGMT    | FSVM        |             | MGMT           | 10.0.1.0/24    | TCP      |        |        | OUTPUT |
| DNS     | FSVM        |             | CF-DNS         | 8.8.8.8        | UDP      | 53     |        | OUTPUT |
| VPN     | WG-Client-F | 10.0.40.153 | FSVM           |                | TCP      |        |        | INPUT  |
| MGMT    | MGMT        | 10.0.1.0/24 | FSVM           |                | TCP      |        |        | INPUT  |
| DNS     | CF-DNS      | 8.8.8.8     | FSVM           |                | UDP      |        | 53     | INPUT  |
	
#### VMs on Gameserver (in PVE GUI):
| Service | Source CN   | SOURCE_IP   | Destination CN | DESTINATION_IP | PROTOCOL | S_PORT | D_PORT | Chain  |
| --------| ----------- | ----------- | -------------- | -------------- | -------- | ------ | ------ | ------ |
| VPN     | GSVM        |             | WG-Client-G    | 10.0.30.154    | TCP      |        |        | OUTPUT |
| MGMT    | GSVM        |             | MGMT           | 10.0.1.0/24    | TCP      |        |        | OUTPUT |
| DNS     | GSVM        |             | CF-DNS         | 8.8.8.8        | UDP      | 53     |        | OUTPUT |
| VPN     | WG-Client-G | 10.0.30.154 | GSVM           |                | TCP      |        |        | INPUT  |
| MGMT    | MGMT        | 10.0.1.0/24 | GSVM           |                | TCP      |        |        | INPUT  |
| DNS     | CF-DNS      | 8.8.8.8     | GSVM           |                | UDP      |        | 53     | INPUT  |


### Routing Commands per Chain:
#### INPUT:	
```bash
sudo iptables -A INPUT -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```
#### FORWARD:
```bash
sudo iptables -A FORWARD -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -j ACCEPT
```
#### OUTPUT:
```bash
sudo iptables  -A OUTPUT -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```
#### NAT (PRE-&POSTROUTING):
```bash
sudo iptables -t nat -A PREROUTING -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -j DNAT --to-destination $NEW_DESTINATION
sudo iptables -t nat -A POSTROUTING -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -j MASQUERADE
```


### commands:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install iptables-persistent
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo nano /etc/iptables/rules.v4
```

#### VPS:
```bash

```
#### WGCF:
```bash


```
#### WGCG:
```bash


```







## ToDo
- make iptables config permanent (https://askubuntu.com/questions/311053/how-to-make-ip-forwarding-permanent)
	```bash
	sudo iptables-save > /etc/iptables/rules.v4
	```

## \[deprecated\]
### Specific commands (NAT & Masquerade with DNAT)
(Port ranges with `First_Port:Last_Port`):

#### SSH from MGMT VLAN to WGCF and WGCG:
- `sudo iptables -A INPUT -p tcp -s 10.0.1.0/24 --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT`
- `sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#### SSH to VPS:
- `sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT`
- `sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT`
#### General:
- `sudo iptables -t nat -A PREROUTING -p $PROTOCOL -s $SOURCE_IP --dport $S_PORT -j DNAT --to-destination $DESTINATION_IP:$D_PORT`
- `sudo iptables -t nat -A POSTROUTING -p $PROTOCOL -d $DESTINATION_IP --dport $D_PORT -j MASQUERADE`
#### VM's:
- default Policy:
	- `sudo iptables -P INPUT DROP`
	- `sudo iptables -P OUTPUT ACCEPT`
- loopback Interface:
	- `sudo iptables -A INPUT -i lo -j ACCEPT`
	- `sudo iptables -A OUTPUT -o lo -j ACCEPT`
- already established connections:
	- `sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`
- outgoing connections:
	- `sudo iptables -A OUTPUT -j ACCEPT`
- incoming connections (get details from tables above):
	- `sudo iptables -A INPUT -p $PROTOCOL --dport $D_PORT -j ACCEPT`
### Possible commands (only Forward)
#### VPS:
```bash
# Allow traffic from VPS to any destination
sudo iptables -A FORWARD -s 152.89.105.252 -j ACCEPT

# Allow HTTP, HTTPS, and Wireguard (UDP 51820) traffic from VPS to any destination
sudo iptables -A FORWARD -s 152.89.105.252 -p tcp --dport 80:443 -j ACCEPT
sudo iptables -A FORWARD -s 152.89.105.252 -p udp --dport 51820 -j ACCEPT

# Allow SSH traffic from VPS to Management (MGMT) interface
sudo iptables -A FORWARD -s 152.89.105.252 -p tcp --dport 22 -d 10.0.10.0/24 -j ACCEPT

# Allow traffic from any source to VPS for HTTP, HTTPS, and Wireguard
sudo iptables -A FORWARD -d 152.89.105.252 -p tcp --dport 80:443 -j ACCEPT
sudo iptables -A FORWARD -d 152.89.105.252 -p udp --dport 51820 -j ACCEPT
```
#### Wireguard Client on Fileserver:
```bash
# Allow Wireguard and TCP traffic from WG-Client-F to VPS and FSVM
sudo iptables -A FORWARD -s 10.0.20.2 -p udp --dport 51820 -j ACCEPT
sudo iptables -A FORWARD -s 10.0.20.2 -p tcp --dport 80:443 -j ACCEPT

# Allow TCP traffic from WG-Client-F to MGMT interface
sudo iptables -A FORWARD -s 10.0.20.2 -p tcp --dport 22 -d 10.0.10.0/24 -j ACCEPT

# Allow traffic from VPS to WG-Client-F for Wireguard
sudo iptables -A FORWARD -s 152.89.105.252 -p udp --dport 51820 -d 10.0.20.2 -j ACCEPT

# Allow traffic from FSVM to WG-Client-F
sudo iptables -A FORWARD -s 10.0.40.0/24 -p tcp --dport 80:443 -d 10.0.20.2 -j ACCEPT
```
#### Wireguard Client on Gameserver:
```bash
# Allow Wireguard and TCP traffic from WG-Client-G to VPS and GSVM
sudo iptables -A FORWARD -s 10.0.20.3 -p udp --dport 51820 -j ACCEPT
sudo iptables -A FORWARD -s 10.0.20.3 -p tcp --dport 80:443 -j ACCEPT

# Allow TCP traffic from WG-Client-G to MGMT interface
sudo iptables -A FORWARD -s 10.0.20.3 -p tcp --dport 22 -d 10.0.10.0/24 -j ACCEPT

# Allow traffic from VPS to WG-Client-G for Wireguard
sudo iptables -A FORWARD -s 152.89.105.252 -p udp --dport 51820 -d 10.0.20.3 -j ACCEPT

# Allow traffic from GSVM to WG-Client-G
sudo iptables -A FORWARD -s 10.0.30.0/24 -p tcp --dport 80:443 -d 10.0.20.3 -j ACCEPT
```


## Mermaid Graphs
### Ping not working
```
flowchart TD
    A[ping doesn't work]
    B[cat /proc/sys/net/ipv4/ip_forward]
    D[sudo sysctl -w net.ipv4.ip_forward=1]
    C[sudo wg]
    E[wg-quick up wg0]
    F[still not working]
    G[cry]
    H[still doesn't work]

    A --> B
    B --> | 1 | F
    B --> | 0 | D
    D --> F
    F --> C
    C --> | interface: wg0... | H
    C --> | nothing | E
    E --> H
    H --> G
```

| VLAN ID |     Subnet     | Common Name |
|---------|----------------|-------------| 
|     1   |   10.0.1.0/24  | MGMT        |
|    20   |  10.0.20.0/24  | Wireguard   |
|    30   |  10.0.30.0/24  | Gameserver  |
|    40   |  10.0.40.0/24  | Fileserver  |

| Public IP:Ports           | <-> | I/F  | WG-Server IP | I/F | <-> | I/F | WG-Client IP | I/F     | <-> | I/F | VM IP           |
|---------------------------|:---:|------|--------------|-----|:---:|-----|--------------|---------|:---:|-----|-----------------|
| 152.XXX.XXX.XXX:9000-9099 | <-> | ens3 | 10.0.20.1    | wg0 | <-> | wg0 | 10.0.20.3    | enp6s18 | <-> | XXX | 10.0.30.XXX:XXX |
| 152.XXX.XXX.XXX:9100-9199 | <-> | ens3 | 10.0.20.1    | wg0 | <-> | wg0 | 10.0.20.2    | enp6s18 | <-> | XXX | 10.0.40.XXX:XXX |