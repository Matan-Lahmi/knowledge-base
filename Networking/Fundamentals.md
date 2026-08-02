# Networking Fundamentals

## OSI Model

7 layers, how data actually gets from one machine to another. Mostly used to explain "at what level" a problem is.

1. Physical - cables, signals
2. Data Link - MAC addresses, switches
3. Network - IP addresses, routers
4. Transport - TCP/UDP, ports
5. Session - keeps a connection open between two apps
6. Presentation - encryption/encoding (SSL/TLS lives around here)
7. Application - HTTP, DNS, the stuff apps actually use

## TCP/IP Model

Simpler, 4-layer version people actually use day to day:
- Network Access (Physical+Data Link combined)
- Internet (IP)
- Transport (TCP/UDP)
- Application (HTTP, DNS, etc.)

## Layer 4 vs Layer 7

- **L4 (Transport)** - routes based on IP + port only, doesn't look at the actual content. Faster, dumber.
- **L7 (Application)** - reads the actual request (like the URL path, headers), can route based on that. Slower, smarter. This is what Traefik/nginx do when routing by hostname or path.

## IP - Internet Protocol

Address of a machine on a network.

- **Private IP** - only reachable inside a local network (10.x.x.x, 172.16-31.x.x, 192.168.x.x)
- **Public IP** - reachable from the whole internet

## NAT (Network Address Translation)

How a private IP "talks" to the internet at all. A private IP isn't routable on the public internet, so something has to translate it to a public IP and back. Your home router does this for every device on your home network - they all share one public IP.

### Internet Gateway vs NAT Gateway (AWS terms, worth knowing ahead of time)

- **Internet Gateway (IGW)** - lets resources with a public IP talk directly to the internet, both ways. Used for public subnets.
- **NAT Gateway** - lets resources with only a private IP reach out to the internet (for updates, pulling images, etc.), but nothing from the internet can initiate a connection back in. Used for private subnets. Costs money per hour + per GB, unlike an IGW.

## CIDR & Subnets

CIDR notation = IP + how many bits are the "network part".
Smaller number after / = bigger network (more hosts). /16 > /24 in size.

```
192.168.1.0/24
```
`/24` = first 24 bits are network, last 8 bits are for hosts = 256 total addresses.
- On-Prem: 254 usable (2 reserved: Network ID, Broadcast).
- AWS VPC: 251 usable (AWS reserves 5 IPs: Network ID, VPC Router, AWS DNS, Future use, Broadcast).

`/32` - single IP address
Great for AWS Security Groups and firewall rules to allow exactly one machine. 
Note: You CANNOT create a /32 *subnet*. A subnet always requires routing overhead. 
- Minimum AWS subnet is /28 (16 IPs).
- Minimum On-Prem subnet is /30 (4 IPs: Network, Gateway, Endpoint, Broadcast).

## Ports

A port is how a machine tells apart different services running on the same IP.

```bash
sudo ss -tulpn
```

- `LISTEN` status = something is actively waiting for connections on that port
- Common ports to know: 22 (SSH), 80 (HTTP), 443 (HTTPS), 53 (DNS)

## nc (netcat) - port connectivity check

Checks if a specific port on a host is actually reachable, without needing the full app (browser, ssh client, etc.) to test it.

```bash
nc -zv <host> <port>
```
- `z` - just scan/check, don't actually send data
- `v` - verbose, print the result

## ICMP / ping / Traceroute

**ping** - sends ICMP packets, checks if a host is reachable and how long it takes.
```bash
ping <host>
```

**TTL (Time To Live)** - counter on a packet, drops by 1 at every router hop. Hits 0 = packet dropped. Used to prevent infinite loops in routing, and traceroute abuses this on purpose.

**traceroute** - shows every hop (router) between you and the destination.
```bash
traceroute <host>
traceroute -I <host>     # use ICMP instead of UDP
traceroute -T <host>      # use TCP
traceroute -n <host>       # don't resolve hostnames, just show IPs (faster)
```

## whois

Who owns a domain / IP range.
```bash
whois <domain>
```

## DNS

Translates domain names to IP addresses.

- **hosts file** - local override, checked before any DNS server (`/etc/hosts`)
- **cache** - your machine/resolver remembers answers for a while (TTL), so it doesn't ask again every time
- **Recursive resolver** - the server that does the actual lookup work for you (like your ISP's DNS, or 8.8.8.8)
- **Root servers** - top of the DNS tree, know where to find the TLD servers
- **TLD (Top Level Domain)** - `.com`, `.io`, etc. servers, know where to find the authoritative server for a specific domain
- **Authoritative server** - has the actual real answer for a domain

Flow:
You -> Recursive Resolver -> Root -> TLD -> Authoritative -> Real IP

```bash
dig <domain>
nslookup <domain>
```

### Record types
- **A record** - domain → IPv4 address
- **CNAME (alias)** - domain → another domain name (not an IP directly)

## TCP vs UDP

- **TCP** - reliable, ordered, has a handshake, retransmits lost packets. Slower but safe. (HTTP, SSH)
- **UDP** - no handshake, no guarantee of delivery or order, just fire and forget. Faster. (DNS queries, video streaming)

## HTTP vs HTTPS

- **HTTP** - plain text, anyone on the network path can read it
- **HTTPS** - HTTP wrapped in TLS encryption.

### Status Codes
- **2xx** - success (200 OK)
- **3xx** - redirect (301, 302)
- **4xx** - client error (404 Not Found, 401 Unauthorized, 403 Forbidden)
- **5xx** - server error (500, 502 Bad Gateway, 503 Service Unavailable)

### curl - testing HTTP from the command line
```bash
curl <url>
curl -I <url>            # headers only, no body
curl -v <url>             # verbose, shows the full request/response including handshake
curl -o file.html <url>    # save response to a file
```
