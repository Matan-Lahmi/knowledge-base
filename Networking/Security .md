# Networking Security

## Firewalls (Security Groups)

A firewall decides what traffic is allowed in/out based on rules (IP, port, protocol). "Security Group" is just AWS's name for a firewall attached to an instance/service.

### Stateful vs Stateless

- **Stateful** - remembers a connection once it's allowed in. If you allow inbound traffic on a port, the response traffic going back out is automatically allowed too, no separate rule needed. (Most firewalls, Security Groups)
- **Stateless** - every packet is checked against the rules independently, in both directions. Need explicit rules for both inbound and outbound. (Network ACLs in AWS)

## Reverse Proxy

Sits in front of your actual servers, and forwards incoming requests to the right one — based on hostname, path, etc. Client only ever talks to the reverse proxy, never directly to the backend. Traefik/nginx are reverse proxies.

Also usually where TLS termination happens (see below) - so the actual backend servers don't need to deal with certificates at all.

## SSL/TLS

Encrypts traffic between client and server so nobody in the middle can read it.

### Asymmetric Encryption

Two keys, mathematically linked, but you can't get one from the other:
- **Public key** - shared with everyone, used to encrypt / verify
- **Private key** - kept secret, used to decrypt / sign

Anything encrypted with the public key can only be decrypted with the matching private key (and vice versa for signing).

### Certificates

A certificate is basically: "here's my public key, and a trusted third party (CA) confirms this key really belongs to this domain." Without that trust chain, anyone could claim to be any website.

### The Handshake (simplified)

1. Client connects, server sends its certificate (public key)
2. Client checks the certificate is valid / signed by a CA it trusts
3. Both sides agree on a symmetric key for the actual session (asymmetric encryption is slow, so it's only used to set this up)
4. From here on, traffic is encrypted with the faster symmetric key

### Let's Encrypt & Certbot

Let's Encrypt is a free, automated CA. Certbot (or Traefik's built-in ACME client) talks to it to get and renew certificates automatically, instead of buying/renewing manually.

### HTTP-01 vs DNS-01 Challenge

Both are ways Let's Encrypt verifies you actually own the domain before issuing a cert.

- **HTTP-01** - Let's Encrypt gives you a token, you host it at a specific URL on port 80, Let's Encrypt checks it from the outside. Needs port 80 open. Can't be used for wildcard certs.
- **DNS-01** - Let's Encrypt gives you a value, you add it as a TXT record in your DNS. No need for port 80 open at all. Required for wildcard certs (`*.domain.com`). Needs API access to your DNS provider.

(Traefik-specific note: where it actually stores the certs it gets - `acme.json` - is in the traefik folder, not here, since that's implementation-specific)
