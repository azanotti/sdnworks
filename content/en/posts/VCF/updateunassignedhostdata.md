+++
date = '2025-09-15T14:41:26+02:00'
draft = false
title = 'Update unassigned host data'
categories = ['VCF']
tags = ['operations', 'commissioning', 'SDDC Manager']
+++

A few months ago, I faced an interesting challenge involving VMware Cloud Foundation (VCF) and SDDC Manager. A customer had commissioned several ESXi hosts into SDDC Manager, only to later realize that their ESXi versions were lower than the version of the domain they were intended to join.

Normally, this mismatch halts progress and requires a tedious process:
- Remove the host from SDDC Manager
- Reinstall ESXi with the correct version
- Recommission the host

This approach, while safe, is time-consuming — and of course, everything needed to be done fast.

## The Goal: Avoid Reinstallation, Save Time
Knowing that hosts can be upgraded outside of SDDC Manager using esxcli, I decided to explore a faster path. 
The issue? Even after upgrading the host, SDDC Manager doesn’t automatically recognize the new version — it continues to reference the version stored in its internal database.

So, here was my plan:
- Upgrade the ESXi host manually using esxcli.
- Update the version in the SDDC Manager database to match.
- Proceed with cluster expansion using the now-compatible host.

And yes — it worked perfectly. Here's a breakdown of the process I followed.

## Step-by-Step: Update ESXi Host Version in SDDC Manager DB

⚠️ DISCLAIMER: This approach is unsupported and potentially risky. Use it at your own discretion, and always create backups. 
Any modifications to the SDDC Manager database can void support agreements.


1. Snapshot the SDDC Manager VM (without memory!)
2. Connect to SDDC Manager and escalate to root user using `su -`
3. Access the database using `psql --host=localhost -U postgres`
4. Switch the platform database using `\c platform`
5. Find the IDs of the affected hosts: `SELECT * from host;`
6. Update each host individually: `UPDATE host SET version = 'x.y.z-build' WHERE id = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';`
7. Proceed with the cluster expansion

## Final thoughts
This workaround saved significant time and helped avoid unnecessary reinstallation across multiple hosts. 
However, it’s not without risks — you’re essentially telling SDDC Manager to trust the version you manually inserted.

If you go this route:
- Understand what you’re doing.
- Always have a rollback plan (hint: snapshot!).