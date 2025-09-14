+++
date = '2025-09-14T21:16:27+02:00'
draft = false
title = 'Reset VCF domain upgrade target version'
categories = ['VCF']
tags = ['operations', 'upgrade', 'SDDC Manager']
+++

During an upgrade from VMware Cloud Foundation (VCF) 4.4 to VCF 5.1, I encountered an unexpected compatibility issue that blocked the domain upgrade process.

While attempting to upgrade a domain, the SDDC Manager flagged the following error: "Not interoperable: ESX_HOST 8.0.2-22380479 -> SDDC_MANAGER 5.0.0.1-22485660"

This was surprising because the ESXi build 8.0.2-22380479 is clearly listed in the Bill of Materials (BOM) for VCF 5.1. However, the upgrade workflow within SDDC Manager wouldn’t proceed due to this incompatibility check.

![VCF 5.1 BOM available at: https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-5-2-and-earlier/5-1/vcf-release-notes/vmware-cloud-foundation-51-release-notes.html](/images/vcf_51_bom.png)

To resolve this issue the following steps were performed:
1. Access SDDC Manager's development center
2. Select API category **Domains**
3. Run the **GET /v1/domains** API and note down the affected domain ID
4. Move to API category **TargetUpgradeVersion**
5. Run the **PATCH /v1/releases/domains/{domainId}** API using the domain ID obtained in step 3
6. Retry the upgrade by selecting 5.1 as target version