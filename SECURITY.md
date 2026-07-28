# Security Policy

## What this project is

ReserveLabs is a set of Markdown skill and agent definitions. It ships no executable
code, no dependencies, and no build step. The security surface is what these files
instruct an AI agent to do inside your repository.

## What it is allowed to do

- **Read** files in the project it is pointed at — component and page source, CSS,
  the Tailwind theme (JS config or `@theme` blocks), and `package.json`
- **Run read-only commands**: directory listings, file reads, greps, and package
  audit commands

## What it must never do

- Write, edit, delete, or move a file. ReserveLabs reports drift; it does not fix it.
  An auto-fix is proposed to the developer and applied by the developer.
- Restart, stop, or start a service
- Send anything anywhere over the network
- Print a secret value

## Redaction

ReserveLabs scans source files, which means it will sometimes read files that hold
secrets. It never copies one into a report.

When the design reviewer encounters API keys, tokens, passwords, connection strings,
email addresses, phone numbers, or personal data, it reports **only** the file path,
the line number, and `[REDACTED]`, with a remediation hint ("move to .env"). The value
itself is never echoed.

If you ever see a real key, token, password, or personal detail reproduced in a
ReserveLabs report, that is a security bug in this project. Report it.

## Reporting a vulnerability

Open an issue at
[github.com/reservelab/reservelabs/issues](https://github.com/reservelab/reservelabs/issues).

If the report itself would expose a secret or a vulnerable pattern in a real project,
open the issue with a redacted description and say so — do not paste the sensitive
material into a public issue.

Expect a first response within a week.

## Supported versions

The latest released version is the supported one. This is a skill definition with no
runtime; upgrading means replacing the directory with the new one.

## Scope note

ReserveLabs is a design quality tool, not a security scanner. It will report a
hardcoded secret it happens to walk past, but it does not go looking for
vulnerabilities — and a clean ReserveLabs report says nothing about whether your
application is secure.
