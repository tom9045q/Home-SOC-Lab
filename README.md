# Home SOC Lab

A home lab built around Wazuh SIEM, used to practice detection, alert triage, and 
writing up investigations like a real SOC analyst would.

## Overview

This runs on my Ubuntu Server (hostname d01) with Wazuh installed as the SIEM. The 
goal is to get hands-on experience with the full cycle of SOC work: deploying tools, 
triaging alerts, figuring out what's real vs a false positive, and documenting it 
properly.

## Environment

Host: Ubuntu Server 24.04, hostname d01
SIEM: Wazuh (Manager, Indexer, Dashboard)
Planned: Kali Linux VM as an attacker/agent machine for simulated attacks

## What's in here

- 01-lab-architecture - setup notes, VM specs, network layout
- 02-detections - writeups of alerts I've investigated
- 03-attack-simulation - planned, once Kali VM is up
- evidence - screenshots and logs backing up each investigation

## Status

Active - currently expanding log sources and tuning detection rules.
