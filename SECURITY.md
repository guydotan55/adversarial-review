# Security

## Reporting

Report vulnerabilities privately through GitHub's "Report a vulnerability" (Security tab of this repository). Please do not open a public issue for them.

## Threat model

This repository is a prompt (a Claude Code skill) plus documentation; it ships no executable code. The reviewer reads whatever artifact you point it at, so instructions hidden inside that artifact can try to steer the review. Treat reviewer output as advice, not proof, and do not review untrusted artifacts with tools or permissions you would not give that content directly.
