# Accesso all'Hardware e Configurazione dello Storage (Integrato)

*Corso di (Laboratorio di) Amministrazione di Sistemi — Prof. Marco Prandini, Università di Bologna*

---

## INDICE

1. [Ruoli del Sistema Operativo](#1-ruoli-del-sistema-operativo)
2. [Device Driver e Kernel Modules](#2-device-driver-e-kernel-modules)
3. [Caricamento Dinamico dei Moduli](#3-caricamento-dinamico-dei-moduli)
4. [Funzionamento dei Moduli: Tipi di Dispositivo](#4-funzionamento-dei-moduli-tipi-di-dispositivo)
5. [Device Files (`/dev`)](#5-device-files-dev)
6. [Device Files di Uso Comune](#6-device-files-di-uso-comune)
7. [Il Sistema di Storage: Le Tre Operazioni](#7-il-sistema-di-storage-le-tre-operazioni)
8. [Partizionamento](#8-partizionamento)
9. [Master Boot Record (MBR)](#9-master-boot-record-mbr)
10. [MBR-EBR: Partizioni Estese e Unità Logiche](#10-mbr-ebr-partizioni-estese-e-unità-logiche)
11. [GPT (GUID Partition Table)](#11-gpt-guid-partition-table)
12. [Device File per Dischi e Partizioni](#12-device-file-per-dischi-e-partizioni)
13. [Criteri di Partizionamento](#13-criteri-di-partizionamento)
14. [Formattazione e Virtual File System](#14-formattazione-e-virtual-file-system)
15. [Filesystem ext2](#15-filesystem-ext2)
16. [Filesystem ext3](#16-filesystem-ext3)
17. [Filesystem ext4](#17-filesystem-ext4)
18. [Altri Filesystem Journaled: ReiserFS, XFS, JFS](#18-altri-filesystem-journaled-reiserfs-xfs-jfs)
19. [Il Futuro: btrfs](#19-il-futuro-btrfs)
20. [Mount: Gerarchia Locale e Gerarchia Globale](#20-mount-gerarchia-locale-e-gerarchia-globale)
21. [Sicurezza e Gestione dei Privilegi](#21-sicurezza-e-gestione-dei-privilegi)
22. [Esercizi di Laboratorio](#22-esercizi-di-laboratorio)

---

## 1. Ruoli del Sistema Operativo

Il sistema operativo svolge una varietà di ruoli, sinteticamente riconducibili a due macrocategorie:

**Astrazione delle risorse:**
- Fisiche (*device*): storage, porte USB, schede di rete, …
- Logiche: filesystem, stack di rete, …

**Controllo dell'accesso alle risorse:**
- Scheduling della CPU
- Allocazione della memoria
- Accesso a dispositivi
- Gestione dei permessi
- …

In Linux queste funzioni sono realizzate attraverso un framework complesso per essere attivabili e disattivabili modularmente. L'accesso all'hardware in particolare è astratto da moduli che implementano i *device driver*.

---

## 2. Device Driver e Kernel Modules

Alcuni driver sono cablati nel kernel (*built-in*); la maggior parte sono però implementati da **moduli del kernel dinamicamente caricabili**.

I moduli risiedono in `/lib/modules/<versione_kernel>/`.

**Comandi per la gestione dei moduli:**

| Comando | Funzione |
|---------|----------|
| `lsmod` | Elenca i moduli correntemente caricati nel kernel |
| `modinfo <modulo>` | Mostra le informazioni di un modulo (autore, licenza, alias, dipendenze…) |
| `insmod <file.ko>` | Carica manualmente un modulo dal file specificato |
| `modprobe <nome>` | Carica un modulo risolvendo automaticamente le dipendenze |

Il **codice del modulo** definisce:
- come "farsi trovare" (le stringhe identificative dei dispositivi fisici gestibili, tramite il campo `alias`);
- come sono implementate le versioni specifiche delle system call per il dispositivo gestito dal modulo.

Approfondimento: navigare nel codice sorgente del kernel Linux su https://elixir.bootlin.com/linux/latest/source

---

## 3. Caricamento Dinamico dei Moduli

Quando viene collegato un nuovo dispositivo fisico, la sequenza di caricamento del modulo corrispondente si articola in quattro passi:

1. **Rilevamento:** viene rilevato un dispositivo fisico.
2. **Interrupt:** il controller I/O manda un interrupt alla CPU.
3. **Kernel handler → dbus:** la CPU esegue l'interrupt handler (parte del kernel), che identifica l'evento e lo scrive su **dbus** (un canale publish-subscribe per tutti gli eventi di sistema).
4. **udev:** il demone `udev` riceve l'evento e, consultando il file `/lib/modules/<kernel_version>/modules.alias`, individua il modulo in grado di gestire il dispositivo e lo carica.

Il file `modules.alias` è generato dal comando `depmod`, che raccoglie tutti gli alias dichiarati nei moduli e li scrive in quel file.

**Esempio — output di `modinfo psmouse`:**
```
filename:   /lib/modules/4.13.0-37-generic/kernel/drivers/input/mouse/psmouse.ko
license:    GPL
description: PS/2 mouse driver
author:     Vojtech Pavlik <vojtech@suse.cz>
alias:      serio:ty05pr*id*ex*
alias:      serio:ty01pr*id*ex*
name:       psmouse
```

---

## 4. Funzionamento dei Moduli: Tipi di Dispositivo

Un modulo definisce in che modo vanno implementate le system call previste per la macro-categoria di dispositivi a cui appartiene. Il kernel dispone di una **tabella di puntatori** che associa ogni coppia (device, system call) alla specifica implementazione del modulo:

| syscall \ device | open | close | read | write | seek |
|------------------|------|-------|------|-------|------|
| psmouse | → | → | → | → | — |
| keyboard | → | → | → | → | — |
| disk | → | → | → | → | → |

Le tre macro-categorie di dispositivi sono:

### Dispositivi a blocchi (*block devices*)
- Utilizzano **buffer e cache** per ottimizzare il trasferimento di blocchi di dimensione data.
- Si comportano come file normali nel senso che "conservano" un elenco di byte singolarmente indirizzabili.
- **Supportano la ricerca casuale (seek)**.
- Esempi: dischi di vario tipo (IDE, SCSI, SATA, USB, virtuali…).

### Dispositivi a caratteri (*character devices*)
- Gestiscono il trasferimento dati **un carattere/byte alla volta**.
- Possono consumare caratteri e farci qualcosa (es. mostrarli su un terminale).
- Possono fornire caratteri se disponibili, o lasciare il consumatore in attesa (es. tastiera).
- **Non supportano la ricerca casuale (seek)**.

### Dispositivi vari (*misc*)
- Categoria residuale per dispositivi che non rientrano nelle due precedenti.

---

## 5. Device Files (`/dev`)

I punti di accesso alle periferiche sono astratti come **file speciali** salvati nella directory `/dev`. Sono memorizzati sul filesystem come normali inode, ma **non hanno data block associati**: su di essi, invocando le system call tipiche dei file, si scatenano le operazioni definite nel corrispondente device driver.

**Struttura dell'output di `ls -l /dev/sd*`:**
```
brw-rw---- 1 root  disk  8, 1 Mar 20 11:14 /dev/sda1
crw--w---- 1 las   tty  136, 0 Mar 20 11:17 /dev/pts/0
```

- La lettera iniziale (`b` o `c`) indica il tipo: **b**lock device o **c**haracter device.
- Il **major number** (es. `8`) individua quale device driver usare.
- Il **minor number** (es. `1`) identifica l'istanza specifica del dispositivo gestita da quel driver.

---

## 6. Device Files di Uso Comune

### Device files "virtuali" (non rappresentano periferiche reali)

Questi file sono implementati dal kernel e sono utili per lavorare con i processi:

| Device file | Comportamento |
|-------------|---------------|
| `/dev/zero` | Produce uno stream infinito di zeri binari |
| `/dev/null` | Ogni `read` restituisce EOF; ogni `write` viene scartata silenziosamente |
| `/dev/random` | Produce byte casuali ad alta entropia; **bloccante** se l'entropia disponibile non è sufficiente |
| `/dev/urandom` | Produce uno stream pseudocasuale illimitato (non bloccante) |

### Device files di periferiche notevoli

| Device file | Dispositivo |
|-------------|-------------|
| `/dev/tty*` | Terminali fisici del sistema |
| `/dev/pts/*` | Pseudo-terminali (dentro finestre del sistema grafico) |
| `/dev/sd*` | Dischi e partizioni (framework SCSI) |

---

## 7. Il Sistema di Storage: Le Tre Operazioni

I dispositivi a blocchi rappresentano tipicamente supporti di storage, sia reali (hard disk, SSD, USB drive, dischi ottici, …) sia logici (sistemi RAID, componenti LVM, …). Un dispositivo a blocchi è utilizzabile come un semplice elenco di blocchi dati di dimensione fissa, numerati.

Per renderlo fruibile per l'utente, sono necessarie **tre operazioni in sequenza**:

1. **Partizionamento** — Opzionale ma quasi sempre effettuato. Suddivide il dispositivo in sottoinsiemi.
2. **Formattazione** — Crea i metadati che organizzano lo spazio in modo comprensibile (filesystem).
3. **Mount** — Associa i singoli filesystem alla gerarchia di directory del sistema operativo.

---

## 8. Partizionamento

Il partizionamento suddivide un disco in **sottoinsiemi di blocchi**. Ogni partizione si presenta come un dispositivo indipendente. È utile per separare spazi con esigenze diverse:
- Di organizzazione (filesystem Linux vs. Microsoft vs. …)
- Di persistenza (reinstallazione sistema vs. dati utente)
- Di politiche di accesso (dati vs. programmi, …)

**Standard di partizionamento più diffusi su PC:**
- **MBR** (Master Boot Record) — lo standard tradizionale, con limite di 4 partizioni primarie e 2 TiB per partizione.
- **GPT** (GUID Partition Table) — lo standard moderno, usato con UEFI.

Altri standard esistenti ma meno diffusi: Extended Boot Record, Boot Engineering Extension Record, Apple Partition Map, Rigid Disk Block, BSD disklabel.

---

## 9. Master Boot Record (MBR)

L'MBR prende il nome dal passo essenziale per la modalità di avvio più frequente: il caricamento del sistema operativo da un hard disk.

**Struttura fisica dell'MBR (primo settore del disco, a partire dal byte `0x000`):**

```
0x000 ─────────────────────────────────
      Bootstrap (440 byte di codice
      caricato in memoria dal BIOS
      ed eseguito)
      ...
0x1BE ── First partition table entry ──
0x1CE ── Second partition table entry ─
0x1DE ── Third partition table entry ──
0x1EE ── Fourth partition table entry ─
```

**Struttura di ciascuna entry (×4):**
```
[attributi | indirizzo inizio CHS | tipo | indirizzo fine CHS | indirizzo inizio LBA | dimensione in settori]
```

**Proprietà chiave:**
- Spazio per descrivere al massimo **quattro partizioni primarie**.
- Una (al massimo) può essere indicata come *attiva* per essere riconosciuta come contenitore del bootloader da caricare per proseguire nel processo di avvio.
- L'ordine delle entry non deve necessariamente seguire la collocazione fisica dello spazio.

---

## 10. MBR-EBR: Partizioni Estese e Unità Logiche

Il **tipo** di una partizione è un'indicazione del filesystem che vi sarà contenuto (es. `83` = Linux, `82` = Swap).

Una delle quattro entry dell'MBR può avere come tipo **partizione estesa**: lo spazio occupato dalla partizione estesa contiene un'ulteriore tabella detta **Extended Boot Record (EBR)**.

**Struttura dell'EBR:**
- L'EBR usa solo due entry significative (le altre sono allocate ma inutilizzate):
  - La prima descrive l'**unità logica** (indirizzo inizio relativo alla partizione estesa, dimensione, ecc.).
  - La seconda (eventualmente) punta a un'ulteriore EBR, realizzando una **linked list** che permette di aggiungere teoricamente infinite unità logiche all'interno della partizione estesa (sarà riempita di zeri nell'ultima EBR della catena).

Dal punto di vista dell'utilizzo, **partizioni primarie e unità logiche sono equivalenti**.

**Nota sui gap:** i gap nella tabella delle partizioni possono rappresentare due effetti distinti:
- **Separazione volontaria:** lo spazio non deve essere necessariamente usato tutto; può essere lasciato libero per creare in futuro nuove partizioni o far crescere quelle esistenti. Vincolo fondamentale: se si vuole estendere un filesystem esistente **non deve mai essere modificato il suo punto di inizio**.
- **Spreco involontario:** inizio e fine delle partizioni potrebbero dover essere allineate ai confini di settore.

---

## 11. GPT (GUID Partition Table)

Lo standard GPT è utilizzato specialmente con UEFI (ma è utilizzabile anche con BIOS legacy).

**Confronto con MBR:**

| | MBR | GPT |
|---|---|---|
| Max partizioni | 4 primarie (+ logiche) | 128 |
| Max dimensione partizione | 2 TiB | 8 ZiB |

**Struttura fisica del disco con GPT:**
```
LBA 0:   Protective MBR (retrocompatibilità con tool legacy)
LBA 1:   Primary GPT Header
LBA 2-33: Partition entries 1-128
...
         Partitions 1, 2, ... N
...
LBA -34: Secondary partition entries (backup)
LBA -33: ...
LBA -2:  ...
LBA -1:  Secondary GPT Header (backup)
```

Oltre al limite di dimensione e al numero, ogni entry GPT contiene: Nome, GUID univoco, Attributi.

---

## 12. Device File per Dischi e Partizioni

### Dischi SCSI/SATA/USB (`/dev/sdXXNN`)

I dischi tradizionali sono comunemente unificati sotto il **framework SCSI**:
```
/dev/sdXXNN   es. /dev/sda1
```
- `XX` = una o più lettere che identificano il "disco" (a, b, c, …)
- `NN` = numero della partizione (1, 2, 3, …)

**Attenzione:** gli identificatori di disco possono cambiare a seconda dell'ordine in cui i dischi vengono rilevati al boot.

### Dischi NVMe/M.2 (`/dev/nvmeXnYpZ`)

I moderni dischi NVMe compaiono con una nomenclatura diversa:
```
/dev/nvmeXnYpZ   es. /dev/nvme0n1p2
```
- `X` = identificatore del "disco" (0, 1, 2, …)
- `Y` = identificatore del namespace (una sorta di macro-partizione hardware)
- `Z` = numero della partizione

---

## 13. Criteri di Partizionamento

### Partizione di Swap (memoria virtuale)

La swap è una partizione (o un file in una partizione dati) usata come **estensione della memoria virtuale**:
- In origine si consigliava di allocare il **doppio della RAM**, ma oggi la penalità di prestazioni è inaccettabile per usarla davvero come memoria (eccetto con NVMe veloci).
- **Uso sui laptop:** partizione dedicata di dimensione superiore alla RAM, necessaria per la **ibernazione** (il contenuto della RAM viene scritto su disco prima dello spegnimento).
- **Uso più comune oggi:** allocare un po' di spazio (anche come file *sparse*) solo per evitare che un piccolo esubero di uso della memoria mandi in crash l'intero sistema.

### Collocazione dati — approccio minimale

Un'unica partizione con spazio sufficiente per tutto.

### Collocazione dati per funzioni

Una partizione per ogni "tipo di accesso":
- `/` — partizione obbligatoria per la root
- Partizione per i file di boot
- Partizione per librerie e applicazioni → sola lettura
- Partizione per code e log → alto traffico in lettura e scrittura
- Partizione per aree utente → alto traffico e necessità di quota

---

## 14. Formattazione e Virtual File System

La formattazione **crea il filesystem** in una partizione: genera i metadati necessari per organizzare lo spazio in modo comprensibile al sistema.

**Proprietà fondamentali:**
- Richiede spazio contiguo; può crescere o ridursi ma non può avere "buchi".
- Permette l'accesso ai file secondo il modello del **Virtual File System (VFS)** di Linux.

### Virtual File System (VFS)

Il VFS è uno strato di astrazione del kernel che interpone tra i processi utente e i diversi filesystem specifici, consentendo di trattarli in modo uniforme:

```
User process
     ↓  system call (trap)
System calls interface
     ↓
     VFS
    / | \ \
Minix  DOS  ext  ext2   ← diversi FS specifici
     \
   Buffer Cache
     ↓
  Device drivers
     ↓
  I/O request
     ↓
Disk controller (Hardware)
```

---

## 15. Filesystem ext2

Il *second extended filesystem* è il più tradizionale in Linux.

**Caratteristiche:**
- **Non journaled**: è piuttosto robusto e veloce, ma non è adatto a filesystem di grandi dimensioni, poiché il minimo guasto richiederebbe ore per la rilevazione e la riparazione.
- Con blocchi da 4 KB, le dimensioni massime sono: **2 TiB per i singoli file**, **16 TiB per l'intero filesystem**.
- Struttura del FS Unix, con **inode** che supportano fino a due livelli di indirettezza.
- Poiché le directory non sono indicizzate (la ricerca dei file avviene sequenzialmente), è consigliabile non collocare più di **10.000–15.000 file** in una singola directory per non rallentare troppo le operazioni.

---

## 16. Filesystem ext3

ext3 è nato per essere essenzialmente **ext2 più un journal**, mantenendo la compatibilità col predecessore. Per questo motivo non esibisce le prestazioni dei filesystem nativamente journaled, ma rispetto a ext2 ha due vantaggi significativi:

- La possibilità di **crescere**, anche a caldo nelle ultime versioni.
- La possibilità di **indicizzare le directory con htree**, e quindi di gestire directory contenenti un maggior numero di file.

### Tre livelli di journaling in ext3

| Livello | Comportamento |
|---------|---------------|
| `journal` | Registra sia i dati che i metadati nel journal prima del commit sul filesystem. Massima sicurezza, minime prestazioni. |
| `ordered` | Registra solo i metadati, ma garantisce che ne sia fatto il commit solo dopo che i dati corrispondenti sono stati scritti sul FS. Compromesso equilibrato. |
| `writeback` | Registra solo i metadati senza alcuna garanzia sui dati. Massime prestazioni, minima sicurezza. |

---

## 17. Filesystem ext4

ext4 è l'ultima evoluzione stabile del filone "ext", introdotta nel kernel 2.6.28. Offre retrocompatibilità con ext3 pur introducendo importanti novità.

### Limiti aumentati rispetto a ext3

| | ext3 | ext4 |
|---|---|---|
| Dimensione max file | 2 TiB | 16 TiB |
| Dimensione max filesystem | 16 TiB | 1 EiB |
| Max subdirectory | 32.000 | illimitate |

### Nuovi concetti introdotti

- **Extent** — al posto del sistema di allocazione indiretta: i file grandi sono allocati in modo contiguo anziché essere suddivisi in moltissimi blocchi indirizzati in modo indiretto.
- **Allocazione dei blocchi a gruppi** — l'allocatore dei blocchi può riservarne più di uno per volta.
- **Allocazione ritardata** (*delayed allocation*) — l'allocatore è invocato solo quando c'è l'effettiva necessità di scrivere su disco. **Attenzione:** di default riduce la resistenza agli spegnimenti bruschi.
- **Journal checksumming** — la consistenza del journal è ottenuta con un checksum invece che con una procedura di commit a due fasi, migliorando affidabilità e velocità.

### Caratteristiche per sistemi embedded e real-time

- **Journal disattivabile** — elimina la causa delle maggiori lentezze di accesso ai dischi SSD e l'eccesso di scritture concentrate in pochi blocchi (usura).
- **Inode più grandi** — molti attributi estesi possono essere ospitati direttamente nell'inode.
- **Inode reservation** — aumenta il determinismo dei tempi di creazione dei file nelle directory.
- **Persistent preallocation** — aumenta il determinismo dei tempi di scrittura dei dati in un file.

> **Nota pratica:** di default la formattazione ext4 riserva il **5% dei blocchi a root** — strategia utilissima per evitare situazioni critiche, ma uno spreco per partizioni dati pure. È disattivabile con `tune2fs`.

---

## 18. Altri Filesystem Journaled: ReiserFS, XFS, JFS

### ReiserFS

Il primo filesystem journaled ad essere inserito nel kernel Linux. Teoricamente molto performante su sistemi che gestiscono grandi quantità di file di piccole dimensioni. Ha tuttavia attratto critiche per la sua stabilità e lo sviluppatore capo ha perso definitivamente credibilità per i suoi guai giudiziari.

### XFS

Sviluppato da **Silicon Graphics** per IRIX. Stabile, maturo, nativamente a 64 bit (su SO altrettanto a 64 bit può reggere partizioni da 8 ExaByte). Offre diverse ottimizzazioni molto vantaggiose. La scarsa diffusione è probabilmente dovuta alla consuetudine della maggior parte degli utenti Linux verso ext2/3.

### JFS

Sviluppato da **IBM** per AIX. Caratteristiche simili a XFS in termini di stabilità, maturità, 64 bit e ottimizzazioni.

---

## 19. Il Futuro: btrfs

**B-tree FS** (abbreviato in *btrfs*, pronunciato "Butter F S") è un progetto sviluppato da Oracle sotto licenza GPL: http://btrfs.wiki.kernel.org/

Gli obiettivi sono quelli di integrare funzionalità per la scalabilità enterprise, simili a quelle offerte da **ZFS** di Sun:

- **Pooling di risorse HW, multi-device spanning** — un filesystem può estendersi su più dispositivi fisici.
- **Copy-on-write (CoW)** — versioning, writable snapshots, uso di supporti read-only.
- **Checksum su dati e metadati** — integrità dei dati garantita strutturalmente.
- **Compressione/deframmentazione/checking online** — operazioni di manutenzione senza downtime.

---

## 20. Mount: Gerarchia Locale e Gerarchia Globale

### Concetto

Il partizionamento e la formattazione creano una **gerarchia locale** di directory all'interno di ciascuna partizione. L'operazione di **mount** "innesta" tale gerarchia locale in un punto specifico (detto *mount point*) della **gerarchia globale delle directory** di Linux.

### Esempio con quattro partizioni

```
1) mount /dev/sda1 /
2) mount /dev/sda5 /home
3) mount /dev/sdb2 /mnt/tools
4) mount /dev/sdc1 /mnt/tools/ext/new
```

Il risultato è che il percorso assoluto di un file è determinato dalla **combinazione della sua posizione relativa nel device** e del **mount point del device stesso**. Ad esempio, il file `shiny` nella root di `/dev/sdc1`, dopo il mount (4), sarà accessibile come `/mnt/tools/ext/new/shiny`.

### Mount permanente tramite `/etc/fstab`

Per rendere un mount persistente ai riavvii, aggiungere una riga a `/etc/fstab`:

```
/dev/sdb1  /punto_di_mount  ext4  defaults  0  2
```

I campi sono: dispositivo, mount point, tipo filesystem, opzioni, dump flag, fsck order.

---

## 21. Sicurezza e Gestione dei Privilegi

### Principio di minimo privilegio

Anche quando si lavora come sysadmin, la quantità di operazioni da svolgere con i privilegi di *root* è molto limitata. È fortemente raccomandabile utilizzare sempre l'account standard, che anche in caso di errori può provocare danni molto limitati.

### `su` (switch user)

Dopo il login, è possibile cambiare identità con il comando `su` (*switch user*), che permette di "diventare" l'utente specificato come parametro (a patto di conoscerne la password). Senza parametri equivale a `su root`. Il parametro `-` carica l'ambiente dell'utente di destinazione:

```bash
su -         # diventa root caricando il suo ambiente
su - mario   # diventa mario caricando il suo ambiente
```

### `sudo` (super-user do)

`sudo` permette di eseguire un **singolo comando** come un altro utente (incluso root) **senza conoscerne la password**, a patto di essere stati preventivamente autorizzati tramite la configurazione in `/etc/sudoers`.

Vantaggi rispetto a `su`:
- L'utente non deve conoscere la password di root.
- Ogni comando eseguito con `sudo` viene **registrato nel log di sistema**, garantendo tracciabilità completa delle operazioni privilegiate.

---

## 22. Esercizi di Laboratorio

### Esercizio 1 — Partizionamento, Formattazione, Mount

**Obiettivo:** Creare da zero una partizione usabile su un nuovo disco virtuale.

1. Spegnere la VM Xubuntu.
2. In VirtualBox, connettere un nuovo disco virtuale da **1 GB** (dinamicamente allocato). Collocare il file in `~/large`.
3. Avviare la VM e verificare come viene riconosciuto il nuovo disco:
   ```bash
   ls -l /dev/sd*
   ```
4. Creare una partizione primaria che occupi interamente il disco (tag `83` = Linux):
   ```bash
   sudo fdisk /dev/sdb    # premere 'm' per il manuale interno
   ```
5. Formattare la partizione con ext4:
   ```bash
   sudo mkfs.ext4 /dev/sdb1
   ```
6. Creare il mount point e montare la partizione:
   ```bash
   sudo mkdir /nuovo
   sudo mount /dev/sdb1 /nuovo
   df -h    # verificare l'esito
   ```

---

### Esercizio 2 — Layout MBR-EBR con Gap e Swap

**Obiettivo:** Riprodurre il seguente layout su un nuovo disco da 1 GB:

```
┌──────────┬──────────────────┬──────────────────┬─────────────────────┐
│ 100 MB   │   PRIMARIA       │   400 MB         │  UNITÀ LOGICA       │
│ Non alloc│   400 MB         │   Non allocato   │  100 MB             │
│          │   Tag: Linux     │                  │  Tag: Linux Swap    │
└──────────┴──────────────────┴──────────────────┴─────────────────────┘
```

**Procedura:** connettere un disco da 1 GB (~/large), avviare la VM, verificare il riconoscimento con `ls -l /dev/sd*`, creare la struttura con `fdisk` usando partizioni temporanee per gestire i gap.

---

### Esercizio Domanda — Resize Non Distruttivo

**Scenario:** a partire dal layout dell'Esercizio 2, si vuole trasformare la partizione `/dev/sdb1` da 1 GB a 500 MB, lasciando 100 MB liberi e creando una nuova partizione da 100 MB:

```
Prima: [Partizione 1 (1 GB)]
Dopo:  [Part.1 (500MB)] [Free 100MB] [Part.2 (100MB)] [Free (300MB)]
```

**Domanda:** è possibile eseguire questa trasformazione in modo **non distruttivo** (senza perdita di dati)?

**Risposta:** sì, a condizione di seguire l'ordine **Software → Hardware**:
1. `sudo umount /dev/sdb1` — smontare la partizione.
2. `sudo fsck -f /dev/sdb1` — controllo integrità (obbligatorio prima del resize).
3. `sudo resize2fs /dev/sdb1 500M` — ridurre il filesystem a 500 MB.
4. Rientrare in `fdisk`: cancellare la partizione 1 e ricrearla con la nuova dimensione (500 MB), partendo dallo **stesso identico settore iniziale**. Rispondere **NO** alla domanda sulla rimozione della signature ext4.
5. Creare la nuova partizione nel spazio liberato.
6. Per il mount permanente, aggiornare `/etc/fstab`.

---

*Nota: per le procedure di partizionamento avanzato con MBR-EBR vedere anche il file `LAB/Partizionamento Avanzato, MBR-EBR e Area di Swap.md`.*
