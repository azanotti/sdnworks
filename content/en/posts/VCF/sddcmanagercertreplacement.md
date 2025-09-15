+++
date = '2025-09-15T11:17:00+02:00'
draft = false
title = 'SDDC Manager Certificate Replacement Fails Without Error'
categories = ['VCF']
tags = ['operations', 'certificates', 'SDDC Manager']
+++

## Introduction

While performing a full certificate replacement for a customer, I encountered an issue where the certificate replacement task for **SDDC Manager** failed without providing any meaningful error details.

The task failed with the following generic message:
```
Message: Failed to replace certificate for sddc-manager.xxx.yyy due to:
Remediation Message:
Reference Token:
Cause:
```

## Troubleshooting

Initially, I suspected that the issue was with the certificate itself — possibly missing SANs or some change introduced by the CA. 
However, after comparing it with the existing certificate, I confirmed they were functionally identical.

I began reviewing the logs in SDDC Manager. 
The usual starting point, `operationsmanager.log`, only showed the task as failed, without any additional context or useful information.

Digging deeper, I examined the `vcf-commonsvcs` log and found a critical clue:
```
[2025-09-08T11:50:10] INFO  [common,456#######################8972d6,aa7e] [c.v.e.s.a.u.utils.DnsResolutionUtils,http-nio-127.0.0.1-7100-exec-6] Dns name sddc-manager.xxx.yyy, resolved to IPs [127.0.0.1]
[2025-09-08T11:50:10] ERROR [common,684#######################8972d6,aa7e] [c.v.e.s.a.u.utils.SslCertValidator,http-nio-127.0.0.1-7100-exec-6] Certificate validations failed
java.security.cert.CertificateException: Hostname in CN field sddc-manager.xxx.yyy could not be resolved to an IP address of the SDDC manager aaa.bbb.ccc.ddd
```

## Root Cause

It turned out that **`vcf-commonsvcs`** was resolving the SDDC Manager FQDN to `127.0.0.1` based on entries in the `/etc/hosts` file. 

Since the certificate validation logic compares the FQDN resolution to the actual IP of the SDDC Manager, the mismatch caused the replacement task to silently fail.

## Solution

The `/etc/hosts` entry for `127.0.0.1` is required for internal operations, so it cannot be permanently removed. However, it **can be temporarily modified** to allow the certificate replacement task to succeed.

Follow these steps:

1. **Take a snapshot** of the SDDC Manager VM before making any changes.
2. SSH into the SDDC Manager using the `vcf` user, then escalate to root using `su -`.
3. Backup the current /etc/hosts file: `cp /etc/hosts /etc/hosts.bak`.
4. Edit the /etc/hosts file and comment out the lines beginning with 127.0.0.1 and ::1.
5. Verify that DNS resolution now returns the correct IP: `nslookup sddc-manager.xxx.yyy` must not return 127.0.0.1.
6. Retry the certificate replacement task from the SDDC Manager UI.
7. Once the task completes successfully, restore the original /etc/hosts: `mv /etc/hosts.bak /etc/hosts`.

This workaround resolves the certificate replacement failure caused by improper hostname resolution during validation.