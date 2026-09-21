<img src="https://my-badges.github.io/my-badges/fix-2.png" alt="I did 2 sequential fixes." title="I did 2 sequential fixes." width="128">
<strong>I did 2 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/3e4c3b1e26084f1d7bf9bb90441f72e4a1327e82">3e4c3b1</a>: fix(mailer): resolve SMTP host to IPv4 ourselves, bypass nodemailer's random v4/v6 pick

nodemailer resolves both A and AAAA records and picks a random address
to connect to. Railway has no outbound IPv6 route, so any AAAA hit on
smtp.hostinger.com (Cloudflare-proxied) caused ENETUNREACH on roughly
half of connection attempts. The earlier family:4 attempt was a no-op —
that option isn't read by nodemailer's own DNS step.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/8436c67ba5ab6db4efb260dbe11de561c2bdf6d5">8436c67</a>: fix(mailer): force IPv4 for SMTP transport

Railway has no outbound IPv6 route; smtp.hostinger.com's AAAA record
was causing ENETUNREACH there. Same class of issue as the Supabase
pooler IPv6 workaround already in DATABASE_URL config.

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>