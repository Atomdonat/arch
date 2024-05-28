**Allgemein gilt ‘positive default policy’ (alles was nicht in iptables ist, wird gedroppt)**

  
- enable IP-forwarding: `echo 1 > /proc/sys/net/ipv4/ip_forward`
- get current config: `iptables -filter -L`
- initialize firewall with negative default policy:
	- drop incoming traffic: `iptables -P INPUT DROP`
	- drop forwarding traffic: `iptables -P FORWARD DROP`
	- drop outgoing traffic: `iptables -P OUTPUT DROP`
- get current config: `iptables -filter -L`
	Chain INPUT (policy DROP)
	Chain FORWARD (policy DROP)
	Chain OUTPUT (policy DROP)
- plan Traffic rules:
  | Device      | Source CN      | Source IP      | Destination CN | Destination IP | TCP/UDP | Protocol2 | s-port | d-port  | Chain   | Action | Interface |
| ----------- | -------------- | -------------- | -------------- | -------------- | ------- | --------- | ------ | ------- | ------- | ------ | --------- |
| VPS         | VPS            | 152.89.105.252 | Any            | Any            | TCP     | HTTP      | 80     | Any     | FORWARD | ACCEPT | eth0      |
| VPS         | 152.89.105.252 | Any            | Any            | TCP            | HTTPS   | 443       | Any    | FORWARD | ACCEPT  | eth0   |
| VPS         | 152.89.105.252 | Any            | Any            | UDP            | Any     | 51820     | Any    | FORWARD | ACCEPT  | wg0    |
| VPS         | 152.89.105.252 | WG-Client-F    | 10.0.60.2      | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
| VPS         | 152.89.105.252 | WG-Client-G    | 10.0.60.3      | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
| Any         | Any            | VPS            | 152.89.105.252 | TCP            | HTTP    | Any       | 80     | FORWARD | ACCEPT  | eth0   |
| Any         | Any            | VPS            | 152.89.105.252 | TCP            | HTTPS   | Any       | 443    | FORWARD | ACCEPT  | eth0   |
| Any         | Any            | VPS            | 152.89.105.252 | UDP            | Any     | Any       | 51820  | FORWARD | ACCEPT  | wg0    |
| WG-Client-F | 10.0.60.2      | VPS            | 152.89.105.252 | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
| WG-Client-G | 10.0.60.3      | VPS            | 152.89.105.252 | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
|             |                |                |                |                |         |           |        |         |         |        |           |
| WGCF        | WG-Client-F    | 10.0.60.2      | VPS            | 152.89.105.252 | UDP     | Any       | 51820  | 51820   | FORWARD | ACCEPT | wg0       |
| WG-Client-F | 10.0.60.2      | TrueNAS        | 10.0.40.151    | TCP            | Any     |           |        | FORWARD | ACCEPT  |        |
| WG-Client-F | 10.0.60.2      | Mailserver     | 10.0.40.???    | TCP            | Any     |           |        | FORWARD | ACCEPT  |        |
| WG-Client-F | 10.0.60.2      | Dashboard      | 10.0.40.???    | TCP            | HTTPS   |           | 443    | FORWARD | ACCEPT  |        |
| VPS         | 152.89.105.252 | WG-Client-F    | 10.0.60.2      | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
| TrueNAS     | 10.0.40.151    | WG-Client-F    | 10.0.60.2      | TCP            | Any     |           |        | FORWARD | ACCEPT  |        |
| Mailserver  | 10.0.40.???    | WG-Client-F    | 10.0.60.2      | TCP            | Any     |           |        | FORWARD | ACCEPT  |        |
| Dashboard   | 10.0.40.???    | WG-Client-F    | 10.0.60.2      | TCP            | HTTPS   | 443       |        | FORWARD | ACCEPT  |        |
|             |                |                |                |                |         |           |        |         |         |        |           |
| WGCG        | WG-Client-G    | 10.0.60.3      | VPS            | 152.89.105.252 | UDP     | Any       | 51820  | 51820   | FORWARD | ACCEPT | wg0       |
| WG-Client-G | 10.0.60.3      | PteroPanel     | 10.0.30.152    | TCP            | HTTPS   |           | 443    | FORWARD | ACCEPT  |        |
| WG-Client-G | 10.0.60.3      | PteroWings     | 10.0.30.155    | Both           | HTTP    |           | Any    | FORWARD | ACCEPT  |        |
| VPS         | 152.89.105.252 | WG-Client-G    | 10.0.60.3      | UDP            | Any     | 51820     | 51820  | FORWARD | ACCEPT  | wg0    |
| PteroPanel  | 10.0.30.152    | WG-Client-G    | 10.0.60.3      | TCP            | HTTPS   | 443       |        | FORWARD | ACCEPT  |        |
| PteroWings  | 10.0.30.155    | WG-Client-G    | 10.0.60.3      | Both           | HTTP    | Any       |        | FORWARD | ACCEPT  |        |
