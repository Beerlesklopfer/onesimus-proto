# Security policy

This repository holds the protocol definitions (`.proto`) of the Onesimus
backup stack, shared by the Director `odir`, the file daemon `ofd`, the
storage daemon `osd`, the console `oconsole` and the graphical client. It
contains no running code of its own, but a flaw in a definition (a missing
authorisation field, a message that cannot be bounded, an ambiguous
signature format of the plugin contract) reaches every implementation. This
policy also serves the vulnerability handling required by the EU Cyber
Resilience Act (Regulation (EU) 2024/2847).

## Supported versions

| Version | Security fixes |
|---|---|
| `development` branch (before v1.0.0) | yes, newest state only |
| 0.x | no, update to the newest state |
| from v1.0.0 | the support period is stated with each release |

## Reporting a vulnerability

Please report a vulnerability **privately**, never in a public issue,
discussion or pull request:

- on GitHub: **Security → Report a vulnerability** in this repository
  (private vulnerability reporting), or
- by e-mail to **contact@onesimus.io** with the subject `SECURITY`.

Please include the affected file and commit, the message or call concerned,
the steps to reproduce and what an attacker gains.

## What happens next

1. We confirm receipt within **7 days**.
2. We assess the report and tell you whether we consider it a vulnerability
   and how severe it is.
3. We fix it in the supported versions and agree a publication date with you.
   We aim to publish a fix within 90 days.
4. We publish a GitHub Security Advisory (with a CVE where it applies) and
   credit you, unless you prefer not to be named.

Actively exploited vulnerabilities and severe incidents are reported to the
authorities as Article 14 of the Cyber Resilience Act requires.

## Scope

In scope: the definitions in this repository and what they permit.
Vulnerabilities in an implementation (odir, ofd, osd, oconsole) belong in the
`onesimus-director` repository, in the graphical client in its own
repository; report them there, or here if you are unsure, and we pass them on.
Out of scope: Bareos itself and its protocols (report to the Bareos project),
third-party libraries (report upstream), and findings that need an already
compromised machine.
