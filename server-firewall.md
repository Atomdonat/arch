## pre-configuration
- install iptables: `sudo apt install iptables iutils-ping`
- enable IP-forwarding: `echo 1 > /proc/sys/net/ipv4/ip_forward`
- get current config: `iptables -filter -L`

### Wireguard
#### Server on VPS:
```bash
atomdonat@gnfproxy:~$ sudo wg
interface: wg0
  public key: 0wXbJyU3Sqf2tMk0qZsnwggwHjv8EHm+q/zKaQAOzEY=
  private key: (hidden)
  listening port: 51820

peer: VhyJeBNnKpDRjVCt3GP7dh+OSRsWfKwmQ2utK8kdBDE=
  endpoint: 134.147.24.39:36906
  allowed ips: 10.0.20.2/32

peer: AUik3EsHsaIk3NJLYkI+gONW2FmNtzgKLUQJodizOko=
  endpoint: 134.147.24.39:40381
  allowed ips: 10.0.20.3/32
```
#### Client on WGCG:
```bash
simon@wg-client-g:~$ sudo wg
interface: wg0
  public key: AUik3EsHsaIk3NJLYkI+gONW2FmNtzgKLUQJodizOko=
  private key: (hidden)
  listening port: 40381
  fwmark: 0xca6c

peer: 0wXbJyU3Sqf2tMk0qZsnwggwHjv8EHm+q/zKaQAOzEY=
  endpoint: 152.89.105.252:51820
  allowed ips: 10.0.20.0/24, 10.0.30.0/24
  latest handshake: 1 minute, 15 seconds ago
  transfer: 276 B received, 924 B sent
  persistent keepalive: every 25 seconds
```
#### Client on WGCF:
```bash
user@wireguard-client:~$ sudo wg
interface: wg0
  public key: VhyJeBNnKpDRjVCt3GP7dh+OSRsWfKwmQ2utK8kdBDE=
  private key: (hidden)
  listening port: 36906

peer: 0wXbJyU3Sqf2tMk0qZsnwggwHjv8EHm+q/zKaQAOzEY=
  endpoint: 152.89.105.252:51820
  allowed ips: 10.0.40.0/24, 10.0.20.0/24
  latest handshake: 2 seconds ago
  transfer: 92 B received, 180 B sent
  persistent keepalive: every 25 seconds
```
## Firewall Configuration
### VM Level Layout:
#### VPS:
| Service   | Incoming Interface | Outgoing Interface |
|-----------|--------------------|--------------------|
| MGMT      | ens3               | ens3               |
| Webserver | ens3               | ens3               |
| VPN in    | ens3               | wg0                |
| VPN out   | wg0                | ens3               |
#### WGCF:
| Service | Incoming Interface | Outgoing Interface |
|---------|--------------------|--------------------|
| MGMT    | new Interface      | new Interface      |
| VPN in  | enp6s18            | wg0                |
| VPN out | wg0                | enp6s18            |
#### WGCG:
| Service | Incoming Interface | Outgoing Interface |
|---------|--------------------|--------------------|
| MGMT    | new Interface      | new Interface      |
| VPN in  | enp6s18            | wg0                |
| VPN out | wg0                | enp6s18            |
#### VM:

 
### Routing Tables:
#### VPS (on VM level):
| Service   | Source CN   | SOURCE_IP      | Destination CN | DESTINATION_IP | PROTOCOL | S_PORT | D_PORT | Chain       |
| ----------| ----------- | -------------- | -------------- | -------------- | -------- | ------ | ------ | ----------- |
| MGMT      | VPS         |                | MGMT           | 0.0.0.0/0      | TCP      | 22     | 22     | OUTPUT      |
| Webserver | VPS         |                | User           | 0.0.0.0/0      | TCP      | 80     |        | OUTPUT      |
| Webserver | VPS         |                | User           | 0.0.0.0/0      | TCP      | 443    |        | OUTPUT      |
| VPN       | VPS         |                | User           | 0.0.0.0/0      | UDP      | 51820  |        | OUTPUT      |
| DNS       | VPS         |                | CF-DNS         | 8.8.8.8        | UDP      | 53     |        | OUTPUT      |
| VPN       | VPS         |                | WG-Client-F    | 10.0.20.2      | UDP      | 51820  | 51820  | POSTROUTING |
| VPN       | VPS         |                | WG-Client-G    | 10.0.20.3      | UDP      | 51820  | 51820  | POSTROUTING |
| MGMT      | MGMT        | 0.0.0.0/0      | VPS            |                | TCP      | 22     | 22     | INPUT       |
| Webserver | User        | 0.0.0.0/0      | VPS            |                | TCP      |        | 80     | INPUT       |
| Webserver | User        | 0.0.0.0/0      | VPS            |                | TCP      |        | 443    | INPUT       |
| VPN       | User        | 0.0.0.0/0      | VPS            |                | UDP      |        | 51820  | INPUT       |
| DNS       | CF-DNS      | 8.8.8.8        | VPS            |                | UDP      |        | 53     | INPUT       |
| VPN       | WG-Client-F | 10.0.20.2      | VPS            |                | UDP      | 51820  | 51820  | POSTROUTING |
| VPN       | WG-Client-G | 10.0.20.3      | VPS            |                | UDP      | 51820  | 51820  | POSTROUTING |

#### Wireguard Client on Fileserver (on VM level): 
| Service | Source CN   | SOURCE_IP      | Destination CN | DESTINATION_IP | PROTOCOL | S_PORT | D_PORT | Chain       |
| --------| ----------- | -------------- | -------------- | -------------- | -------- | ------ | ------ | ----------- |
| VPN     | WG-Client-F |                | VPS            | 152.89.105.252 | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | WG-Client-F |                | FSVM           | 10.0.40.0/24   | TCP      |        |        | OUTPUT      |
| MGMT    | WG-Client-F |                | MGMT           | 10.0.1.0/24    | TCP      |        |        | OUTPUT      |
| VPN     | VPS         | 152.89.105.252 | WG-Client-F    |                | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | FSVM        | 10.0.40.0/24   | WG-Client-F    |                | TCP      |        |        | INPUT       |
| MGMT    | MGMT        | 10.0.1.0/24    | WG-Client-F    |                | TCP      |        |        | INPUT       |

#### Wireguard Client on Gameserver (on VM level):
| Service | Source CN   | SOURCE_IP      | Destination CN | DESTINATION_IP | PROTOCOL | S_PORT | D_PORT | Chain       |
| --------| ----------- | -------------- | -------------- | -------------- | -------- | ------ | ------ | ----------- |
| VPN     | WG-Client-G |                | VPS            | 152.89.105.252 | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | WG-Client-G |                | GSVM           | 10.0.30.0/24   | TCP      |        |        | OUTPUT      |
| MGMT    | WG-Client-G |                | MGMT           | 10.0.1.0/24    | TCP      |        |        | OUTPUT      |
| VPN     | VPS         | 152.89.105.252 | WG-Client-G    |                | UDP      | 51820  | 51820  | POSTROUTING |
| VPN     | GSVM        | 10.0.30.0/24   | WG-Client-G    |                | TCP      |        |        | INPUT       |
| MGMT    | MGMT        | 10.0.1.0/24    | WG-Client-G    |                | TCP      |        |        | INPUT       |

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
#### VMS:
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
#### FORWARD:
```bash
sudo iptables -A FORWARD -s $SOURCE_IP -d $DESTINATION_IP -p $PROTOCOL --sport $S_PORT --dport $D_PORT -j ACCEPT
```