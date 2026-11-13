# Security policy

TrustDiagram is published by [Blooo](https://blooo.io), a cybersecurity company. We take reports seriously.

## Reporting a vulnerability

Please do not open a public issue. Use GitHub's private reporting instead: **Security → Report a vulnerability** on this repository.

Include what you can: the affected component (Obsidian plugin, Confluence app, CLI, library), the version or commit, a minimal reproduction, and the impact you observed.

We acknowledge reports within 5 working days and keep you informed until a fix is released. We credit reporters who wish to be named.

## Scope

In scope:

- the code in this repository;
- the Obsidian plugin and the Confluence app built from it.

Out of scope: Obsidian, Atlassian Confluence and Forge themselves (report to their vendors), and denial of service against your own instance.

Please test only against your own Confluence site, vault and data.

## What we already do

- Diagram text reaches the page as text only: no HTML from a block is ever interpreted.
- The Confluence app writes as the signed-in user, rewrites only the targeted block, and checks the page version and open drafts before saving.
- Block size, text length and element counts are capped, so a block written by one page editor cannot freeze the page for its readers.
