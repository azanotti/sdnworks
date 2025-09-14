+++
date = '2025-09-14T21:56:32+02:00'
draft = true
title = 'Reset versione target upgrade VCF'
categories = ['VCF']
tags = ['operazioni', 'aggiornamento', 'SDDC Manager']
+++

Durante un aggiornamento da VMware Cloud Foundation (VCF) 4.4 a VCF 5.1, ho riscontrato un problema di compatibilità imprevisto che ha bloccato il processo di upgrade del dominio.

Durante il tentativo di aggiornare un domain, SDDC Manager ha segnalato il seguente errore:
"Not interoperable: ESX_HOST 8.0.2-22380479 -> SDDC_MANAGER 5.0.0.1-22485660"

Questo è stato sorprendente, poiché la build ESXi 8.0.2-22380479 è chiaramente elencata nella Bill of Materials (BOM) per VCF 5.1. 
Tuttavia, il processo di aggiornamento all'interno di SDDC Manager non poteva proseguire a causa di questo controllo di incompatibilità.

![BOM di VCF 5.1 disponibile qui: https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-5-2-and-earlier/5-1/vcf-release-notes/vmware-cloud-foundation-51-release-notes.html](/images/vcf_51_bom.png)

Sono stati eseguiti i seguenti passaggi utilizzando l’interfaccia Developer Center di SDDC Manager:

1. Accedere al Developer Center di SDDC Manager
2. Selezionare la categoria API **Domains**
3. Eseguire la chiamata API **GET /v1/domains** e annotare l’ID del domain interessato
4. Passare alla categoria API **TargetUpgradeVersion**
5. Eseguire la chiamata API **PATCH /v1/releases/domains/{domainId}** utilizzando l’ID del domain ottenuto al passo 3
6. Riprovare l’aggiornamento selezionando la versione 5.1 come versione di destinazione

Questo workaround ha permesso di forzare l’SDDC Manager a riconoscere la compatibilità e ha sbloccato il processo di aggiornamento del dominio a VCF 5.1.