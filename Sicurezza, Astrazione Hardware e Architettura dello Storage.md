# Riepilogo Teorico-Pratico: Sicurezza, Astrazione Hardware e Architettura dello Storage

---

## 1. Sicurezza e Amministrazione

Essere un amministratore di sistema significa avere un grande potere, che comporta grandi rischi.

### Il Rischio dei Privilegi

Molte attività di amministrazione (modifica di permessi, manipolazione filesystem, configurazioni di rete) richiedono i **privilegi massimi** e possono causare danni irreparabili, come la cancellazione di file vitali per la stabilità della macchina.

> **Esempio Pratico — L'Ambiente Sicuro:** Per non rischiare di compromettere il computer reale, si è evitata l'installazione *"bare metal"* e tutte le esercitazioni sono state svolte isolandole all'interno di una **macchina virtuale** con **Xubuntu**, appoggiandosi a **VirtualBox**.

### Gestione dell'Identità

La quantità di operazioni da svolgere costantemente come utente `root` è limitata, ed è raccomandato usare sempre un **account standard** e "cambiare identità" solo quando necessario.

> **Esempio Pratico — Uso di `sudo`:** Si è lavorato sempre con l'utente standard creato in fase di installazione. Tuttavia, per compiere azioni strutturali come gestire le partizioni (con `fdisk`) o montare dischi (con `mount`), si è dovuta delegare l'azione anteponendo il comando `sudo` (*super-user do*), inserendo la password per validare l'operazione.

---

## 2. Il Sistema Operativo e l'Hardware

Il sistema operativo astrae l'accesso all'hardware attraverso i **device driver**, implementati come moduli del kernel caricabili dinamicamente.

### Dispositivi a Blocchi

Utilizzano **buffer e cache** per trasferire dati in blocchi di dimensioni fisse e supportano la **ricerca casuale (seek)**, comportandosi come file con byte singolarmente indirizzabili.

### I Device Files (`/dev`)

I punti di accesso alle periferiche sono astratti come **file speciali** salvati nella directory `/dev` e, per i dischi sotto framework SCSI, seguono la nomenclatura `/dev/sdXXNN`.

> **Esempio Pratico — Esplorazione:** Interrogando il sistema con `ls -l /dev/sd*`, si sono ispezionati fisicamente questi device files. Si è notata la lettera `b` nei permessi (es. `brw-rw----`) che ne conferma la natura di "dispositivi a blocchi". Si sono inoltre letti i **major number** (es. l'`8`, che individua la categoria del driver) e i **minor number** (es. `0`, `1`, `2`, che identificano la specifica partizione o disco).

> **Esempio Pratico — Aggiunta Dischi:** "Agganciando" fisicamente due nuovi dischi virtuali da **1 GB** dalle impostazioni di VirtualBox, il sistema li ha riconosciuti al volo, assegnando loro le lettere successive nell'alfabeto e generando i file speciali `/dev/sdb` e `/dev/sdc`.

---

## 3. L'Architettura dello Storage (Le 3 Fasi)

Per utilizzare un dispositivo a blocchi per salvare dati, sono necessarie tre operazioni logiche in sequenza: **partizionamento**, **formattazione** e **mount**.

---

### Fase 1: Partizionamento

Suddivide il disco in sottoinsiemi per separare spazi con esigenze diverse, tenendo presente che se si vuole estendere un filesystem esistente **non deve mai essere modificato il suo punto di inizio**.

#### MBR ed EBR

Il **Master Boot Record (MBR)** ha spazio per descrivere al massimo **quattro partizioni primarie**. Per aggirare il limite, si usa una **Partizione Estesa** che contiene un **Extended Boot Record (EBR)**, il quale permette di concatenare ulteriori **Unità Logiche**.

#### Swap

È una partizione usata come **memoria virtuale** per evitare che un esubero di memoria mandi in crash il sistema.

> **Esempio Pratico — Esercizio 1:** Usando l'utility `fdisk`, si è manipolato il disco `sdb` creando una singola partizione primaria da 1 GB (`/dev/sdb1`), assegnandole il tag esadecimale **`83`** per indicare un formato Linux.

> **Esempio Pratico — Esercizio 2:** Sul disco `sdc`, si è lavorato con la **separazione volontaria dello spazio** creando partizioni e poi eliminandole (comando `d`) per posizionare le aree "Non allocate". Si sono raggiunti i limiti dell'MBR venendo bloccati alla quinta partizione primaria, risolvendo il problema inserendo una **Partizione Estesa** (`/dev/sdc4`) come contenitore per l'**Unità Logica** (`/dev/sdc5`). Infine, si è usato il tag **`82`** per indicare al sistema che quell'unità logica era un'area di **Swap**.

---

### Fase 2: Formattazione (Il Filesystem)

Crea i **metadati** che organizzano lo spazio, richiedendo spazio contiguo senza buchi all'interno della singola partizione.

#### Filesystem ext4

Evoluzione stabile della famiglia *"ext"* che:

- Aumenta i **limiti di dimensione** (fino a **1 EiB** per il filesystem)
- Migliora il **determinismo dei tempi** introducendo la **persistent preallocation** e l'**inode reservation**
- Usa il **journal checksumming** per l'affidabilità

> **Esempio Pratico:** Avendo la partizione vergine `/dev/sdb1`, la si è trasformata in uno spazio utilizzabile costruendovi sopra le fondamenta del filesystem tramite il comando:
> ```bash
> sudo mkfs.ext4 /dev/sdb1
> ```

---

### Fase 3: Mount

Il mount "innesta" la **gerarchia locale** di una partizione all'interno della **gerarchia globale delle directory** di Linux.

> **Esempio Pratico:** Dopo aver formattato la partizione, per potervi accedere si è creata una directory di appoggio vuota e vi si è agganciato sopra l'intero spazio del disco:
> ```bash
> sudo mkdir /discoB
> sudo mount /dev/sdb1 /discoB
> ```
> L'esito e le dimensioni reali in Gigabyte sono state confermate lanciando l'utility `df -h`.

---

> **Flusso complessivo:** Partizionamento (`fdisk`, tag `83`/`82`) → Formattazione (`mkfs.ext4`) → Mount (`mount` + mount point) → Verifica (`df -h`)
