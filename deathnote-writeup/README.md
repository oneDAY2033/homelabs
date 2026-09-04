---
tags: [pentest, vulnhub, ctf, writeup, wordpress]
---

> [!info] Companion PDF
> This note is a text version of the DeathNote VulnHub writeup. The full report with all 21 embedded screenshots is saved as **DeathNote_VulnHub_Writeup.pdf** — keep that alongside this note for the visual evidence.

**VulnHub CTF Writeup**

**DeathNote**

Full Compromise Walkthrough

Prepared by: Kelvin Bigson

Organization: Inveteck Global

Date: September 3, 2026

1\. Engagement Overview

<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 72%" />
</colgroup>
<tbody>
<tr class="odd">
<td><blockquote>
<p><strong>Target</strong></p>
</blockquote></td>
<td><blockquote>
<p>deathnote.vuln (VulnHub)</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>IP Address</strong></p>
</blockquote></td>
<td><blockquote>
<p>192.168.100.32</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Platform</strong></p>
</blockquote></td>
<td><blockquote>
<p>VulnHub</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Difficulty</strong></p>
</blockquote></td>
<td><blockquote>
<p>Easy</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>OS</strong></p>
</blockquote></td>
<td><blockquote>
<p>Debian GNU/Linux (Kernel 4.19.0-17-amd64)</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Web Stack</strong></p>
</blockquote></td>
<td><blockquote>
<p>Apache 2.4.38 + WordPress 5.x + MariaDB 10.3</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Objective</strong></p>
</blockquote></td>
<td><blockquote>
<p>Gain root access via full exploit chain</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Tester</strong></p>
</blockquote></td>
<td><blockquote>
<p>Kelvin Bigson — Inveteck Global</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Date</strong></p>
</blockquote></td>
<td><blockquote>
<p>September 3, 2026</p>
</blockquote></td>
</tr>
</tbody>
</table>

2\. Attack Chain Summary

<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 72%" />
</colgroup>
<tbody>
<tr class="odd">
<td><blockquote>
<p><strong>Phase 1</strong></p>
</blockquote></td>
<td><blockquote>
<p>Reconnaissance — Nmap, /etc/hosts, web enumeration, Feroxbuster</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Phase 2</strong></p>
</blockquote></td>
<td><blockquote>
<p>Credential discovery — robots.txt, important.jpg, directory listing,
WPScan brute-force</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Phase 3</strong></p>
</blockquote></td>
<td><blockquote>
<p>Remote Code Execution — malicious WordPress plugin upload</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Phase 4</strong></p>
</blockquote></td>
<td><blockquote>
<p>Lateral movement — wp-config.php credentials → SSH as user l</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Phase 5</strong></p>
</blockquote></td>
<td><blockquote>
<p>Encoded file analysis — Brainfuck, Hex+Base64 → SSH as kira</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Phase 6</strong></p>
</blockquote></td>
<td><blockquote>
<p>Privilege escalation — sudo (ALL:ALL) → root</p>
</blockquote></td>
</tr>
</tbody>
</table>

3\. Phase 1 — Reconnaissance

3.1 Port Scanning

An Nmap scan was run against the target to identify open ports and
running services:

> nmap -sC -sV -A -T4 192.168.100.32

Two ports were identified: SSH on 22 and Apache HTTP on 80.

📸 *(screenshot in PDF report)*

*Figure 1 — Nmap scan confirming SSH (22) and Apache HTTP (80) on
192.168.100.32*

3.2 /etc/hosts Configuration

Browsing directly to deathnote.vuln/wordpress before adding a hosts
entry returned "Server Not Found":

📸 *(screenshot in PDF report)*

*Figure 2 — Browser unable to resolve deathnote.vuln before /etc/hosts
update*

The target hostname was added to /etc/hosts to enable proper name
resolution:

> echo "192.168.100.32 deathnote.vuln" \>\> /etc/hosts

📸 *(screenshot in PDF report)*

*Figure 3 — /etc/hosts updated with 192.168.100.32 deathnote.vuln entry*

3.3 Web Enumeration

After DNS resolution was fixed, the WordPress site was browsable. The
site used a Death Note theme with a navigation menu including a "HINT"
link:

📸 *(screenshot in PDF report)*

*Figure 4 — WordPress site showing Death Note theme, KIRA navigation,
and HINT menu item*

Feroxbuster was run for directory brute-forcing:

> feroxbuster -u http://deathnote.vuln -w
> /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
> --recursion-depth 4

Notable findings included /wordpress/wp-content/uploads/ with directory
listing enabled and xmlrpc.php present:

📸 *(screenshot in PDF report)*

*Figure 5 — Feroxbuster output confirming WordPress paths and directory
listing on uploads/*

4\. Phase 2 — Credential Discovery

4.1 robots.txt and important.jpg

The robots.txt file pointed to /important.jpg. Downloading it and
running the file command revealed it was actually ASCII text, not an
image:

> curl http://deathnote.vuln/robots.txt
>
> wget http://deathnote.vuln/important.jpg && file important.jpg && cat
> important.jpg

The file contained a message from "Soichiro Yagami" stating the login
username was in user.txt and the password was in the site's hint
section.

📸 *(screenshot in PDF report)*

*Figure 6 — robots.txt pointing to /important.jpg; file command
revealing it is ASCII text; content showing username clue*

4.2 WordPress Uploads Directory Listing

The exposed uploads directory at /wordpress/wp-content/uploads/2021/07/
revealed two files — notes.txt (password wordlist) and user.txt
(username list) — alongside site images:

📸 *(screenshot in PDF report)*

*Figure 7 — Directory listing of uploads/2021/07/ exposing notes.txt and
user.txt*

4.3 Downloading the Wordlists

Both files were downloaded directly from the exposed uploads directory:

> wget
> http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt
>
> wget
> http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt

user.txt contained Death Note character names; notes.txt contained a
death4\* password pattern list. The widget on the site had also revealed
the string "iamjustic3" which was added to notes.txt.

📸 *(screenshot in PDF report)*

*Figure 8 — Downloading notes.txt and user.txt from the exposed uploads
directory*

4.4 WordPress Brute-Force with WPScan

Hydra produced false positives due to WordPress's cookie requirement.
WPScan was used instead, correctly handling the authentication flow via
xmlrpc.php:

> wpscan --url http://deathnote.vuln/wordpress/ --usernames users.txt
> --passwords notes.txt

Valid credentials found: KIRA / iamjustic3 and kira / iamjustic3.

📸 *(screenshot in PDF report)*

*Figure 9 — WPScan brute-force via xmlrpc confirming kira:iamjustic3 as
valid credentials*

4.5 WordPress Admin Access

Login to wp-admin with kira:iamjustic3 was successful, granting full
administrative access to the WordPress installation:

📸 *(screenshot in PDF report)*

*Figure 10 — WordPress admin dashboard accessed as user kira*

5\. Phase 3 — Remote Code Execution via Plugin Upload

5.1 Payload Preparation and Packaging

The Kali-built-in pentestmonkey PHP reverse shell was copied, edited
with the attacker IP and port, a WordPress plugin header was prepended,
and it was packaged into a zip file:

> cp /usr/share/webshells/php/php-reverse-shell.php
> shellplugin/shell.php
>
> mkdir shellplugin && cp php-reverse-shell.php shellplugin/shell.php &&
> zip -r shellplugin.zip shellplugin/

📸 *(screenshot in PDF report)*

*Figure 11 — Plugin folder created, shell.php copied in, and
shellplugin.zip packaged*

5.2 Upload, Listener, and Shell

A Netcat listener was started before upload. The plugin was uploaded via
Plugins → Add New → Upload Plugin → shellplugin.zip:

> nc -lvnp 1111

WordPress confirmed successful installation. Clicking Activate Plugin
triggered the reverse shell callback:

📸 *(screenshot in PDF report)*

*Figure 12 — Plugin installed successfully; nc listener waiting on port
1111*

📸 *(screenshot in PDF report)*

*Figure 13 — Reverse shell received as www-data immediately upon plugin
activation*

6\. Phase 4 — Post-Exploitation and Lateral Movement

6.1 Initial Enumeration as www-data

After shell stabilisation with Python pty, the filesystem was
enumerated. /home showed two users: kira and l. The file
/home/l/user.txt contained Brainfuck-encoded content:

📸 *(screenshot in PDF report)*

*Figure 14 — Shell as www-data; filesystem enumeration showing home
directories; Brainfuck content in user.txt*

6.2 Brainfuck Decoding

The Brainfuck content from user.txt was decoded using copy.sh/brainfuck:

📸 *(screenshot in PDF report)*

*Figure 15 — Brainfuck decoded to: "i think u got the shell, but you
wont be able to kill me -kira"*

This was a narrative taunt, not a password. Investigation continued into
/opt/L/.

6.3 /opt/L/ Investigation

The /opt/L/ directory contained two subdirectories: kira-case and
fake-notebook-rule. The case-file.txt pointed to the fake-notebook-rule
folder:

📸 *(screenshot in PDF report)*

*Figure 16 — /opt/L/kira-case/case-file.txt referencing
fake-notebook-rule folder*

6.4 case.wav — Hex + Base64 Encoding

Inside fake-notebook-rule, a file named case.wav contained raw hex data.
A hint file stated "use cyberchef":

> cat case.wav
>
> 63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a
> 70 62 43 41 3d

📸 *(screenshot in PDF report)*

*Figure 17 — case.wav hex content and hint file directing use of
CyberChef*

6.5 CyberChef Decoding — kiraisevil

CyberChef was used with a two-step recipe: From Hex → From Base64.
Output: passwd : kiraisevil

📸 *(screenshot in PDF report)*

*Figure 18 — CyberChef From Hex + From Base64 decoding case.wav to
reveal passwd : kiraisevil*

6.6 SSH Credential Verification for User l

Hydra was used to verify the DB credential death4me (from wp-config.php)
as a valid SSH login for user l:

> hydra -l l -P notes.txt -t 4 192.168.100.33 ssh

📸 *(screenshot in PDF report)*

*Figure 19 — Hydra confirming l:death4me as valid SSH credentials*

7\. Phase 5 & 6 — SSH as kira and Privilege Escalation

7.1 SSH as kira + sudo Root

Using the decoded password to SSH as kira, sudo -l immediately revealed
unrestricted access. A root shell was obtained with sudo su:

> ssh kira@deathnote.vuln \# password: kiraisevil
>
> sudo -l
>
> sudo su

Output: User kira may run the following commands on deathnote: (ALL :
ALL) ALL

📸 *(screenshot in PDF report)*

*Figure 20 — SSH as kira; sudo -l confirming (ALL:ALL); root shell
obtained via sudo su*

8\. Vulnerability Findings Summary

<table>
<colgroup>
<col style="width: 35%" />
<col style="width: 20%" />
<col style="width: 44%" />
</colgroup>
<tbody>
<tr class="odd">
<td><blockquote>
<p><strong>Vulnerability</strong></p>
</blockquote></td>
<td><blockquote>
<p><strong>Severity</strong></p>
</blockquote></td>
<td><blockquote>
<p><strong>Impact</strong></p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p>Weak WordPress Credentials</p>
</blockquote></td>
<td><blockquote>
<p><strong>High</strong></p>
</blockquote></td>
<td><blockquote>
<p>Admin panel access via brute-force</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p>Malicious Plugin Upload (RCE)</p>
</blockquote></td>
<td><blockquote>
<p><strong>Critical</strong></p>
</blockquote></td>
<td><blockquote>
<p>Remote code execution as www-data</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p>Credentials in wp-config.php</p>
</blockquote></td>
<td><blockquote>
<p><strong>High</strong></p>
</blockquote></td>
<td><blockquote>
<p>Credential reuse → SSH access as l</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p>Sensitive Data in Encoded Files</p>
</blockquote></td>
<td><blockquote>
<p><strong>Medium</strong></p>
</blockquote></td>
<td><blockquote>
<p>Passwords recoverable via CyberChef</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p>Unrestricted sudo for kira</p>
</blockquote></td>
<td><blockquote>
<p><strong>Critical</strong></p>
</blockquote></td>
<td><blockquote>
<p>Full privilege escalation to root</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p>Directory Listing Enabled</p>
</blockquote></td>
<td><blockquote>
<p><strong>Low</strong></p>
</blockquote></td>
<td><blockquote>
<p>Uploads directory publicly enumerable</p>
</blockquote></td>
</tr>
</tbody>
</table>

9\. Tools Used

<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 72%" />
</colgroup>
<tbody>
<tr class="odd">
<td><blockquote>
<p><strong>Nmap</strong></p>
</blockquote></td>
<td><blockquote>
<p>Port scanning and service/OS enumeration</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Feroxbuster</strong></p>
</blockquote></td>
<td><blockquote>
<p>Web directory brute-forcing</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>WPScan</strong></p>
</blockquote></td>
<td><blockquote>
<p>WordPress enumeration and credential brute-force via xmlrpc</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>Hydra</strong></p>
</blockquote></td>
<td><blockquote>
<p>SSH credential brute-force and verification</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>Netcat</strong></p>
</blockquote></td>
<td><blockquote>
<p>Reverse shell listener</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>CyberChef</strong></p>
</blockquote></td>
<td><blockquote>
<p>Hex and Base64 decoding of encoded files</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>copy.sh/brainfuck</strong></p>
</blockquote></td>
<td><blockquote>
<p>Brainfuck decoding of user.txt</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>LinPEAS</strong></p>
</blockquote></td>
<td><blockquote>
<p>Post-exploitation privilege escalation enumeration</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>MySQL Client</strong></p>
</blockquote></td>
<td><blockquote>
<p>Database credential and user table enumeration</p>
</blockquote></td>
</tr>
</tbody>
</table>

10\. Credentials Discovered

<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 72%" />
</colgroup>
<tbody>
<tr class="odd">
<td><blockquote>
<p><strong>kira (WordPress / SSH)</strong></p>
</blockquote></td>
<td><blockquote>
<p>iamjustic3 — brute-forced via WPScan xmlrpc; reused for SSH</p>
</blockquote></td>
</tr>
<tr class="even">
<td><blockquote>
<p><strong>l (DB / SSH)</strong></p>
</blockquote></td>
<td><blockquote>
<p>death4me — extracted from wp-config.php; confirmed via Hydra</p>
</blockquote></td>
</tr>
<tr class="odd">
<td><blockquote>
<p><strong>kira (SSH)</strong></p>
</blockquote></td>
<td><blockquote>
<p>kiraisevil — decoded from hex+base64 encoded case.wav in /opt/L/</p>
</blockquote></td>
</tr>
</tbody>
</table>

11\. Conclusion

The DeathNote VulnHub machine was fully compromised through a
multi-stage attack chain. The engagement demonstrated several common
real-world vulnerability patterns chained together to achieve full root
access.

Key lessons from this engagement:

- File extension trust — important.jpg and case.wav were not their
  declared types; always verify with the file command

- Directory listing — exposing the uploads folder allowed direct
  retrieval of credential wordlists

- Credential reuse — the same password spanned WordPress, database, and
  SSH

- Encoding is not encryption — Brainfuck and Base64/Hex are trivially
  reversible and must not be used to protect sensitive data

- WordPress plugin upload as RCE — admin access to WordPress is
  equivalent to code execution if plugin uploads are enabled

- Principle of least privilege — kira having (ALL:ALL) sudo rights made
  privilege escalation a single command

Full root access was achieved within a single engagement session.
