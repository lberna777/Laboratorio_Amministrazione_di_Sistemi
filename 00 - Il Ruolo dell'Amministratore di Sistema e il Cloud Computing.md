# Il Ruolo dell'Amministratore di Sistema e il Cloud Computing

*Corso di (Laboratorio di) Amministrazione di Sistemi — Prof. Marco Prandini, Università di Bologna*

---

## INDICE

1. [Definizione e Compiti dell'Amministratore di Sistema](#1-definizione-e-compiti-dellamministratore-di-sistema)
2. [Le Problematiche dell'Amministrazione](#2-le-problematiche-dellamministrazione)
3. [Automazione e Strumenti](#3-automazione-e-strumenti)
4. [Il Ruolo, il Personaggio e l'Etica Professionale](#4-il-ruolo-il-personaggio-e-letica-professionale)
5. [Prospettive di Mercato e Formazione](#5-prospettive-di-mercato-e-formazione)
6. [LLM e il Futuro dell'Amministrazione](#6-llm-e-il-futuro-dellamministrazione)
7. [Alta Disponibilità e Data Center](#7-alta-disponibilità-e-data-center)
8. [Disponibilità, Downtime e SLA](#8-disponibilità-downtime-e-sla)
9. [Business Continuity](#9-business-continuity)
10. [Cloud Computing: Fondamenti](#10-cloud-computing-fondamenti)
11. [Livelli di Servizio: IaaS, PaaS, SaaS](#11-livelli-di-servizio-iaas-paas-saas)
12. [Prerequisiti Tecnici del Cloud](#12-prerequisiti-tecnici-del-cloud)
13. [Gli Strumenti DevOps](#13-gli-strumenti-devops)

---

## 1. Definizione e Compiti dell'Amministratore di Sistema

### Definizione

Un amministratore di sistema (*system administrator*, sysadm) è il professionista responsabile del corretto funzionamento di uno o più sistemi informatici all'interno di un'organizzazione. Il ruolo comprende la gestione dell'hardware, del software, della rete e della sicurezza, con l'obiettivo di garantire continuità, affidabilità e prestazioni adeguate ai servizi erogati.

### I Compiti Operativi

I compiti dell'amministratore di sistema si articolano in diverse categorie:

**Gestione degli utenti:** creazione e mantenimento degli account, assegnazione e revisione periodica dei permessi, gestione delle password e delle politiche di accesso.

**Gestione dell'hardware:** installazione, configurazione e sostituzione di componenti fisici (server, dischi, schede di rete), monitoraggio dello stato dei dispositivi, interazione con i fornitori per manutenzione e garanzia.

**Gestione del software:** installazione, aggiornamento e rimozione di sistemi operativi e applicazioni, gestione delle dipendenze e delle versioni, applicazione tempestiva di patch di sicurezza.

**Backup e ripristino:** pianificazione e verifica delle procedure di backup, test periodici di ripristino, definizione e mantenimento dei piani di disaster recovery.

**Documentazione:** redazione e aggiornamento di manuali operativi, registrazione delle configurazioni e delle modifiche apportate ai sistemi (change log), documentazione delle procedure di emergenza.

**Monitoraggio delle risorse:** sorveglianza continua di CPU, memoria, storage e rete; definizione di soglie di allerta; analisi dei trend per la pianificazione della capacità.

**Gestione dei fornitori:** negoziazione e supervisione dei contratti di assistenza, coordinamento con vendor hardware e software, valutazione di nuove soluzioni tecnologiche.

**Definizione e applicazione delle politiche:** redazione di policy di sicurezza, di utilizzo accettabile delle risorse e di gestione degli incidenti.

### Sottodiscipline Specialistiche

La complessità dell'ambito ha generato numerose specializzazioni: amministrazione di rete (*network administration*), sicurezza informatica (*security administration*), amministrazione di database (*DBA*), gestione dei sistemi cloud, e figure ibride come il *DevOps engineer*, che unisce competenze sistemistiche e di sviluppo software.

### Gli Strumenti Fondamentali

Gli strumenti tradizionalmente al centro del lavoro del sistemista Unix/Linux includono: shell scripting per l'automazione; protocolli di accesso remoto (SSH); sistemi di gestione centralizzata degli indirizzi IP (DHCP); protocolli di monitoraggio di rete (SNMP); sistemi di autenticazione centralizzata (LDAP).

---

## 2. Le Problematiche dell'Amministrazione

### Interconnessione

L'amministrazione di un sistema in rete non riguarda mai il sistema isolato: ogni intervento può avere effetti a cascata su tutti i sistemi ad esso connessi. La dipendenza reciproca richiede coordinamento e consapevolezza dell'ecosistema complessivo.

### Eterogeneità

L'ambiente tipico di produzione è caratterizzato da decine di varianti di sistemi operativi, applicativi e protocolli di comunicazione. L'ampio spettro di competenze richiesto, combinato con la necessità di copertura temporale continua (24/7), implica normalmente la presenza di una molteplicità di amministratori che devono coordinarsi efficacemente.

### Dimensioni

Un'interfaccia grafica che può sembrare vantaggiosa su una singola macchina perché intuitiva (point and click) diviene inutilizzabile quando si devono amministrare centinaia o migliaia di macchine. La scalabilità degli strumenti è dunque un requisito imprescindibile.

### Predicibilità

Quasi mai si conoscono a priori tutte le variabili da tenere sotto controllo. La domanda fondamentale è come tracciare ogni possibile attività e successivamente analizzare i dati raccolti in modo significativo.

### Tempistiche

A differenza della filiera progettuale — che si confronta con deadline dell'ordine di settimane o mesi — il sistemista deve frequentemente esercitare le proprie abilità di *problem solver* sotto pressione immediata, con la piena consapevolezza che ogni minuto di interruzione dei servizi può tradursi in perdite economiche significative per l'organizzazione.

---

## 3. Automazione e Strumenti

### Il Principio Fondamentale

La maggior parte dei task dell'amministratore presenta una piccola componente "creativa" e una grande parte ripetitiva. **Automatizzare è la chiave della correttezza e dell'efficienza**: un'operazione automatizzata viene eseguita ogni volta allo stesso modo, eliminando gli errori umani derivanti dalla ripetizione manuale.

Gli strumenti fondamentali per l'automazione sono:

- **Shell scripting** — per sequenze di comandi eseguibili su singole macchine o in parallelo su insiemi di sistemi.
- **Configuration as code** — approccio che codifica la configurazione dei sistemi in file versionati e riproducibili (Ansible, Puppet, Chef, Terraform, ecc.).
- **LLM** — strumenti emergenti, potenzialmente utili per le parti ripetitive (si veda la sezione dedicata).

---

## 4. Il Ruolo, il Personaggio e l'Etica Professionale

### Il Paradosso del Buon Sistemista

Il sistema operato da un bravo sistemista funziona in modo silenzioso e continuo, rendendo il suo lavoro invisibile. Conseguentemente, il sistemista viene notato quasi esclusivamente quando qualcosa va storto. Questo crea un paradosso percettivo nell'organizzazione: il bravo sistemista sembra non necessario, proprio perché fa in modo che i problemi non si verifichino.

### Il Potere Assoluto: Diritti e Doveri

Operare come *root* su un sistema significa disporre di accesso illimitato a tutti i dati, processi e risorse. Questo comporta responsabilità etiche e legali di primaria importanza: rispetto della privacy degli utenti, non abuso delle credenziali di accesso, protezione dell'account privilegiato con misure adeguate (password robuste, autenticazione a più fattori), e presidio fisico dei terminali lasciati incustoditi.

Il principio operativo fondamentale è il **minimo privilegio**: eseguire le operazioni quotidiane con account non privilegiati e ricorrere ai privilegi di *root* solo quando strettamente necessario, attraverso `su` o `sudo`.

Particolare attenzione va riservata alla variabile d'ambiente `PATH`: un sistemista dovrebbe sempre verificare che i comandi privilegiati puntino alle versioni di sistema attese, prevenendo attacchi di tipo *path hijacking*.

### La Sindrome della Personalità da Sysadm

Il profilo psicologico tipicamente associato alla figura del sistemista presenta alcune caratteristiche ricorrenti: tendenza al perfezionismo con delirio di onnipotenza, attacchi di panico da verifica ossessivo-compulsiva (il sistema sta funzionando?), sensazione perenne di disastro incombente, e una forma di depressione da impotenza pianificatoria (la consapevolezza che l'imprevedibile accadrà).

### Il Sysadm Forzato

Nelle realtà a prevalenza di piccole e medie imprese (PMI), è fenomeno comune il *sysadm forzato*: il "chiunque" assegnato ad amministrare i sistemi informatici come compito accessorio, indipendentemente dalla sua formazione o predisposizione. Si tratta di una condizione professionale seria che richiede un approccio consapevole e costruttivo, evitando l'approccio passivo-aggressivo.

---

## 5. Prospettive di Mercato e Formazione

### Il Mercato del Lavoro

I dati di mercato indicano una domanda costantemente elevata di professionisti con competenze Linux e sistemistiche. Il cloud computing e il DevOps sono tra le aree a maggiore crescita: il 65% delle aziende ricerca profili DevOps, il 70% cerca competenze cloud, e la quasi totalità del workload cloud gira su sistemi Linux. Le competenze open source — con particolare peso su cloud technologies (50%), containers (19%), networking (9%) e security (16%) — sono tra le più richieste dai *hiring manager*.

### Percorsi di Formazione

L'amministrazione di sistema è storicamente una disciplina poco formalizzata a livello accademico, trasmessa tradizionalmente per apprendistato. Le principali comunità di riferimento sono state USENIX, SAGE e LOPSA, che hanno favorito la condivisione di conoscenze all'interno della comunità Unix.

I principali canali formativi disponibili oggi sono:

- **Man pages** — la documentazione primaria di ogni sistema Unix/Linux, sempre disponibile offline.
- **Guide ufficiali dei produttori** di distribuzioni Linux.
- **Libri** — O'Reilly's Safari Books Online rimane una delle risorse più complete.
- **RFC** — per i protocolli di rete standard: https://www.rfc-editor.org/
- **Risorse online** — Linux Foundation, SANS, MOOCs, corsi dei produttori (IBM, Canonical, ecc.).
- **Certificazioni professionali** — sempre più valorizzate dai selezionatori.

### News e Comunità

Tra le risorse di riferimento: `lwn.net` (articoli di alta qualità su Linux e OSS), `darkreading.com` (sicurezza), `schneier.com` (privacy e sicurezza), `everythingsysadmin.com` (blog di Thomas Limoncelli). Le comunità Reddit (r/sysadmin, r/linux, r/linuxadmin, r/netsec) e Twitter/X presentano comunità ricchissime ma con un rapporto segnale/rumore impegnativo.

### Conferenze di Settore

Le principali conferenze internazionali includono: LISA (Large Installation System Administration, Q4), DefCon (Las Vegas, luglio — la più antica e grande convenzione hacker), VMWorld (virtualizzazione e cloud), re:Invent (AWS, Q4), DevOpsDays (globale). In Italia: LinuxDay (ottobre, in tutta Italia — https://www.linuxday.it/).

---

## 6. LLM e il Futuro dell'Amministrazione

### Valutazione Critica

I Large Language Models (LLM) rappresentano uno strumento emergente di notevole interesse per il settore. La valutazione del professore può essere sintetizzata come segue:

**Utilizzo raccomandato:** i LLM si rivelano incrementibilmente utili per alleggerire le parti ripetitive del lavoro — generazione di script boilerplate, ricerca di sintassi per comandi poco frequenti, stesura di documentazione — e per le attività che richiedono di ricordare centinaia di dettagli sintattici.

**Utilizzo sconsigliato:** è totalmente sconsigliato inserire i LLM in procedimenti automatici senza supervisionare i risultati. Questa non è una novità esclusiva dell'intelligenza artificiale: l'automazione troppo spinta ha sempre causato problemi anche senza AI. Il riferimento citato è il caso CrowdStrike del 2024, considerato il maggiore outage IT della storia.

---

## 7. Alta Disponibilità e Data Center

### Il Contesto: On Premises vs Cloud

Le attività "tradizionali" vengono svolte *on premises*: l'infrastruttura è di proprietà dell'organizzazione e direttamente accessibile, con totale controllo ma con tutti i problemi connessi agli strati non "core" (alta disponibilità fisica, sicurezza, aggiornamento hardware). La collocazione realmente on premises è diventata progressivamente più rara, riservata ai casi in cui la complessità gestionale di esternalizzare non è giustificata, o quando non si possono delegare il controllo fisico dei dati e le scelte architetturali.

### Data Center e Server Farm

Il *data center* (o *server farm*) è il luogo fisico in cui vengono ospitati in grande quantità i sistemi di calcolo. I modelli di servizio offerti sono:

- **Housing (co-location):** fornitura di spazio e connettività per sistemi acquistati e gestiti dal cliente.
- **Managed housing:** fornitura dei sistemi in housing (su hardware dedicato al cliente) con gestione sistemistica inclusa.
- **Hosting:** fornitura di uno o più servizi specifici (storage, web, posta, ecc.) su hardware condiviso tra più clienti. **Il modello cloud è un caso speciale dell'hosting tradizionale.**

### Alta Disponibilità (HA) Fisica

I data center seri implementano misure di protezione contro molteplici categorie di rischio:

**Cause naturali:** terremoti, inondazioni, eventi atmosferici estremi — affrontati con criteri di costruzione specifici e scelta oculata della localizzazione geografica.

**Cause artificiali:** incidenti aerei e ferroviari, atti di terrorismo — affrontati con strutture rinforzate e localizzazione lontana da zone a rischio.

**Cause interne:** incendi — particolarmente critici perché i sistemi di soppressione stessi possono causare danni agli apparati (ad esempio i sistemi a gas inerte). Ogni sistema complesso introdotto per limitare i danni può causarne altri di imprevisti.

La **sicurezza e il controllo degli accessi** fisico prevedono: perimetro esterno blindato e sorvegliato da staff (eventualmente armato) 24/7, segmentazione dei settori con liste di controllo accessi separate e concordate in anticipo sull'ingresso, apertura dei varchi a più fattori di autenticazione, videosorveglianza con registrazione off-site.

### HA Ambientale

Il **condizionamento dell'aria** è un elemento critico: i server producono grandi quantità di calore e richiedono gestione precisa di temperatura e umidità. I sistemi adottati devono essere tolleranti ai guasti e alle interruzioni dell'alimentazione elettrica.

L'architettura tipica prevede *hot aisle* e *cold aisle* alternati tra le file di rack, con unità CRAC (Computer Room Air Conditioner) che aspirano l'aria calda e reimmettono aria fredda. Le varianti includono *Hot Aisle Containment* e *Cold Aisle Containment* per aumentare l'efficienza.

### HA Infrastrutturale (Alimentazione Elettrica)

L'alimentazione elettrica è gestita attraverso più livelli di ridondanza:

- **Erogazione su almeno due linee indipendenti** per ogni apparato, tramite Power Distribution Unit (PDU).
- **Pulizia della sinusoide** per allungare la vita degli apparati.
- **Sistemi di continuità a intervento istantaneo:** batterie UPS (*Uninterruptible Power Supply*) — intervento immediato ma capacità limitata (minuti).
- **Sistemi di continuità a lunga durata:** motogeneratori diesel — avviamento lento (decine di secondi) ma autonomia prolungata (ore o giorni con rifornimento).
- **Gestione dei consumi:** la metrica PUE (*Power Usage Effectiveness*) misura l'efficienza energetica del data center.

### HA della Connettività

La connettività di rete ridondante si realizza attraverso connessioni con provider indipendenti: i data center principali si avvalgono tipicamente di oltre 10 carrier differenti e spesso fungono da *internet exchange point*. Altrettanto importante è la **collocazione fisica dei cavi su percorsi indipendenti**, per evitare che un singolo guasto fisico (es. scavo accidentale) interrompa tutte le connessioni contemporaneamente.

---

## 8. Disponibilità, Downtime e SLA

### Definizione di Disponibilità

La *disponibilità* (o *availability*) di un sistema è l'indicatore fondamentale della qualità del servizio erogato. È definita come:

```
A = U / O
```

dove **U** (*uptime*) è il tempo per cui il sistema eroga correttamente i servizi e **O** è il tempo di osservazione complessivo. Il complemento a 1 di A è il *downtime ratio*.

### Cause di Downtime

La distribuzione tipica delle cause di downtime per incidenza sul totale è:

| Causa | Incidenza |
|---|---|
| Software | 40% |
| Downtime pianificati | 30% |
| Persone | 15% |
| Hardware | 10% |
| Ambiente | 5% |

Il dato più rilevante è che il software è responsabile della maggioranza dei downtime, e che i downtime pianificati (manutenzioni programmate) rappresentano quasi un terzo del totale — sottolineando l'importanza di pianificare correttamente le finestre di manutenzione.

### I "Nines": Livelli di Disponibilità

La disponibilità viene convenzionalmente espressa tramite il "numero di 9" nella percentuale di uptime. Aggiungere un 9 significa dividere per 10 il downtime ammissibile:

| Disponibilità | Downtime/anno | Downtime/mese | Downtime/settimana |
|---|---|---|---|
| 98% | 7,3 giorni | 14,4 ore | 3,36 ore |
| 99% | 3,65 giorni | 7,20 ore | 1,68 ore |
| 99,5% | 1,83 giorni | 3,60 ore | 50,4 minuti |
| 99,9% | 8,76 ore | 43,2 minuti | 10,1 minuti |
| 99,99% | 52,6 minuti | 4,32 minuti | 1,01 minuti |
| **99,999%** | **5,26 minuti** | **25,9 secondi** | **6,05 secondi** |
| 99,9999% | 31,5 secondi | 2,59 secondi | 0,605 secondi |

Il livello "five nines" (99,999%) è spesso citato come obiettivo di riferimento per sistemi mission-critical.

### Service Level Agreement (SLA)

Un *Service Level Agreement* è un contratto formale sul livello di servizio che va ben oltre la semplice garanzia di uptime. Un SLA può specificare:

- **Distribuzione del downtime:** ad esempio, "four nines nell'arco dell'anno, ma in frazioni non superiori a *n* minuti consecutivi" — prevenendo così un singolo outage prolungato che consumi tutto il budget di downtime annuale.
- **Parametri di servizio non collegati al downtime:** prestazioni dei servizi (latenza, throughput), tipologie di assistenza previste e loro tempi di risposta (SLO, *Service Level Objectives*).
- **Modalità di aggiornamento del contratto** e reportistica periodica sulle variabili monitorate.

---

## 9. Business Continuity

### Tecniche Fondamentali

Per garantire l'accessibilità senza interruzioni di dati e servizi, sono state sviluppate diverse tecniche che operano a livelli architetturali differenti:

- **RAID** (*Redundant Array of Independent Disks*) — robustezza dei supporti di storage mediante ridondanza.
- **LVM** (*Logical Volume Manager*) — flessibilità nell'allocazione e nel ridimensionamento dello storage senza downtime.
- **Multipath** — robustezza nell'accesso allo storage tramite percorsi fisici multipli verso lo stesso dispositivo.
- **Clustering** — ridondanza dei nodi di elaborazione, in modo che il guasto di un nodo non interrompa il servizio.

### I Limiti delle Tecniche di HA

Le tecniche di alta disponibilità difendono efficacemente contro guasti hardware ordinari, ma non proteggono da eventi improbabili ma catastrofici (cataclismi naturali, attentati) né da eventi limitati ma frequenti come gli errori degli operatori o gli attacchi informatici. Questi ultimi possono compromettere l'accessibilità a lungo termine dei dati.

La risposta a questi rischi sono i **backup** e i **piani di disaster recovery**: procedure che garantiscono la possibilità di ripristinare i dati e i servizi anche a seguito di eventi catastrofici, con obiettivi di RPO (*Recovery Point Objective*) e RTO (*Recovery Time Objective*) definiti contrattualmente.

---

## 10. Cloud Computing: Fondamenti

### Definizione ed Evoluzione

Il *cloud computing* rappresenta l'evoluzione naturale del modello di hosting tradizionale applicato su scala industriale. Ben prima dei LLM, il cloud ha modificato radicalmente il mondo dell'amministrazione di sistema.

Un cloud provider si fa carico della realizzazione di uno o più data center allo stato dell'arte — edifici, impianti, hardware di calcolo e networking di diverse fasce. L'insieme delle risorse fisiche viene utilizzato per far funzionare sistemi virtualizzati condivisi tra molti clienti (*multi-tenancy*).

### Caratteristiche Fondamentali

**Multi-tenancy:** molti clienti condividono le risorse fisiche, spalmando i costi fissi e delegando completamente la loro amministrazione infrastrutturale al provider.

**Astrazione dell'infrastruttura:** la configurazione avviene tramite interfacce che nascondono completamente la struttura fisica sottostante.

**Provisioning dinamico:** l'avvio e l'arresto delle risorse è *on demand*, con pagamento solo per il periodo di effettivo utilizzo.

**Scalabilità:** la dimensione del provider dà all'utente l'illusione di poter allocare illimitatamente nuove risorse al bisogno.

### Vantaggi Economici

Dal punto di vista economico, il cloud permette investimenti più flessibili rispetto all'acquisto di hardware proprio:

- Per **progetti piccoli o con fattori di utilizzo lontani dal 100%** (workload stagionali, concentrati in ore del giorno): l'acquisto di hardware dimensionato sul picco sarebbe inefficiente.
- Per **progetti medi e grandi** che richiederebbero un investimento in conto capitale difficile da sostenere.
- Per **progetti con aspettative di forte crescita** ma senza certezze sui tempi di concretizzazione.

Dal punto di vista gestionale, il cloud esternalizza le attività non-core (HA fisica, connettività, alimentazione, sicurezza fisica).

### I Protagonisti del Mercato Cloud

Il mercato del cloud infrastructure (IaaS, PaaS, hosted private cloud) è dominato da tre grandi provider — i cosiddetti "hyperscaler":

- **Amazon Web Services (AWS):** circa 30% del market share mondiale.
- **Microsoft Azure:** circa 20%.
- **Google Cloud Platform (GCP):** circa 13%.

Gli altri provider (Alibaba Cloud, Oracle Cloud, Salesforce, IBM, ecc.) si dividono il restante mercato.

---

## 11. Livelli di Servizio: IaaS, PaaS, SaaS

Il modello cloud si articola in tre livelli principali di astrazione, convenzionalmente denominati "*aaS" (Everything as a Service):

### IaaS — Infrastructure as a Service

Le risorse erogate sono componenti architetturali virtualizzate: capacità di calcolo (CPU, RAM), storage e dispositivi di networking. Il cliente gestisce il sistema operativo e tutto il software applicativo. Il provider si occupa solo dell'infrastruttura fisica e della virtualizzazione.

Esempi: **AWS EC2** (Virtual Servers in the Cloud), **AWS S3** (Scalable Storage), **AWS VPC** (Isolated Cloud Resources), **Google Compute Engine**, **Microsoft Azure Virtual Machines**.

### PaaS — Platform as a Service

Le risorse erogate sono intere piattaforme per l'esecuzione remota di codice caricato dall'utente: componenti e servizi standard fruibili da remoto (database gestiti, runtime per linguaggi, code di messaggi, API gateway, ecc.). Il cliente si occupa solo della logica applicativa, evitando di gestire l'intero stack sistemistico.

Esempi: **AWS Lambda** (serverless), **AWS API Gateway**, **AWS DynamoDB**, **Google Kubernetes Engine**, **Azure Web Apps**.

### SaaS — Software as a Service

Le risorse erogate sono applicazioni complete, rese disponibili via web agli utenti finali. Il cliente fornisce solo configurazione e dati; il provider gestisce tutto il resto.

Esempi: **Gmail**, **Dropbox**, **Salesforce**, **Microsoft Teams**, **G Suite**, **Azure Active Directory**.

### Il Client: l'Unico Componente On-Premises

In tutti i modelli cloud, i *client* (browser, applicazioni desktop, strumenti CLI) rimangono l'unico componente in esecuzione sulle piattaforme fisicamente in mano all'utente. Attraverso di essi, l'utente comunica con applicazioni, sistemi di deploy e sistemi di configurazione e monitoraggio delle infrastrutture, tramite due principali tipi di interfaccia: **API** (per l'automazione) e **Web GUI** (per l'utilizzo interattivo).

---

## 12. Prerequisiti Tecnici del Cloud

### Virtualizzazione: Il Fondamento

Il cloud computing poggia interamente sulla virtualizzazione. I prerequisiti tecnici sono:

**Pool di calcolatori fisici** architetturalmente simili e intercambiabili, su ciascuno dei quali gira un *hypervisor* (VMware vSphere, Xen, Microsoft Hyper-V) che gestisce le VM ospitate.

**Apparati di rete gestibili e riconfigurabili:** utilizzo massiccio di VLAN per partizionare il traffico tra tenant differenti; VXLAN per estendere il layer fisico su scala geografica; evoluzione verso il *Software Defined Networking* (SDN) per la gestione programmabile della rete.

**Sistemi di storage di rete** organizzati in gerarchie prestazioni/costo (SSD NVMe → SSD SATA → HDD) e ad alta scalabilità (object storage come Ceph, AWS S3).

### Gestione: Configuration as Code

La gestione di ambienti cloud su larga scala richiede strumenti specifici:

**Interfacce al sistema:** manuali via web GUI, via command line, e integrabili in piattaforme software tramite API REST.

**Sistemi di monitoraggio** dettagliati e facilmente accessibili, fortemente programmabili per reagire automaticamente agli eventi.

**Modelli di configuration management:** in un ambiente in cui i nodi formano un pool scalabile, non è sufficiente saper intervenire sulla configurazione di un singolo servizio; è necessario garantire modifiche coerenti a servizi interdipendenti e la propagazione delle modifiche sulle molteplici istanze in esecuzione. Le strategie adottate sono:

- *Distribuzione di parametri di configurazione* — utile per aggiornamenti semplici a effetto immediato.
- ***Configuration as code*** — la configurazione è codificata in file versionati, applicata in modo riproducibile su qualsiasi numero di nodi (Ansible, Puppet, Chef, Salt).
- *Immagini immutabili* — il sistema viene configurato una volta, trasformato in un'immagine disco, e le istanze in produzione vengono sostituite gradualmente con nuove versioni dell'immagine (approccio *test → template → sostituzione graduale*).

---

## 13. Gli Strumenti DevOps

### Il Paradigma DevOps

Il termine *DevOps* indica la convergenza tra i team tradizionalmente conflittuali degli sviluppatori (*developers*) e dei sistemisti (*operations*). L'obiettivo è la gestione delle risorse in modo riproducibile e automatico, anziché configurarle "artigianalmente" macchina per macchina.

### L'Ecosistema degli Strumenti

Il panorama degli strumenti DevOps è ricco e in rapida evoluzione. Le categorie principali sono:

**Configuration management e orchestrazione:**
- **Ansible** — orchestrazione agentless basata su SSH, playbook in YAML.
- **Puppet** — modello dichiarativo con agente, largamente diffuso in ambienti enterprise.
- **Chef** — configuration management basato su Ruby DSL.
- **SaltStack** — orchestrazione event-driven con comunicazione push/pull.

**Infrastructure provisioning:**
- **Terraform** — provisioning di infrastrutture cloud tramite linguaggio dichiarativo HCL (*Infrastructure as Code* a livello di cloud provider).
- **Vagrant** — creazione e gestione di ambienti di sviluppo virtualizzati riproducibili (usato in questo corso).
- **Packer** — creazione di immagini disco immutabili per molteplici piattaforme.

**Containerizzazione e orchestrazione di container:**
- **Docker** — containerizzazione delle applicazioni in unità isolate e portabili.
- **Kubernetes** — orchestrazione di cluster di container su larga scala.

**Continuous Integration/Continuous Deployment:**
- **Jenkins** — piattaforma CI/CD open source, largamente diffusa.

### Da Sysadm ad Architetto Cloud

La complessità degli ambienti cloud moderni — esemplificata dalla mappa dei servizi AWS (EC2, S3, VPC, Lambda, RDS, CloudFront, Route53, IAM, CloudWatch, ecc.) — ha determinato l'emergere della figura dell'*architetto cloud*, che progetta l'intera infrastruttura come un sistema integrato di servizi gestiti, andando ben oltre la tradizionale amministrazione di singoli server. Il sysadm moderno è chiamato a padroneggiare non solo i singoli strumenti, ma la logica complessiva dell'architettura distribuita.

---

*Nota: le sezioni relative al programma del corso, alle modalità d'esame, all'ambiente di laboratorio e alla predisposizione del PC personale sono trattate rispettivamente nei documenti "Introduzione al Corso e Setup del Laboratorio" e "Predisposizione del Proprio PC per le Esercitazioni".*
