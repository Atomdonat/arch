## pre-configuration
- enable IP-forwarding: `echo 1 > /proc/sys/net/ipv4/ip_forward`
- get current config: `iptables -filter -L`
### plan Traffic rules:
- (Wireguard Server on) VPS:
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| VPS    | VPS         | 152.89.105.252    | Any            | Any               | TCP         | HTTP      | 80        | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | Any            | Any               | TCP         | HTTPS     | 443       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | Any            | Any               | UDP         | Any       | 51820     | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-F    | 10.0.60.2         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-G    | 10.0.60.3         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | MGMT           | Any               | TCP         | SSH       | 22        | 22         | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | TCP         | HTTP      | Any       | 80         | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | TCP         | HTTPS     | Any       | 443        | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | UDP         | Any       | Any       | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | MGMT        | Any               | VPS            | 152.89.105.252    | TCP         | SSH       | 22        | 22         | FORWARD | ACCEPT |

- Wireguard Client on Fileserver: 
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| WGCF   | WG-Client-F | 10.0.60.2         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | TrueNAS        | 10.0.40.151       | TCP         | Any       |           |            | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | Mailserver     | 10.0.40.???       | TCP         | Any       |           |            | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | Dashboard      | 10.0.40.???       | TCP         | HTTPS     |           | 443        | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.3         | MGMT           | 10.0.10.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-F    | 10.0.60.2         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | TrueNAS     | 10.0.40.151       | WG-Client-F    | 10.0.60.2         | TCP         | Any       |           |            | FORWARD | ACCEPT |
	|        | Mailserver  | 10.0.40.???       | WG-Client-F    | 10.0.60.2         | TCP         | Any       |           |            | FORWARD | ACCEPT |
	|        | Dashboard   | 10.0.40.???       | WG-Client-F    | 10.0.60.2         | TCP         | HTTPS     | 443       |            | FORWARD | ACCEPT |
	|        | MGMT        | 10.0.10.0/24      | WG-Client-F    | 10.0.60.3         | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |

- Wireguard Client on Gameserver:
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| WGCG   | WG-Client-G | 10.0.60.3         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | PteroPanel     | 10.0.30.152       | TCP         | HTTPS     |           | 443        | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | PteroWings     | 10.0.30.155       | Both        | HTTP      |           | Any        | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | MGMT           | 10.0.10.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-G    | 10.0.60.3         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | PteroPanel  | 10.0.30.152       | WG-Client-G    | 10.0.60.3         | TCP         | HTTPS     | 443       |            | FORWARD | ACCEPT |
	|        | PteroWings  | 10.0.30.155       | WG-Client-G    | 10.0.60.3         | Both        | HTTP      | Any       |            | FORWARD | ACCEPT |
	|        | MGMT        | 10.0.10.0/24      | WG-Client-G    | 10.0.60.3         | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	
- *every IP-Tables config for affected VM's can be extracted from their equivalent WG Client
	
### better? Version (+future proof; -security?):
- VPS:
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| VPS    | VPS         | 152.89.105.252    | Any            | Any               | TCP         | HTTP      | 80        | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | Any            | Any               | TCP         | HTTPS     | 443       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | Any            | Any               | UDP         | Any       | 51820     | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Clients     | 10.0.60.0/24      | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | MGMT           | Any               | TCP         | SSH       | 22        | 22         | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | TCP         | HTTP      | Any       | 80         | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | TCP         | HTTPS     | Any       | 443        | FORWARD | ACCEPT |
	|        | Any         | Any               | VPS            | 152.89.105.252    | UDP         | Any       | Any       | 51820      | FORWARD | ACCEPT |
	|        | WG-Clients  | 10.0.60.0/24      | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | MGMT        | Any               | VPS            | 152.89.105.252    | TCP         | SSH       | 22        | 22         | FORWARD | ACCEPT |

- Wireguard Client on Fileserver: 
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| WGCF   | WG-Client-F | 10.0.60.2         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | FSVM           | 10.0.40.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | WG-Client-F | 10.0.60.2         | MGMT           | 10.0.10.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-F    | 10.0.60.2         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | FSVM        | 10.0.40.0/24      | WG-Client-F    | 10.0.60.2         | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | MGMT        | 10.0.10.0/24      | WG-Client-F    | 10.0.60.2         | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |

- Wireguard Client on Gameserver:
	| Device | Source CN   | $(SOURCE_IP)      | Destination CN | $(DESTINATION_IP) | $(PROTOCOL) | Protocol2 | $(S_PORT) | $(D_PORT)  | Chain   | Action |
	| -------| ----------- | ----------------- | -------------- | ----------------- | ----------- | --------- | --------- | ---------- | ------- | ------ |
	| WGCG   | WG-Client-G | 10.0.60.3         | VPS            | 152.89.105.252    | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | GSVM           | 10.0.30.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | WG-Client-G | 10.0.60.3         | MGMT           | 10.0.10.0/24      | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | VPS         | 152.89.105.252    | WG-Client-G    | 10.0.60.3         | UDP         | Any       | 51820     | 51820      | FORWARD | ACCEPT |
	|        | GSVM        | 10.0.30.0/24      | WG-Client-G    | 10.0.60.3         | TCP & UDP   | Any       | Any       | Any        | FORWARD | ACCEPT |
	|        | MGMT        | 10.0.10.0/24      | WG-Client-G    | 10.0.60.3         | TCP         | Any       | Any       | Any        | FORWARD | ACCEPT |
	
- *every IP-Tables config for affected VM's can be extracted from their equivalent WG Client

### Specific commands (NAT & Masquerade with DNAT)
(Port ranges with `First_Port:Last_Port`):

- SSH from MGMT VLAN to WGCF and WGCG:
	- `sudo iptables -A INPUT -p tcp -s 10.0.1.0/24 --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT`
	- `sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
- SSH to VPS:
	- `sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT`
	- `sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT`
- General:
	- `sudo iptables -t nat -A PREROUTING -p $(PROTOCOL) -s $(SOURCE_IP) --dport $(S_PORT) -j DNAT --to-destination $(DESTINATION_IP):$(D_PORT)`
	- `sudo iptables -t nat -A POSTROUTING -p $(PROTOCOL) -d $(DESTINATION_IP) --dport $(D_PORT) -j MASQUERADE`
- VMS:
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
		- `sudo iptables -A INPUT -p $(PROTOCOL) --dport $(D_PORT) -j ACCEPT`
### Possible commands (only Forward)
- VPS:
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
- Wireguard Client on Fileserver:
	```bash
	# Allow Wireguard and TCP traffic from WG-Client-F to VPS and FSVM
	sudo iptables -A FORWARD -s 10.0.60.2 -p udp --dport 51820 -j ACCEPT
	sudo iptables -A FORWARD -s 10.0.60.2 -p tcp --dport 80:443 -j ACCEPT
	
	# Allow TCP traffic from WG-Client-F to MGMT interface
	sudo iptables -A FORWARD -s 10.0.60.2 -p tcp --dport 22 -d 10.0.10.0/24 -j ACCEPT
	
	# Allow traffic from VPS to WG-Client-F for Wireguard
	sudo iptables -A FORWARD -s 152.89.105.252 -p udp --dport 51820 -d 10.0.60.2 -j ACCEPT
	
	# Allow traffic from FSVM to WG-Client-F
	sudo iptables -A FORWARD -s 10.0.40.0/24 -p tcp --dport 80:443 -d 10.0.60.2 -j ACCEPT
	```
- Wireguard Client on Gameserver:
	```bash
	# Allow Wireguard and TCP traffic from WG-Client-G to VPS and GSVM
	sudo iptables -A FORWARD -s 10.0.60.3 -p udp --dport 51820 -j ACCEPT
	sudo iptables -A FORWARD -s 10.0.60.3 -p tcp --dport 80:443 -j ACCEPT
	
	# Allow TCP traffic from WG-Client-G to MGMT interface
	sudo iptables -A FORWARD -s 10.0.60.3 -p tcp --dport 22 -d 10.0.10.0/24 -j ACCEPT
	
	# Allow traffic from VPS to WG-Client-G for Wireguard
	sudo iptables -A FORWARD -s 152.89.105.252 -p udp --dport 51820 -d 10.0.60.3 -j ACCEPT
	
	# Allow traffic from GSVM to WG-Client-G
	sudo iptables -A FORWARD -s 10.0.30.0/24 -p tcp --dport 80:443 -d 10.0.60.3 -j ACCEPT
	```
## ToDo
- NAT & Masquerade with DNAT **or** Forwarding
	1. Scenario
		- NAT: VPS <-> WGCF, VPS <-> WGCG
		- Forward: WGCF <-> VMs, WGCG <-> VMs
	2. Scenario
		- only NAT, no Forward
	3. Scenario
		- only Forward, no NAT
	5. Scenario
		- 
	6. Scenario
		- 