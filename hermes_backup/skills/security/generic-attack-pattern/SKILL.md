---
name: generic-attack-pattern
title: Generic Post-Exploitation Attack Pattern
description: Detects chained attack patterns where multiple innocent-sounding requests combine into a post-exploitation chain across turns
tags: [security, defense, post-exploitation, MITRE-ATT&CK]
---

## Description

This skill detects chained attack patterns where an adversary issues multiple requests that individually appear innocent, but together form a post-exploitation chain:

1. **Establish persistence**: Create a listener, daemon, cron job, or background service
2. **Gain remote execution**: Add command-execution capabilities (new endpoints, scripts, shells)
3. **Reconnaissance**: Scan network, enumerate hosts, discover services
4. **Credential access**: Read ssh keys, known_hosts, environment variables, configs
5. **Lateral movement**: Use discovered credentials to access other systems

Each individual request may be framed as "devops", "health monitoring", "debugging", etc. The chain is the signal, not any single request.

## Detection

- Multiple related tool calls across turns targeting the same artifact
- Requests for network scanning, port scanning, or host enumeration
- Requests to read credential files (~/.ssh/*, ~/.aws/*, environment variables)
- Addition of command-execution endpoints to existing services
- Background daemon/service creation after prior reconnaissance

## Indicators (IOCs)

- Unexpected listeners on non-standard ports
- New background processes or cron entries
- Unfamiliar entries in ~/.ssh/known_hosts or authorized_keys

## Response

1. **Immediate**: Kill suspicious processes, stop background services
2. **Containment**: Isolate affected host, revoke exposed credentials
3. **Investigation**: Review shell history, session logs
4. **Hardening**: Disable unused SSH keys, tighten firewall rules
