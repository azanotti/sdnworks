+++
date = '2025-09-15T11:17:07+02:00'
draft = false
title = 'Sostituzione del certificato SDDC Manager fallisce senza errore'
+++
## Introduzione

Durante la sostituzione completa dei certificati per un cliente, ho riscontrato un problema in cui l'operazione di sostituzione del certificato per **SDDC Manager** falliva senza fornire dettagli di errore significativi.

Il task falliva con il seguente messaggio generico:
```
Message: Failed to replace certificate for sddc-manager.xxx.yyy due to:
Remediation Message:
Reference Token:
Cause:
```

## Analisi del Problema

In un primo momento ho pensato che il problema fosse legato al certificato stesso — forse mancava il SAN o la CA aveva introdotto qualche modifica.  
Tuttavia, dopo aver confrontato il nuovo certificato con quello esistente, ho confermato che erano funzionalmente identici.

Ho quindi iniziato a esaminare i log di SDDC Manager.  
Come sempre, ho iniziato con il file `operationsmanager.log`, ma mostrava solo che il task era fallito, senza ulteriori informazioni utili.

Approfondendo, ho esaminato il log `vcf-commonsvcs` e ho trovato un indizio cruciale:
```
[2025-09-08T11:50:10] INFO  [common,456#######################8972d6,aa7e] [c.v.e.s.a.u.utils.DnsResolutionUtils,http-nio-127.0.0.1-7100-exec-6] Dns name sddc-manager.xxx.yyy, resolved to IPs [127.0.0.1]
[2025-09-08T11:50:10] ERROR [common,684#######################8972d6,aa7e] [c.v.e.s.a.u.utils.SslCertValidator,http-nio-127.0.0.1-7100-exec-6] Certificate validations failed
java.security.cert.CertificateException: Hostname in CN field sddc-manager.xxx.yyy could not be resolved to an IP address of the SDDC manager aaa.bbb.ccc.ddd
```

## Causa

È risultato che **`vcf-commonsvcs`** stava risolvendo il FQDN di SDDC Manager in `127.0.0.1` a causa delle voci presenti nel file `/etc/hosts`.

Poiché la logica di validazione del certificato confronta la risoluzione del FQDN con l'indirizzo IP effettivo di SDDC Manager (SAN del certificato), questo mismatch causava il fallimento del task senza errori espliciti.

## Soluzione

La voce `/etc/hosts` per `127.0.0.1` è necessaria per il funzionamento interno del sistema, quindi **non può essere rimossa permanentemente**. 
Tuttavia, può essere **modificata temporaneamente** per permettere il completamento della sostituzione del certificato.

Eseguire i seguenti passaggi:

1. **Creare uno snapshot** della VM SDDC Manager prima di effettuare modifiche.
2. Connettersi via SSH a SDDC Manager usando l’utente `vcf`, quindi ottenere i privilegi di root con `su -`.
3. Eseguire un backup del file `/etc/hosts`: `cp /etc/hosts /etc/hosts.bak`
4. Modificare il file /etc/hosts e commentare le righe che iniziano con 127.0.0.1 e ::1.
5. Verificare che la risoluzione DNS restituisca l’indirizzo IP corretto: `nslookup sddc-manager.xxx.yyy` non deve ritornare 127.0.0.1.
6. Riprova a eseguire il task di sostituzione del certificato dall’interfaccia UI di SDDC Manager.
7. Una volta completato con successo il task, ripristinare il file originale: `mv /etc/hosts.bak /etc/hosts`.

Questa procedura risolve il fallimento della sostituzione del certificato causato dalla risoluzione errata del nome host durante la validazione.