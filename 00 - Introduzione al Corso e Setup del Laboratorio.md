# Introduzione al Corso e Setup dell'Ambiente di Laboratorio

*Corso di (Laboratorio di) Amministrazione di Sistemi — Prof. Marco Prandini, Università di Bologna*

---

## INDICE

1. [Agenda e Programma del Corso](#1-agenda-e-programma-del-corso)
2. [Modalità di Svolgimento delle Esercitazioni](#2-modalità-di-svolgimento-delle-esercitazioni)
3. [Utilizzo dei Laboratori](#3-utilizzo-dei-laboratori)
4. [Esercitarsi a Casa](#4-esercitarsi-a-casa)
5. [Ambiente per le Esercitazioni: Architettura di Virtualizzazione](#5-ambiente-per-le-esercitazioni-architettura-di-virtualizzazione)
6. [Installazione Virtuale di Xubuntu](#6-installazione-virtuale-di-xubuntu)
7. [Installazione del Software sull'Host](#7-installazione-del-software-sullhost)
8. [Accesso al Sistema e Gestione dei Privilegi](#8-accesso-al-sistema-e-gestione-dei-privilegi)

---

## 1. Agenda e Programma del Corso

Il primo incontro in laboratorio (17 febbraio 2026) è dedicato alla predisposizione immediata delle attività pratiche. La prima lezione in aula (19 febbraio 2026) presenta i contenuti del corso e le modalità di svolgimento di lezioni ed esami.

### Programma

Il corso si articola nelle seguenti macroaree:

**L'amministrazione e l'amministratore di sistema** — ruolo, responsabilità e competenze.

**Gestione locale dei sistemi Linux:**
la shell Bash come linguaggio per l'automazione dei compiti di amministrazione; device, filesystem, utenti e gruppi; installazione del software; gestione dei processi e monitoraggio delle risorse.

**Inserimento in rete dei sistemi Linux:**
configurazione statica e automatica di indirizzi e rotte; diagnostica e monitoraggio; servizi essenziali (NTP, DNS); accesso remoto.

**Interoperabilità e scalabilità:**
monitoraggio centralizzato (SNMP); autenticazione centralizzata (LDAP); configuration as code (Vagrant e Ansible).

---

## 2. Modalità di Svolgimento delle Esercitazioni

Molte delle attività del corso richiedono i **privilegi di amministratore** (modifica di permessi, invio di segnali, uso di funzioni di rete riservate, manipolazione dei filesystem). Operazioni di questo tipo possono di conseguenza causare danni (cancellazione di file di sistema, modifica di file di configurazione) e non sono quindi attuabili su macchine la cui stabilità deve essere garantita.

Per questa ragione si sfrutta a fondo la **virtualizzazione**: le esercitazioni si svolgono su macchine virtuali create e gestite dallo studente, all'interno delle quali è possibile operare come *root* senza rischi per il sistema host.

La modalità di lavoro è **in presenza in laboratorio**. Il LAB3 (come gli altri della Scuola) è attrezzato con postazioni che offrono VirtualBox. Lo stesso ambiente verrà utilizzato per lo svolgimento delle prove pratiche all'esame: è responsabilità degli studenti verificare preliminarmente che tutto funzioni correttamente.

---

## 3. Utilizzo dei Laboratori

### Accesso alle postazioni

Per utilizzare i PC dei laboratori (LAB 0, 2, 3, 4, 9) è necessario disporre delle **credenziali personali di Ingegneria**, ottenibili via web al sito https://infoy.ing.unibo.it/new_account.

Prerequisiti per ottenere le credenziali: essere regolarmente iscritti, aver pagato la rata delle tasse da almeno 3 giorni, e possedere le proprie credenziali di Ateneo (`nome.cognome@studio.unibo.it` oppure `nome.cognome@unibo.it`).

### LAB0 come alternativa

Qualora non si abbia modo di svolgere le esercitazioni a casa, è spesso disponibile il LAB0 (piano terra, corridoio principale del blocco storico). Tutti i laboratori, incluso LAB0, hanno la stessa configurazione e gli stessi prerequisiti: si può utilizzare LAB0 per testare le configurazioni in vista dell'esame.

### Nota sul proxy

Nei laboratori la navigazione avviene obbligatoriamente attraverso un proxy il cui indirizzo è `192.168.12X.249` dove X corrisponde al numero del laboratorio (0, 2, 3, 4, 9). Quando ci si sposta da un laboratorio all'altro, è necessario aggiornare l'indirizzo del proxy.

---

## 4. Esercitarsi a Casa

È possibile — e fa parte delle competenze da acquisire nel corso — riprodurre l'ambiente sul proprio computer personale. L'ambiente occupa qualche GB di spazio disco; per avere un margine di manovra adeguato si stimano circa **30 GB**. Il sistema host è preferibilmente GNU/Linux; verrà dato supporto nelle prime sessioni di laboratorio per chi desideri installare Linux in dual-boot sul proprio laptop (non garantito per Apple). È possibile utilizzare anche la nested virtualization, ma le prestazioni sono tipicamente molto scarse.

---

## 5. Ambiente per le Esercitazioni: Architettura di Virtualizzazione

### Finalità

Apprendere come configurare il sistema e i servizi principali di rete.

### Requisito

Più sistemi su cui operare come *root* e connessi in rete. Per limiti di tempo, saranno tutti identici (GNU/Linux).

### Architettura adottata

```
+----------------------------------------------+
|  Host (PC laboratorio o personale)           |
|  +----------------------------------------+  |
|  |  VirtualBox                            |  |
|  |  +----------+ +----------+ +--------+  |  |
|  |  | VM da    | | VM da    | | VM da  |  |  |
|  |  | vagrant  | | vagrant  | | vagrant|  |  |
|  |  | box uff. | | box uff. | | box uf.|  |  |
|  |  +----------+ +----------+ +--------+  |  |
|  +----------------------------------------+  |
+----------------------------------------------+
```

Verrà utilizzata la VM **Debian Bookworm** ufficiale. L'obiettivo è disporre di un host su cui funzionino tre software fondamentali per lanciare e configurare le VM:

- **VirtualBox** — https://www.virtualbox.org/
- **Vagrant** — https://www.vagrantup.com/
- **Ansible** — https://docs.ansible.com/

---

## 6. Installazione Virtuale di Xubuntu

L'installazione manuale di Linux in VirtualBox serve a prendere contatto con il concetto di virtualizzazione e a simulare ciò che si deve fare per installare Linux su un vero host ("bare metal", sia in esclusiva che in dual boot). Nel resto del corso si adotterà l'approccio **Infrastructure as Code** (IaC), definendo le VM tramite file di configurazione.

### Predisposizione del laboratorio

In laboratorio è già tutto pronto, con pochi ma importanti accorgimenti: l'host parte in modo testo; per avviare l'interfaccia grafica digitare `startx`. La prima volta che si usa il sistema di virtualizzazione, scaricare da Virtuale il file `setup_new.sh`, esplorarlo con un editor di testo e poi eseguirlo con `bash ./setup_new.sh`. **Lavorare sempre nella directory `large` sotto la propria home.**

### Procedura di creazione della VM

1. Avviare VirtualBox → Machine → New (Ctrl-N).
2. Scegliere un nome (es. LabHost).
3. Selezionare dal filesystem l'immagine ISO (si trova sotto `/opt/owa`).
4. **Spuntare** l'opzione "Skip Unattended Installation".
5. Verificare che la Memory size sia almeno **2 GB**.
6. In Hard Disk, verificare che il File Location sia dentro `VOSTRA_HOME/large`, la Size sia almeno **25 GB**, e che **non** sia spuntata l'opzione "Pre-allocate Full Size" (così verrà occupato solo lo spazio effettivamente utilizzato dalla VM).
7. Avviare la VM.

### Procedura di installazione di Xubuntu

1. Consigliato: lingua **inglese**.
2. Settare tastiera **italiana**.
3. Usare wired connection se sotto VirtualBox (o la connessione disponibile sul laptop per bare metal).
4. Selezionare **Interactive installation** (o de-selezionare Unattended installation).
5. Selezionare **Xubuntu Desktop** (in laboratorio va bene anche Xubuntu Minimal per risparmiare tempo).
6. In modalità bare metal si suggerisce di installare third-party software.
7. **Gestione dei dischi** — passaggio critico: selezionare "Erase disk and install Xubuntu" è del tutto sicuro sotto VirtualBox in laboratorio, ma ovviamente **non** in dual boot. L'opzione "Manual Installation" sarebbe più realistica per una vera installazione.
8. Definire il proprio account utente.
9. Selezionare il fuso orario.
10. Verificare le scelte e avviare l'installazione.

---

## 7. Installazione del Software sull'Host

Una volta ottenuto un host funzionante (in laboratorio è già tutto pronto), sul PC personale con una qualsiasi distribuzione GNU/Linux per Intel/AMD 64 (inclusa quella appena installata in dual boot), macOS (sia Apple Silicon sia Intel), o Windows 11, si procede ad installare **VirtualBox**, **Vagrant** e **Ansible**.

---

## 8. Accesso al Sistema e Gestione dei Privilegi

### Principio di minimo privilegio

Anche quando si lavora come sysadmin, la quantità di operazioni da svolgere con i privilegi di *root* è molto limitata. È fortemente consigliabile utilizzare sempre l'account standard, che anche in caso di errori può provocare danni molto limitati.

### su (switch user)

Successivamente al login, è possibile cambiare identità con il comando **`su`** (*switch user*), che permette di "diventare" l'utente specificato come parametro, a patto di conoscerne la password. Senza parametri, equivale a `su root`. È consigliabile usare il parametro `-` per caricare l'ambiente dell'utente di destinazione: il comando più comune sarà dunque `su -`.

### sudo (super-user do)

Oltre a `su`, esiste un modo più fine per utilizzare l'identità di *root*: **`sudo`** (*super-user do*) permette di eseguire un singolo comando come un altro utente (incluso root) senza conoscerne la password, a patto di essere stati preventivamente autorizzati (configurazione in `/etc/sudoers`).

Il vantaggio rispetto a `su` è che l'utente non deve conoscere la password di root e che ogni comando eseguito con `sudo` viene registrato nel log di sistema, garantendo tracciabilità completa delle operazioni privilegiate.
