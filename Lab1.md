# Relazione di Laboratorio: Introduzione all'Amministrazione di Sistemi e Gestione dello Storage

---

## Obiettivo dell'Esercitazione

L'obiettivo di questa prima sessione pratica è stato duplice: da un lato, prendere confidenza con l'ambiente virtualizzato necessario per svolgere in sicurezza le attività di amministrazione di sistema; dall'altro, comprendere e applicare concretamente i tre concetti fondamentali per la gestione dello storage in Linux: **partizionamento**, **formattazione** e **mount**.

---

## Fase 1: Predisposizione dell'Ambiente e Virtualizzazione

Poiché molte delle attività di amministrazione richiedono i privilegi dell'utente `root` e possono potenzialmente causare danni alla macchina o comprometterne la stabilità (come la modifica dei filesystem o la cancellazione di file di sistema), si è scelto di operare all'interno di un ambiente sicuro sfruttando a fondo la **virtualizzazione** tramite **VirtualBox**.

È stata creata una **macchina virtuale (VM)** installando il sistema operativo **Xubuntu** (distribuzione basata su Debian) in modalità interattiva. Durante l'installazione automatica iniziale (opzione *"Erase disk"*), il sistema ha preso in carico la gestione del primo disco virtuale, applicando in autonomia le operazioni di partizionamento, formattazione e mount per creare la cartella radice `/`.

---

## Fase 2: Esplorazione dei Device Files e Astrazione Hardware

In Linux, il sistema operativo astrae l'accesso all'hardware utilizzando dei moduli del kernel (**device driver**). Le periferiche fisiche sono rappresentate nel sistema tramite file speciali chiamati **device files**, collocati nella cartella `/dev`.

Tramite il comando `ls -l /dev/sd*` è stato possibile ispezionare come il sistema riconosce i dischi. Dall'output si sono evinte importanti nozioni teoriche:

### Nomenclatura SCSI

I dischi e le relative partizioni sono comunemente unificati sotto il framework SCSI e identificati con la sigla `/dev/sdXXNN`. Ad esempio, `/dev/sda` rappresenta il primo disco fisico intero, mentre `/dev/sda1` e `/dev/sda2` ne rappresentano le partizioni.

### Dispositivi a Blocchi

La lettera iniziale `b` nei permessi (es. `brw-rw----`) indica che si tratta di **dispositivi a blocchi**, i quali utilizzano buffer e cache per trasferire dati in blocchi di dimensione fissa.

### Major e Minor Numbers

I numeri separati da virgola (es. `8, 0` per `sda`) indicano rispettivamente il **major number** (che definisce quale device driver usare) e il **minor number** (che identifica l'istanza specifica del dispositivo).

---

## Fase 3: Aggiunta di un Nuovo Disco e Partizionamento

A macchina spenta, è stato "collegato" un nuovo disco rigido virtuale da **1 GB**, allocato dinamicamente, per simulare l'aggiunta di nuovo hardware. Al riavvio, il sistema lo ha riconosciuto automaticamente assegnandogli il device file sequenziale `/dev/sdb`.

Essendo un disco "nudo", la prima operazione necessaria è stata il **partizionamento**, ovvero la suddivisione del disco in sottoinsiemi di blocchi. Questa operazione è utile per presentare porzioni di disco come dispositivi indipendenti.

Tramite l'utility interattiva `fdisk`, si è operato (usando i privilegi di amministratore tramite `sudo`) per creare una singola **partizione primaria** grande quanto l'intero disco. È stato inoltre applicato il tag esadecimale **`83`**, che identifica esplicitamente una partizione di tipo *"Linux"*. L'avvenuta creazione ha generato il nuovo block device `/dev/sdb1`.

---

## Fase 4: Formattazione

Una partizione vuota necessita di una struttura per ospitare i dati. La **formattazione** serve proprio a creare il **filesystem**, ovvero a generare i metadati necessari per organizzare lo spazio in modo comprensibile per il sistema.

Si è scelto di formattare la partizione `/dev/sdb1` utilizzando il comando `mkfs.ext4`. Il filesystem **ext4** è l'evoluzione più recente del filone *"ext"*; rispetto ai predecessori offre:

- Limiti di dimensione molto più elevati (sia per i singoli file che per il filesystem totale)
- Funzionalità avanzate come l'**allocazione ritardata dei blocchi** e il **journal checksumming**

---

## Fase 5: Mount del Filesystem

L'ultimo passaggio per rendere fruibile il dispositivo è il **mount**. Mentre il partizionamento e la formattazione creano una gerarchia locale all'interno della partizione, l'operazione di mount serve a "innestare" tale gerarchia locale in un punto specifico della **gerarchia globale delle directory** di Linux.

A tal fine, è stata creata una directory vuota nella root del sistema (`/discoB`, utilizzata come **mount point**). Eseguendo il comando:

```bash
mount /dev/sdb1 /discoB
```

il sistema operativo ha associato il filesystem appena creato a quella specifica directory. L'esito positivo dell'operazione è stato infine verificato consultando l'occupazione dei dischi del sistema.

---

> **Riepilogo del flusso operativo:** Aggiunta disco → Partizionamento (`fdisk`) → Formattazione (`mkfs.ext4`) → Mount (`mount`) → Verifica
