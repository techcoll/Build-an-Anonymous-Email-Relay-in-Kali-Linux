![image.png](https://github.com/techcoll/Build-an-Anonymous-Email-Relay-in-Kali-Linux/blob/main/image.png)
**youtube.com/@cybertech-arena**

# **Build-an-Anonymous-Email-Relay-in-Kali-Linux**
Host your own SMTP server in Kali to send anonymous outbound emails without revealing your real IP, location, or system identity.

**🧰 1. Requirements**
| Item                 | Details                                             |
| -------------------- | --------------------------------------------------- |
| 🐧 OS                | Kali Linux 2024+ (root or sudo access)              |
| 🌐 Internet Access   | Direct or through VPN/Tor                           |
| 📨 Domain (Optional) | Not required, but improves deliverability           |
| 🔐 Tools             | `postfix`, `mailutils`, `gnupg2`, `tor`, `torsocks` |

<br/>

**⚙️ 2. Update Kali & Install Required Packages**<br>
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install postfix mailutils gnupg2 tor torsocks rsyslog -y
````


**🛠️ 3. Configure Postfix for Anonymous Relay**<br>
 **When prompted during installation:**
* Choose Internet Site
* Set system mail name to a any domain of your choice <br>
Edit the main config:
```
sudo nano /etc/postfix/main.cf
```
Replace its content with:
```
# === Basic Identity ===
myhostname = your_choosen_domain
myorigin = $myhostname
mydestination = 
inet_interfaces = all
inet_protocols = ipv4

# === Privacy & Security ===
smtpd_banner = relaynode.local ESMTP
disable_vrfy_command = yes
smtpd_helo_required = no
append_dot_mydomain = no
biff = no
readme_directory = no

# === Privacy Filters ===
header_checks = regexp:/etc/postfix/header_checks

# === Relay Settings ===
relayhost = 
smtpd_recipient_restrictions = permit_mynetworks, reject_unauth_destination
mynetworks = 127.0.0.0/8
```
Save & exit.

**🧹 4. Strip All Identifying Headers**

Create a header stripping rule file:
```
sudo nano /etc/postfix/header_checks
```
**Add:**
```
/^Received:/                        IGNORE
/^X-Originating-IP:/                IGNORE
/^User-Agent:/                      IGNORE
/^X-Mailer:/                        IGNORE
/^Message-ID:/                      IGNORE
/^Return-Path:/                     IGNORE
/^X-Original-From:/                 IGNORE
/^Delivered-To:/                    IGNORE
/^X-Authenticated:/                 IGNORE
```
**Apply it:**
```
sudo postmap /etc/postfix/header_checks
```
**🧅 5. Route Email Traffic Over Tor **

1. Enable Tor:
```
sudo systemctl enable --now tor
```
2. Edit /etc/tor/torrc:
```
sudo nano /etc/tor/torrc
```
Uncomment or add:
```
SocksPort 9050
```

**🪓 6. Disable Mail Logging**

⚠️ This step is optional. It makes auditing impossible but hides traces.
```
sudo systemctl stop postfix rsyslog
sudo ln -sf /dev/null /var/log/mail.log
sudo ln -sf /dev/null /var/log/mail.err
sudo ln -sf /dev/null /var/log/mail.info
sudo systemctl start rsyslog postfix
```
**✉️ 7. Test the Anonymous Relay**
1. Create a test email:
```
nano /tmp/mail.txt
```
```
From: Anonymous <ghost@relaynode.local>
To: youraddress@example.com
Subject: Anonymous Test

This is a test from my Kali-based anonymous relay.
```
2. Send it directly:
```
sendmail -t < /tmp/mail.txt
```
✅ If it arrives in the recipient’s inbox:

* Check email headers.
* Original IP and host should be stripped.
* (Optional) Send via Tor:
```
torsocks sendmail -t < /tmp/mail.txt
```
✅ Final Checklist — What You Now Have

✔️ A fully functional Postfix SMTP relay on Kali<br>
✔️ All identifying headers stripped<br>
✔️ Optional Tor-based routing for outbound traffic<br>
✔️ Optional GPG encryption for email payload<br>
✔️ Minimal or no logs on your system<br>
✔️ Capable of sending mail directly to the internet without exposing your real IP<br>



```
⚠️ Legal & Ethical Notice

This setup is powerful — but also dangerous if misused.
Use it only for:

Ethical cybersecurity research

Anonymous communication in oppressive regimes

Whistleblowing or investigative journalism

Privacy-focused experimentation

Any use for spam, phishing, harassment, or criminal activity is illegal and can be prosecuted.
