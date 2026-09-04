# homelabs

Write-ups from penetration testing lab work — CEH-style engagements, CTF-style boxes, and self-directed exploitation exercises. Built as a running portfolio while working through PwnTillDawn, HackTheBox, VulnHub, and similar platforms.

## Write-ups

| Target | Summary | Severity |
|---|---|---|
| [Jenkins CI/CD](./jenkins/) | Anonymous FTP → leaked creds → Jenkins Script Console RCE → SYSTEM | Critical |
| [Maian Cart 3.8](./maiancart/) | Unauthenticated elFinder RCE (CVE-2021-32172) → token manipulation → SYSTEM | Critical |
| [DeathNote (VulnHub)](./deathnote-writeup/) | WPScan credential brute-force → malicious WordPress plugin upload RCE → wp-config credential reuse → encoded file analysis (Brainfuck, Hex+Base64) → unrestricted sudo to root | Critical |
| [Voting System](./voting-system/) | Unauthenticated SQL injection (login form) → full admin credential dump; exposed web shell in /Images/ → unauthenticated RCE as local Administrator | Critical |

---

More write-ups added as engagements are completed.
