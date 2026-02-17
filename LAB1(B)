# Relazione di Laboratorio: Partizionamento Avanzato, MBR/EBR e Area di Swap (Esercizio 2)

---

## Obiettivo dell'Esercitazione

L'obiettivo di questa seconda fase è stato esplorare scenari di partizionamento complessi su un nuovo dispositivo a blocchi. Nello specifico, si è voluto applicare il concetto di **separazione volontaria dello spazio** (creando blocchi "non allocati"), superare i limiti strutturali dello standard **MBR** creando partizioni estese e logiche, e infine predisporre un'area di **memoria virtuale (Swap)**.

---

## Fase 1: Identificazione del Disco e Separazione Volontaria (Gap)

A macchina spenta è stato aggiunto un terzo disco virtuale da **1 GB**. Al riavvio, il kernel di Linux, tramite il modulo (device driver) competente, ha generato il relativo device file a blocchi assegnandogli sequenzialmente l'identificativo `/dev/sdc`.

L'esercizio richiedeva di posizionare la prima partizione non all'inizio del disco, ma lasciando **100 MB** di spazio vuoto iniziale, e di creare un ulteriore "buco" di **400 MB** in seguito.

### Richiamo Teorico: Gap e Separazione Volontaria

La presenza di questi spazi vuoti (**gap**) non è un errore, ma rappresenta una **separazione volontaria dello storage**. Lo spazio non deve essere necessariamente utilizzato tutto subito; può essere lasciato libero di proposito per creare in futuro nuove partizioni o per far crescere dimensionalmente quelle esistenti, rispettando il vincolo che una partizione deve sempre essere uno **spazio contiguo**.

Per realizzare questi gap in modo accurato con l'utility `fdisk`, si è optato per la creazione di **partizioni temporanee** (successivamente eliminate con il comando `d`) al fine di forzare il corretto allineamento dei settori fisici.

---

## Fase 2: Il Limite del Master Boot Record (MBR)

Si è proceduto alla creazione della partizione primaria `/dev/sdc2` da **400 MB**, assegnandole il tag standard Linux (codice **`83`**).

### Richiamo Teorico: Struttura dell'MBR

Il disco utilizza lo standard di partizionamento basato su **Master Boot Record (MBR)**. Nell'MBR (situato nel primissimo settore del disco, a partire dal byte `0x000`) vi è spazio fisico per descrivere al massimo **quattro partizioni primarie**. Avendo occupato gli slot nella tabella, il sistema ha correttamente impedito la creazione di una quinta partizione primaria, obbligando all'utilizzo del paradigma **MBR-EBR**.

---

## Fase 3: Partizioni Estese e Unità Logiche (Paradigma MBR-EBR)

Per poter allocare ulteriori spazi (la partizione da **100 MB** finale) superando il limite delle 4 partizioni dell'MBR, è stato necessario modificare la struttura della tabella.

### Richiamo Teorico: Extended Boot Record e Linked List

Per aggirare il limite dell'MBR, una delle quattro entry (nel nostro caso la numero 4) è stata dichiarata come **"Partizione Estesa"** (`/dev/sdc4`). Una partizione estesa funge esclusivamente da **"contenitore"**. Al suo interno viene creata un'ulteriore tabella chiamata **Extended Boot Record (EBR)**.

L'EBR funziona come una **lista concatenata (linked list)**: descrive una **"Unità Logica"** (nel nostro caso `/dev/sdc5`) e può puntare a un'altra EBR successiva, permettendo di aggiungere teoricamente **infinite unità logiche**.

Dal punto di vista del sistema operativo e dell'utente finale, le partizioni primarie e le unità logiche sono **perfettamente equivalenti** nel loro utilizzo pratico.

---

## Fase 4: Configurazione della Memoria Virtuale (Swap)

L'unità logica appena creata (`/dev/sdc5`) è stata configurata assegnandole il tag esadecimale **`82`**, che la identifica come partizione di tipo **Swap**.

### Richiamo Teorico: Ruolo e Dimensionamento della Swap

La partizione di Swap è un'area del disco dedicata a fungere da **estensione della memoria virtuale**. Viene utilizzata dal sistema operativo per spostare temporaneamente dati dalla memoria RAM al disco fisico quando la RAM è satura.

Se in passato era consuetudine allocare uno spazio di Swap pari al doppio della RAM, oggi (a causa della lentezza dei dischi tradizionali rispetto alla memoria solida) si tende ad allocare una quantità di spazio più contenuta, utile principalmente per evitare che un piccolo esubero improvviso nell'uso della memoria causi il crash dell'intero sistema.

---

Al termine delle configurazioni, il comando `w` ha scritto definitivamente la tabella delle partizioni sul settore iniziale del disco `/dev/sdc`, rendendo le modifiche **permanenti e persistenti** ai riavvii.

---

> **Riepilogo del flusso operativo:** Aggiunta disco → Creazione gap volontari (`fdisk` + partizioni temporanee) → Partizionamento primario (tag `83`) → Limite MBR raggiunto → Partizione Estesa + EBR → Unità Logica (tag `82`, Swap) → Scrittura tabella (`w`)
