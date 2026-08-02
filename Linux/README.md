# Linux

Commands and configs I use for server administration and troubleshooting. Not a tutorial — a reference to come back to.

## Contents
- [Useful Utility Commands](#useful-utility-commands)
- [User Management & Sudo](#user-management--sudo)
- [Permissions & Ownership](#permissions--ownership)
- [Package Management](#package-management)
- [Process Management & Signals](#process-management--signals)
- [System Monitoring](#system-monitoring)
- [Service Management (systemd)](#service-management-systemd)
- [Logs](#logs)
- [Local Port Check](#local-port-check)
- [SSH Hardening](#ssh-hardening)
- [Firewall (UFW)](#firewall-ufw)
- [Fail2Ban](#fail2ban)
- [Cron / Scheduling](#cron--scheduling)
- [Keeping Processes Alive After Disconnect](#keeping-processes-alive-after-disconnect)

---

## Useful Utility Commands

```bash
# tar - archive/extract, always forget the flags
tar -czf archive.tar.gz /path/to/dir     # compress
tar -xzf archive.tar.gz                   # extract
tar -tf archive.tar.gz                    # list contents without extracting

# grep - search text
grep "pattern" file.txt
grep -r "pattern" /some/dir      # recursive
grep -i "pattern" file.txt        # case-insensitive
grep -n "pattern" file.txt         # show line numbers
grep -v "pattern" file.txt          # invert - show lines that don't match
```

---

## User Management & Sudo

```bash
sudo adduser <username>
sudo usermod -aG sudo <username>    # gives sudo rights
groups <username>                     # check
su - <username>                        # switch user
```

---

## Permissions & Ownership

r=4 ,w=2, x=1
owner,gruop,other

```bash
ls -l                              # see permissions
chmod 755 file                     # rwxr-xr-x
chmod +x file                       # make executable
chown user:group file
chown -R user:group dir/             # recursive
```

---

## Package Management

```bash
sudo apt update       # refresh package list
sudo apt upgrade       # upgrade installed packages
sudo apt install <pkg>
sudo apt remove <pkg>
```

---

## Process Management & Signals

```bash
ps aux | grep <name>

kill <PID>        # SIGTERM (15) - polite, ask process to shut down clean
kill -9 <PID>       # SIGKILL (9) - force kill, no cleanup
pkill <name>         # kill by process name
```

SIGTERM first, always. SIGKILL only if the process doesn't respond.

---

## System Monitoring

```bash
top
htop                  # nicer version of top

free -h                 # memory usage
df -h                    # disk usage per mount
du -sh <dir>               # size of a specific directory
```

---

## Service Management (systemd)

```bash
sudo systemctl status <service>
sudo systemctl start <service>
sudo systemctl stop <service>
sudo systemctl restart <service>
sudo systemctl enable <service>    # auto-start on boot
sudo systemctl disable <service>
```

---

## Logs

```bash
journalctl -u <service>            # logs for a specific service
journalctl -u <service> -f          # follow live
journalctl -b                         # since last boot

tail -f /var/log/syslog               # general system log
tail -f /var/log/auth.log              # auth attempts (check this with fail2ban)
grep "Failed password" /var/log/auth.log
```

---

## Local Port Check

```bash
sudo ss -tulpn
```

flags I actually use:
- `t` - tcp
- `u` - udp
- `l` - listening sockets only
- `p` - show process name/PID
- `n` - numeric (don't resolve hostnames, faster)

---

## SSH Hardening

Key-based auth first, then disable password/root login — in that order, so you don't lock yourself out.

```bash
# 1. on my own machine, generate a key 
ssh-keygen -t ed25519 -C "email@example.com" (chmod 600 ~/.ssh/id_ed25519)

# 2. copy the public key to the server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server_ip

# 3. test that key login works BEFORE disabling password auth
ssh user@server_ip
```

Only after key login is confirmed working, edit `/etc/ssh/sshd_config` on the server:
```
PermitRootLogin no
PasswordAuthentication no
```
```bash
sudo systemctl restart ssh
```

---

## Firewall (UFW)

Rules first, `enable` last — otherwise you can lock yourself out over SSH.

```bash
sudo ufw status verbose

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable          # only after the rules above are set

sudo ufw delete allow 80/tcp
sudo ufw reset
```

---

## Fail2Ban

Bans an IP after repeated failed login attempts (mainly SSH).

```bash
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

in `jail.local`, `[sshd]` section:
```
[sshd]
enabled = true
maxretry = 5
bantime = 3600
```

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip <IP>
```

---

## Cron / Scheduling

```bash
crontab -e
crontab -l
```

format: `minute hour day-of-month month day-of-week`
```
0 2 * * *      /path/script.sh    # every day at 02:00
*/15 * * * *    /path/script.sh     # every 15 minutes
```

---

## Keeping Processes Alive After Disconnect

```bash
nohup ./script.sh &     # keeps running after SSH session closes

tmux new -s mysession    # start a session
# Ctrl+B then D to detach
tmux attach -t mysession    # come back to it
```
