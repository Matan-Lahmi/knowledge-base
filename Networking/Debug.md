# Networking - Debug Cheatsheet

Quick commands, no explanations here - see [fundamentals.md](Fundamentals.md) / [security.md](Security.md) for the concepts behind them.

```bash
# connectivity
ping <host>
traceroute <host>
traceroute -I <host>
traceroute -T <host>
traceroute -n <host>

# DNS
dig <domain>
nslookup <domain>
whois <domain>

# ports
sudo ss -tulpn
nc -zv <host> <port>

# HTTP
curl <url>
curl -I <url>
curl -v <url>
curl -o file.html <url>
```
