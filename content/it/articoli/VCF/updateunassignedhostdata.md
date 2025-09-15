+++
date = '2025-09-15T14:41:35+02:00'
draft = false
title = 'Modifica dati host unassigned'
categories = ['VCF']
tags = ['operazioni', 'commissioning', 'SDDC Manager']
+++

Qualche mese fa, mi sono trovato ad affrontare una sfida interessante con VMware Cloud Foundation (VCF) e SDDC Manager. 
Un cliente aveva commissionato diversi host ESXi in SDDC Manager, salvo poi rendersi conto che le versioni di ESXi erano inferiori rispetto a quelle del dominio a cui dovevano essere assegnati.

Normalmente, questo tipo di incompatibilità blocca il processo e richiede una procedura lunga e noiosa:
- Rimuovere l’host da SDDC Manager
- Reinstallare ESXi con la versione corretta
- Ricommissionare l’host

Questo approccio, pur essendo sicuro, richiede tempo — e come al solito, tutto doveva essere fatto in fretta.

## Obiettivo: Evitare la Reinstallazione, Risparmiare Tempo
Sapendo che è possibile aggiornare gli host al di fuori di SDDC Manager utilizzando esxcli, ho deciso di esplorare una strada più veloce.
Il problema? Anche dopo l’aggiornamento, SDDC Manager non riconosce automaticamente la nuova versione — continua a fare riferimento a quella salvata nel proprio database interno.

Quindi, ecco il mio piano:
- Aggiornare manualmente l’host ESXi usando esxcli
- Aggiornare la versione nel database di SDDC Manager
- Procedere con l’espansione del cluster usando l’host ora compatibile

E sì — ha funzionato perfettamente. Di seguito spiego i passaggi eseguiti.

## Passo-Passo: Aggiornare la Versione ESXi nel DB di SDDC Manager

⚠️ DISCLAIMER: Questo approccio è non supportato e potenzialmente rischioso. Usalo a tuo rischio e pericolo, e assicurati sempre di creare backup.
Qualsiasi modifica al database di SDDC Manager può invalidare gli accordi di supporto.

1. Crea uno snapshot della VM SDDC Manager (senza memoria!).
2. Connettiti a SDDC Manager e passa all’utente root usando `su -`.
3. Accedi al database di SDDC con `psql --host=localhost -U postgres`.
4. Passa al database platform con `\c platform`.
5. Trova gli ID degli host interessati con `SELECT * from host;`.
6. Aggiorna ogni host individualmente: `UPDATE host SET version = 'x.y.z-build' WHERE id = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';`.
7. Procedi con l’espansione del cluster

## Considerazioni finali
Questa soluzione alternativa ha permesso di risparmiare molto tempo e ha evitato reinstallazioni inutili su più host.
Tuttavia, non è priva di rischi — stai praticamente dicendo a SDDC Manager di fidarsi della versione che hai inserito manualmente.

Se decidi di seguire questa strada:
- Assicurati di sapere cosa stai facendo
- Abbi sempre un piano di rollback (spoiler: snapshot!)