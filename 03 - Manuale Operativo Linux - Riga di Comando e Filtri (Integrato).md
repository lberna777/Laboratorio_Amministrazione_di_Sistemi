# Manuale Operativo Linux: Riga di Comando e Filtri di Testo

**Appunti di Laboratorio — Teoria, Logica ed Esercizi Pratici**
*Corso di (Laboratorio di) Amministrazione di Sistemi — Prof. Marco Prandini, Università di Bologna*

---

## INDICE

1. [Login, Shell e Interprete dei Comandi](#1-login-shell-e-interprete-dei-comandi)
2. [Identificazione dell'Utente](#2-identificazione-dellutente)
3. [Esecuzione e Tipi di Comandi](#3-esecuzione-e-tipi-di-comandi)
4. [Alias e Priorità di Risoluzione](#4-alias-e-priorità-di-risoluzione)
5. [Ricerca dei Comandi Esterni e la Variabile PATH](#5-ricerca-dei-comandi-esterni-e-la-variabile-path)
6. [File di Configurazione della Shell](#6-file-di-configurazione-della-shell)
7. [Documentazione: man pages, info e HOWTO](#7-documentazione-man-pages-info-e-howto)
8. [Argomenti, Opzioni e Interazione con la Command Line](#8-argomenti-opzioni-e-interazione-con-la-command-line)
9. [History e il Comando script](#9-history-e-il-comando-script)
10. [Brevissima Introduzione all'Editor VIM](#10-brevissima-introduzione-alleditor-vim)
11. [Standard Streams, Pipeline e Variabili d'Ambiente](#11-standard-streams-pipeline-e-variabili-dambiente)
12. [Filtri: Concetti Generali](#12-filtri-concetti-generali)
13. [Concatenazione di File — cat e tac](#13-concatenazione-di-file--cat-e-tac)
14. [Impaginazione — less](#14-impaginazione--less)
15. [Inversione dei Caratteri — rev](#15-inversione-dei-caratteri--rev)
16. [Estrazione Verticale: head e tail](#16-estrazione-verticale-head-e-tail)
17. [Estrazione Orizzontale: cut](#17-estrazione-orizzontale-cut)
18. [Ordinamento e Unicità: sort e uniq](#18-ordinamento-e-unicità-sort-e-uniq)
19. [Conteggio: wc](#19-conteggio-wc)
20. [Ricerca di Pattern: grep ed Espressioni Regolari](#20-ricerca-di-pattern-grep-ed-espressioni-regolari)
21. [Modifiche Complesse: sed e tr](#21-modifiche-complesse-sed-e-tr)
22. [Elaborazione Avanzata dei Campi: awk](#22-elaborazione-avanzata-dei-campi-awk)
23. [Variazioni sul Tema Filtri: xargs, Process Substitution, tee, Command Substitution](#23-variazioni-sul-tema-filtri)
24. [Generazione Casuale: $RANDOM e shuf](#24-generazione-casuale-random-e-shuf)
25. [Strumenti Multi-File: diff, paste, join](#25-strumenti-multi-file-diff-paste-join)
26. [Laboratorio: Esercizi Svolti e Commentati](#26-laboratorio-esercizi-svolti-e-commentati)
27. [Appendice: Gestione Dischi e Partizioni](#27-appendice-gestione-dischi-e-partizioni)

---

## 1. Login, Shell e Interprete dei Comandi

Lo strumento più potente, flessibile e soprattutto più *standard* con cui amministrare un sistema Unix è l'interfaccia testuale a **riga di comando**.

La componente essenziale di tale interfaccia è la **shell**, ovvero l'interprete dei comandi. La shell svolge compiti di *job control*, mette a disposizione variabili e strutture di controllo per scrivere semplici programmi, e consente di invocare gli eseguibili installati nel sistema. In senso lato, possiamo definire shell qualsiasi interfaccia tra utente e sistema operativo.

Una delle shell più diffuse è **bash** (Bourne-again shell), un'evoluzione della classica Bourne Shell (`sh`) di Unix. Dopo l'autenticazione dell'utente, il sistema presenta un **prompt** — una stringa visualizzata nella parte iniziale della prima linea vuota — che indica la disponibilità dell'interprete ad accettare comandi. I caratteri digitati dall'utente dopo il prompt e terminati dal ritorno a capo costituiscono la **command line**.

---

## 2. Identificazione dell'Utente

Poiché è del tutto comune disporre di differenti identificativi utente, risulta utile potersi identificare in ogni momento tramite appositi comandi:

- `whoami` — restituisce il proprio username corrente.
- `id` — fornisce informazioni dettagliate sull'identità e sul gruppo di appartenenza (UID, GID, gruppi supplementari).
- `who` — elenca chi è attualmente collegato alla macchina.

---

## 3. Esecuzione e Tipi di Comandi

I comandi digitati sulla command line possono appartenere a cinque categorie distinte:

1. **Keyword** — un comando il cui compito è modificare l'esecuzione di altri comandi (ad esempio, misurare il tempo di esecuzione o innescare un ciclo).
2. **Built-in** — un comando direttamente eseguito dal codice interno della shell, senza bisogno di un file eseguibile esterno. Un esempio tipico è costituito dai comandi di navigazione del filesystem (`cd`, `pwd`).
3. **Comando esterno** — un file eseguibile che viene localizzato e messo in esecuzione dalla shell, tipicamente in un processo figlio. La shell può attenderne o meno la conclusione prima di accettare nuovi comandi.
4. **Alias** — una stringa che viene sostituita da un'altra prima dell'esecuzione.
5. **Funzione** — un'intera sequenza di comandi shell, con un nome, a cui possono essere passati parametri.

Per determinare quale tipo di comando viene effettivamente eseguito si utilizza il comando **`type`**:

```bash
$ type -a echo
echo is a shell builtin
echo is /bin/echo
```

L'opzione `-a` mostra tutte le forme disponibili per un dato nome.

---

## 4. Alias e Priorità di Risoluzione

La shell mette a disposizione il meccanismo degli **alias** per memorizzare linee di comando complesse sotto nomi più semplici da invocare:

```bash
alias miols='ls -l'
```

Le associazioni definite tramite `alias` vanno perse al termine della sessione (si veda la sezione sulla configurazione per renderle persistenti).

### Priorità di risoluzione e override

Una volta definiti, gli alias (e le funzioni) hanno la **priorità** rispetto ai built-in e ai comandi esterni omonimi. Per analizzare e pilotare la configurazione risultano utili i comandi `type`, `builtin`, `command` e `unalias`:

- Il **backslash** `\` davanti al comando previene *solo* l'espansione degli alias.
- La keyword **`builtin`** previene l'espansione degli alias e l'uso di funzioni, invocando l'esecuzione del built-in specificato (se esiste).
- La keyword **`command`** utilizza un comando esterno anche se esiste una funzione con lo stesso nome.
- Il comando **`unalias`** cancella un alias definito in precedenza.

Esempio pratico:

```bash
$ alias echo='echo ~~~'
$ echo test
~~~ test
$ \echo test        # \ previene solo l'alias
test
$ builtin echo test # builtin fa override di alias e funzioni
test
$ type echo
echo is aliased to 'echo ~~~'
$ unalias echo
$ type -a echo
echo is a shell builtin
echo is /bin/echo
```

---

## 5. Ricerca dei Comandi Esterni e la Variabile PATH

Per lanciare un eseguibile lo si può individuare tramite il percorso completo, sia in forma assoluta (`/usr/local/bin/top`) sia in forma relativa (`./mycommand`).

La shell utilizza la variabile d'ambiente **`PATH`** per la ricerca automatica dei comandi nel file system. La struttura di `PATH` è un elenco di directory separate dal carattere `:`:

```
PATH=/bin:/usr/bin:/sbin
```

In presenza di più eseguibili omonimi in directory diverse, la lista viene percorsa in ordine e il sistema usa la **prima istanza** trovata. Il comando **`which`** permette di sapere quale versione verrà effettivamente utilizzata:

```bash
# which passwd
/usr/bin/passwd
```

Per consentire l'esecuzione automatica di programmi presenti nella directory corrente, la variabile `PATH` dovrebbe contenere la directory `.` — tuttavia questa **non è una buona norma** di sicurezza, poiché è facile lanciare per distrazione comandi errati o potenzialmente malevoli. È preferibile usare sempre il percorso esplicito (`./nomeprogramma`).

---

## 6. File di Configurazione della Shell

Per ottenere automaticamente all'avvio della shell determinati comportamenti (assegnazione di variabili come `PATH`, definizione di alias, esecuzione di comandi di inizializzazione), si ricorre ai **file di configurazione di bash**. La documentazione completa si trova nelle sezioni INVOCATION e FILES di `man bash`.

**File globali** (validi per tutti gli utenti):

- `/etc/profile`
- `/etc/bash.bashrc`

**File personali** (nella home directory dell'utente):

- `.bash_profile`
- `.bash_login`
- `.profile`
- **`.bashrc`** — il più comunemente usato per personalizzazioni della sessione interattiva.

---

## 7. Documentazione: man pages, info e HOWTO

### man pages

Ogni applicazione installata fornisce **pagine di manuale** relative al suo utilizzo e configurazione. Si consultano con il comando `man`:

```bash
man <nome della pagina>
```

Spesso il nome della pagina coincide con il comando o il file di configurazione documentato. Le man page sono raggruppate in **sezioni**:

1. User commands
2. Chiamate al sistema operativo
3. Funzioni di libreria
4. File speciali (`/dev/*`)
5. Formati dei file, protocolli e relative strutture C
6. Giochi
7. Varie: macro, header, filesystem, concetti generali
8. Comandi di amministrazione riservati a *root*
9. (n) Comandi predefiniti del linguaggio Tcl/Tk

Opzioni utili:

- `man -a <comando>` — cerca in tutte le sezioni.
- `man <sez.> <comando>` — cerca nella sezione specificata.
- `man -k <keyword>` — cerca tutte le pagine attinenti alla parola chiave specificata.

I **built-in**, non essendo programmi installati indipendentemente, non hanno una man page dedicata. Un sommario del loro funzionamento si ottiene con `help <builtin>`; la documentazione completa risiede nella man page `bash(1)`.

### info files

A metà strada tra la man page e l'ipertesto, i file info si leggono con il comando `info`, che invoca l'editor emacs appositamente esteso per gestire tali file.

### HOWTO

Documenti specifici per la risoluzione dei più svariati problemi pratici, raccolti in un pacchetto installato in `/usr/[share/]doc/HOWTO`. In `/usr/doc/HOWTO/translations/it` si trovano le traduzioni italiane. Un punto di partenza online è il Linux Documentation Project (http://tldp.org/).

---

## 8. Argomenti, Opzioni e Interazione con la Command Line

Ogni comando, sia built-in che esterno, può accedere ai caratteri che seguono la propria invocazione sulla command line (la shell inserisce in memoria l'ARGV prima di generare il processo).

I gruppi di caratteri, **separati da spazi**, rappresentano gli **argomenti**: i dati su cui si vuole che il comando operi. Un argomento che inizia con il carattere `-` è chiamato solitamente **opzione**: non è un vero dato da elaborare, bensì una variante del comportamento del comando. Più opzioni possono solitamente essere raggruppate in un'unica stringa.

Esempi con `ls`:

```
ls /home              # arg #1="/home": directory da elencare
ls -l /home           # arg #1=opzione l (formato lungo)
ls -l -a /home/alex   # opzioni l (lunga) e a (all)
ls -la /home/alex     # identico: opzioni concatenate l+a
```

### Completamento automatico (TAB)

Iniziando a scrivere un comando o un nome di file e premendo **TAB**, bash completa automaticamente la stringa se non ci sono ambiguità, o suggerisce come completarla correttamente. In caso di ambiguità, una seconda pressione di TAB mostra i completamenti possibili.

---

## 9. History e il Comando script

### history

Il comando **`history`** mostra l'elenco di tutti i comandi eseguiti nel terminale. Per richiamarli sulla command line si usa la freccia-su (appaiono editabili, dal più recente al più vecchio).

La history è anche ricercabile interattivamente: al prompt basta digitare **CTRL-r** per attivare la *reverse-i-search*; digitando una stringa, verrà mostrato il comando più recente che la contiene. Premendo nuovamente CTRL-r si naviga verso comandi precedenti. Individuato il comando desiderato, si può lanciare direttamente con Invio oppure renderlo editabile con le frecce.

### script

Il comando **`script`** permette di catturare in un file l'intera sessione di lavoro al terminale, esattamente come compare a video (comandi impartiti e relativo output). Per terminare la cattura: digitare `exit` o premere **CTRL-d**.

---

## 10. Brevissima Introduzione all'Editor VIM

VIM è un editor a "tutto schermo", versione più amichevole dello storico VI. Utilizza un'interfaccia **modale**, per cui il programma si può trovare in uno dei seguenti stati:

- **COMMAND** — il cursore è posizionato sul testo; la tastiera è utilizzabile solo per richiedere l'esecuzione di comandi, non per introdurre testo. I caratteri digitati non vengono visualizzati.
- **INPUT** — tutti i caratteri digitati vengono visualizzati e inseriti nel testo.
- **DIRECTIVE** — il cursore si posiziona nella linea direttive (l'ultima linea del video) e si possono richiedere tutti i comandi per il controllo del file.

### Passaggi di stato

| Da | A | Tasti |
|---|---|---|
| COMMAND | INPUT | `o` `O` `i` `I` `a` `A` `C` `R` |
| INPUT | COMMAND | `<ESC>` |
| COMMAND | DIRECTIVE | `:` `/` `?` |
| DIRECTIVE | COMMAND | `<RET>` (Invio) |

---

## 11. Standard Streams, Pipeline e Variabili d'Ambiente

### Standard Streams

I comandi Linux comunicano attraverso flussi di dati (streams):

- **Standard Input (stdin)** — i dati in ingresso.
- **Standard Output (stdout)** — i risultati stampati a schermo.
- **Standard Error (stderr)** — i messaggi di errore.

### Pipeline

La **pipeline** (il simbolo `|`) è il concetto operativo più potente della shell: collega lo stdout del primo comando allo stdin del comando successivo, creando catene di elaborazione senza necessità di file temporanei intermedi.

### Variabili d'ambiente

Le variabili d'ambiente sono contenitori di memoria che il sistema usa per configurare la sessione dell'utente. Si visualizzano tutte con il comando `env`. Per richiamare il valore di una specifica variabile si antepone il simbolo `$`:

```bash
echo $PATH
```

---

## 12. Filtri: Concetti Generali

Il meccanismo di ridirezione è utilizzato da un insieme di comandi pensati esattamente per elaborare stream di testo ricevuti via stdin, producendo risultati su stdout: i **filtri**.

Molti di questi consentono comunque di operare direttamente anche su file specificati come parametro, da cui prelevano i dati da elaborare in alternativa alla lettura da stdin.

---

## 13. Concatenazione di File — cat e tac

**`cat`** è il più semplice dei filtri: invocato senza parametri, copia stdin su stdout; invocato con uno o più file come parametri, ne produce in sequenza il contenuto su stdout.

```bash
cat file1 file2
```

Tra le opzioni utili si segnala la numerazione delle righe e l'evidenziazione di tab e fine linea (consultare `man cat`).

**`tac`** riproduce le righe in ingresso (da stdin o da file) su stdout in ordine **inverso**, dall'ultima alla prima.

---

## 14. Impaginazione — less

`less` non è propriamente un filtro (il suo output è destinato al terminale), ma è tra i comandi più utili per l'uso interattivo. Posto al termine di una pipeline, intercetta l'output e lo mostra riempiendo lo spazio disponibile sul terminale, permettendo la navigazione tramite i seguenti comandi principali (per l'elenco completo: `man less`):

| Tasto | Azione |
|---|---|
| `h` | help dei comandi disponibili |
| frecce/pag-su/giù | movimento |
| `F` | Follow: scorre fino al termine dell'input e resta in attesa di nuove righe |
| `<N>g` | si porta alla riga numero N (default: 1) |
| `G` | si porta al termine del file |
| `/<pattern>` | cerca la riga successiva contenente `<pattern>` |
| `?<pattern>` | cerca la riga precedente contenente `<pattern>` |
| `n` | ripete la ricerca fatta in precedenza |
| `N` | ripete la ricerca nel verso opposto |
| `q` | esce da less |

---

## 15. Inversione dei Caratteri — rev

**`rev`** è un filtro che inverte l'ordine dei caratteri di ogni linea dello stream in input verso lo stream di output.

L'utilità del comando consiste tipicamente nell'accompagnare `cut` nell'estrazione di campi la cui posizione sia nota relativamente al fine linea:

```bash
cat /etc/passwd | rev | cut -f1 -d: -s | rev
```

Questa pipeline inverte ogni linea, prende il primo campo (che era l'ultimo dell'originale), e infine inverte nuovamente per ripristinare il campo selezionato.

---

## 16. Estrazione Verticale: head e tail

### head

`head` estrae la parte iniziale di un file (default: prime 10 righe).

- `-c NUM` — produce i primi NUM caratteri; con `-NUM` produce tutto il file eccetto gli ultimi NUM caratteri.
- `-n NUM` — produce le prime NUM righe; con `-NUM` produce tutto il file eccetto le ultime NUM righe.

### tail

`tail` estrae la parte finale di un file (default: ultime 10 righe).

- `-c NUM` — produce gli ultimi NUM caratteri; con `+NUM` produce tutto il file a partire dal carattere NUM.
- `-n NUM` — produce le ultime NUM righe; con `+NUM` produce tutto il file a partire dalla riga NUM.

### Opzioni particolari di tail

L'opzione `-f` (*follow*) è particolarmente importante: dopo aver mostrato le ultime righe, mantiene il file aperto e visualizza in tempo reale eventuali nuove righe che vi vengano appese da altri processi. `tail -f` è quindi uno strumento fondamentale per monitorare log in tempo reale.

- Con `-f` si può usare `--pid=PID` per far sì che, alla terminazione del processo PID, termini anche `tail`.
- Con `--retry`, `tail` continuerà a tentare l'apertura del file anche se non esiste ancora al momento dell'avvio.
- L'opzione `-F` equivale a `-f --retry`.

### Regola di efficienza

I comandi che partono dall'inizio (`head -n 20`, `tail -n +20`) sono velocissimi. Quelli che contano dal fondo (`tail -n 20`, `head -n -20`) sono inefficienti per file enormi, poiché costringono il sistema a leggere prima l'intero file.

---

## 17. Estrazione Orizzontale: cut

Il comando `cut` permette di tagliare parti di righe secondo due modalità operative.

### Modalità caratteri (-c)

```bash
cut -cELENCO_POSIZIONI_CARATTERI [input_file]
```

"Ritaglia" le righe producendo per ciascuna una riga in uscita composta dai soli caratteri elencati. Esempi:

```
cut -c15       # restituisce solo il 15° carattere
cut -c8-30     # restituisce i caratteri dall'8° al 30°
cut -c-30      # restituisce i caratteri fino al trentesimo
cut -c8-       # restituisce i caratteri dall'ottavo in poi
```

### Modalità campi (-d e -f)

Su file organizzati a "record" (uno per riga), in cui ogni record rappresenta una lista di campi opportunamente delimitati, le opzioni `-d` e `-f` consentono di estrarre uno o più campi di ciascun record:

```bash
cut -dCARATTERE_DELIMITATORE -fELENCO_CAMPI
```

L'opzione `-s` evita che vengano prodotte in output le righe che non contengono il delimitatore (che altrimenti sarebbero riprodotte per intero).

Esempio — estrarre solo il campo username dal file passwd:

```bash
cat /etc/passwd | cut -f1 -d: -s
```

Esempio — estrarre l'iniziale del cognome dal campo note (formato "Nome Cognome"):

```bash
cat /etc/passwd | cut -f5 -d: -s | cut -f2 -d' ' | cut -c1
```

**Limite intrinseco di cut:** è rigido con i delimitatori. Se si usa lo spazio come delimitatore (`-d' '`), la presenza di più spazi consecutivi (come nell'output di `ls -l`) genera campi vuoti. Per questi casi si ricorre ad `awk`.

---

## 18. Ordinamento e Unicità: sort e uniq

### sort

`sort` ordina le linee di uno stream in modo **lessicale** (per default). L'ordine dei caratteri dipende dal locale attivo: con `LC_ALL=C` corrisponde al valore dei byte; diversamente, segue l'ordine stabilito dal locale scelto.

**Opzioni di comportamento globale:**

- `-u` — elimina le entry multiple (equivale a `sort | uniq`).
- `-r` — reverse (ordinamento decrescente).
- `-R` — random (permutazione casuale delle righe).
- `-m` — merge di file già ordinati.
- `-c` — controlla se il file è già ordinato.

**Opzioni avanzate di ordinamento:**

- `-b` — ignora gli spazi a inizio riga.
- `-d` — considera solo i caratteri alfanumerici e gli spazi.
- `-f` — ignora la differenza minuscole/maiuscole.
- `-n` — interpreta le stringhe di numeri per il valore numerico (2 viene prima di 10).
- `-h` — interpreta i numeri "leggibili" come 2K, 1G, ecc.

**Ordinamento per chiave specifica:**

- `-tSEP` — imposta SEP come separatore tra campi (default: spazi).
- `-kKEY` — chiave di ordinamento nella forma `F[.C][,F[.C]][OPTS]`, dove F = numero di campo, C = posizione in caratteri nel campo, OPTS = una delle opzioni di ordinamento `[bdfgiMhnRrV]`. Se usato più volte, ordina per la prima chiave e, a parità, per la seconda, ecc.

Esempio — ordinare un elenco di indirizzi IP numericamente per ciascun ottetto:

```bash
sort -t. -k 1,1n -k 2,2n -k 3,3n -k 4,4n
```

### uniq

`uniq` elimina i duplicati **consecutivi** (per questo è quasi sempre preceduto da `sort`).

- `-c` — indica anche il numero di righe compattate in ciascuna.
- `-d` — mostra solo le entry non singole (i duplicati).

---

## 19. Conteggio: wc

**`wc`** (word count) è un filtro di conteggio:

- `-c` — conta i caratteri (byte).
- `-l` — conta le linee.
- `-w` — conta le parole (stringhe separate da spazi).

---

## 20. Ricerca di Pattern: grep ed Espressioni Regolari

### grep

`grep` esamina le righe del testo in ingresso (su stdin o su file specificati come argomento) e riproduce in uscita quelle che contengono un pattern corrispondente a un'**espressione regolare** (nel caso più semplice, una sottostringa).

Da qui in avanti si fa riferimento alla variante che supporta le espressioni regolari "moderne" (estese): **`egrep`** (equivalente a `grep -E`).

### Opzioni principali di grep

**Controllo del tipo di matching:**

- `-E` — usa le extended RE (come `egrep`).
- `-F` — disattiva le RE e usa il parametro come stringa letterale.
- `-w` / `-x` — fa match solo con RE "whole word" o "whole line".
- `-i` — rende l'espressione insensibile a maiuscole e minuscole.

**Controllo dell'input:**

- `-r` — cerca ricorsivamente in tutti i file di una cartella.
- `-f FILE` — prende le RE da un FILE invece che come parametro.

**Controllo dell'output:**

- `-o` — restituisce solo le sottostringhe che corrispondono alla RE (una per riga di output).
- `-v` — restituisce le linee che **non** contengono l'espressione.
- `-l` — restituisce solo i nomi dei file in cui l'espressione è stata trovata.
- `-n` — restituisce anche il numero della riga contenente l'espressione.
- `-c` — restituisce solo il conteggio delle righe che contengono la RE.
- `--line-buffered` — disattiva il buffering (utile in combinazione con `tail -f`).

### Espressioni Regolari Moderne (Estese)

La documentazione completa è reperibile nella man page `regex(7)`.

**Struttura formale:**

- **RE** = uno o più *rami* non vuoti separati da `|`.
- **ramo** = uno o più *pezzi* concatenati tra loro.
- **pezzo** = *atomo* eventualmente seguito da un *moltiplicatore*.
- **atomo** = uno di: `(RE)`, `[charset]`, `^`, `$`, `.`, backslash sequence, singolo carattere.

**Atomi speciali:**

- `.` — indica un qualsiasi carattere.
- `^` — indica l'inizio della linea.
- `$` — indica la fine della linea.

**Backslash sequence:**

- `\<` e `\>` — la stringa vuota all'inizio e alla fine di una parola.
- `\b` — la stringa vuota al confine di una parola.
- `\B` — la stringa vuota a condizione che *non* sia al confine di una parola.
- `\w` — sinonimo di "una qualsiasi lettera, numero o `_`".
- `\W` — sinonimo di "un qualsiasi carattere non compreso in `\w`".

**Moltiplicatori (quantificatori):**

- `{n,m}` — da n a m occorrenze dell'atomo che lo precede.
- `?` — zero o una occorrenza.
- `*` — zero o più occorrenze.
- `+` — una o più occorrenze.

**Charset — esempi:**

- `[abc]` — indica un qualsiasi carattere fra a, b o c.
- `[a-z]` — indica un qualsiasi carattere fra a e z compresi.
- `[^dc]` — indica un qualsiasi carattere che non sia né d né c.

**Charset basati su character class** — nella forma `[:NOME_CLASSE:]`, dove NOME_CLASSE appartiene all'insieme definito in `wctype(3)`: `alnum`, `digit`, `punct`, `alpha`, `graph`, `space`, `blank`, `lower`, `upper`, `cntrl`, `print`, `xdigit` (o eventualmente dal locale attivo).

### Regole di matching (Greediness)

Nel caso in cui una RE possa corrispondere a più di una sottostringa, la RE corrisponde a quella che inizia per prima nella stringa. Se a partire da quel punto la RE può corrispondere a più di una sottostringa, selezionerà la più lunga.

Nelle RE multilivello, le sottoespressioni selezionano sempre le sottostringhe più lunghe possibili, dando la priorità alle sottoespressioni che iniziano prima nella RE.

### Esempi di espressioni regolari

I caratteri speciali delle RE sono spesso anche caratteri speciali della shell: è buona norma racchiudere l'intera RE tra apici.

```bash
egrep '^Nel.*vita\.$' miofile
# righe che iniziano per "Nel" e finiscono per "vita."

egrep '.es[^es]{3,5}e' miofile
# righe contenenti: 1 carattere qualsiasi, "es", da 3 a 5 caratteri diversi da 'e' e 's', "e"
```

---

## 21. Modifiche Complesse: sed e tr

### sed (Stream EDitor)

`sed` è un editor di flusso capace di trasformare testo al volo. Il formato base è `sed -e 'comando'` o `sed -f 'script'`.

Nel corso ci si limita al comando di **sostituzione**:

```bash
sed 's/VECCHIO_PATTERN/NUOVO_VALORE/[modificatori]'
```

Sostituisce in ogni riga il NUOVO_VALORE alla parte di testo coincidente con VECCHIO_PATTERN. Con `sed -E` i pattern sono quelli delle espressioni regolari estese (come `egrep`).

**Modificatori del comando di sostituzione:**

- `i` — case insensitive.
- `g` — global (sostituisce tutte le occorrenze sulla riga, non solo la prima).
- `NUM` — sostituisce solo l'occorrenza NUM-esima.

**Opzioni sulla riga di comando:**

- `-i[SUFFIX]` — edita il file dato *in place* (con backup se si fornisce un SUFFIX).
- `-u` — unbuffered.

Esempio — inserire la stringa "Linea:" all'inizio di ogni riga di `/etc/passwd`:

```bash
cat /etc/passwd | sed 's/^/Linea:/'
```

### tr

Per sostituire più rapidamente singoli caratteri (senza regex), si utilizza **`tr`**:

```bash
tr 'A-Z' 'a-z'          # trasforma maiuscole in minuscole
tr ';:.!?' ','           # sostituisce ogni occorrenza dei caratteri del primo set con ','
tr -d '\r'               # elimina ogni occorrenza del carriage return
```

Se il secondo set è più limitato del primo, il suo ultimo carattere viene ripetuto quanto basta a generare la corrispondenza 1:1. Ad esempio `tr ';:.!?' ',-'` produce: `;` → `,`, `:` → `-`, `.` → `-`, `!` → `-`, `?` → `-`.

---

## 22. Elaborazione Avanzata dei Campi: awk

**`awk`** è un interprete per AWK (POSIX 1003.1), un linguaggio Turing-completo, definito *data-driven* in quanto pensato per applicare un algoritmo a ogni riga di testo fornita in ingresso.

Nel corso lo si utilizza primariamente come **evoluzione di `cut`**, poiché consente di considerare qualsiasi sequenza di caratteri come un unico delimitatore. Questo è il suo vantaggio principale rispetto a `cut`, che tratta ciascun carattere delimitatore singolarmente (generando campi vuoti in caso di delimitatori consecutivi, come gli spazi multipli nell'output di `ls -l`).

```bash
cat personale | awk '{print $2}'    # stampa il secondo campo
```

Per specificare un delimitatore diverso si usa l'opzione `-F`:

```bash
cat log | awk -F 'stat=' '{print $2}' | awk '{print $1}'
```

A differenza di `cut`, `awk` non ha il concetto di "-f 5-" (dal quinto campo in poi); l'equivalente è:

```bash
cat file | awk '{print substr($0, index($0,$5)) }'
```

Per approfondire: http://awklang.org/

---

## 23. Variazioni sul Tema Filtri

### xargs — Costruzione di linee di comando

Può essere necessario inserire in una pipeline comandi che non leggono stdin, ma vogliono parametri sulla riga di comando. `xargs <comando>` si aspetta sullo standard input un elenco di stringhe ed invoca `comando` con tali stringhe come argomenti.

```bash
pipeline | che_produce | nomi_di_file | xargs ls -l
```

**Particolarità:** xargs raggruppa le invocazioni per ridurre il carico. Inoltre, gli spazi nell'input vengono interpretati come separatori di parametri distinti.

**Opzioni utili:**

- `-0` (zero) — utilizza null, non lo spazio, come terminatore di argomento.
- `-L MAX` — usa al più MAX linee di input per ogni invocazione.
- `-p` — chiede interattivamente conferma del lancio di ogni comando.

### Process Substitution

Quando si ha un comando *producer* che produce dati su stdout e un comando *consumer* che vuole un file come parametro (e non accetta stdin), non si possono connettere con una pipe. La **process substitution** risolve il problema:

```bash
cmd_consumer <(cmd_producer_su_stdout)
```

Il processo tra parentesi è lanciato concorrentemente e la shell genera un nome di file (una named pipe) da fornire al primo. Il caso simmetrico è:

```bash
cmd_producer_su_file >(cmd_consumer_da_stdin)
```

### tee

**`tee`** è un comando utile per **duplicare** uno stream di output: invia una copia del proprio stdin a stdout, e una copia identica in un file passato come parametro. L'opzione `-a` apre il file in append.

```bash
comando1 | tee FILE | comando2
```

Combinato alla process substitution, permette elaborazioni ramificate complesse:

```bash
ls | tee >(grep foo | wc > foo.count) |
     tee >(grep bar | wc > bar.count) |
     grep baz | wc > baz.count
```

### Command Substitution

La **command substitution** permette di utilizzare direttamente un processo per generare parametri da collocare sulla command line per un altro comando. Due sintassi equivalenti:

```bash
`comando`
$(comando)
```

Il comando viene eseguito in una subshell e il suo stdout compare sulla riga di comando al posto del token di command substitution. La forma `$(...)` è preferita poiché più leggibile e annidabile.

```bash
ls $(cat /etc/passwd | cut -f6 -d:)
# estrae le home dir degli utenti e le pone come parametri a ls
```

---

## 24. Generazione Casuale: $RANDOM e shuf

**`$RANDOM`** è una variabile interna della shell (non un comando) che genera un numero intero casuale tra 0 e 32.767. Non è adatta per processare file più lunghi di circa 32.000 righe.

**`shuf`** è il filtro adatto per la generazione di permutazioni casuali: prende un flusso di testo e ne scombussola completamente l'ordine, indipendentemente da quante righe ci siano. Equivale funzionalmente a `sort -R`.

Opzioni aggiuntive:

- `-e arg1 arg2 ...` — genera permutazioni degli argomenti passati.
- `-i LO-HI` — genera permutazioni di un range di numeri.
- `-r` — ripete all'infinito estrazioni casuali dagli argomenti.

---

## 25. Strumenti Multi-File: diff, paste, join

### diff

Più frequentemente usato in modo interattivo, `diff` permette di mostrare le **differenze tra due file**. L'output indica righe cancellate (`d`), cambiate (`c`) e aggiunte (`a`), con il prefisso `<` per il file sinistro e `>` per il file destro.

```bash
diff filesinistro filedestro
```

Esempio di output:

```
2d1         # riga 2 del sinistro cancellata (deleted)
< riga due
4c3         # riga 4 del sinistro cambiata (changed) in riga 3 del destro
< riga quattro
---
> riga 4
5a5         # dopo riga 5 del sinistro, aggiunta (added) riga 5 del destro
> riga 5bis
```

### paste

In un certo senso l'opposto di `cut`: **unisce "orizzontalmente"** le righe di posizione omologa in vari file.

```bash
paste nazioni superfici abitanti
# Italia  301230  61261000
# Francia 547030  65630000
# ...
```

### join

Stesso principio di `paste`, ma anziché selezionare le righe sulla base della posizione, le unisce se iniziano con la stessa **"chiave"**. Necessita di file ordinati in modo identico sulla chiave selezionata.

```bash
join db_superfici db_abitanti
# Italia  301230  61261000
# Francia 547030  65630000
# ...
```

---

## 26. Laboratorio: Esercizi Svolti e Commentati

Di seguito la risoluzione degli esercizi proposti sulle slide, con la spiegazione logica.

### Esercizi 2 e 3: Estrazione con head e tail

**Obiettivo:** Estrarre dal file `/usr/share/dict/words` le righe dalla 30.000 alla 30.020.

**Approccio 1 — Rispetto all'inizio del file (metodo efficiente):**

```bash
tail -n +30000 /usr/share/dict/words | head -n 21
```

`tail` salta le prime 29.999 righe e stampa il resto; la pipe lo passa a `head`, che prende solo le prime 21 righe (dalla 30.000 alla 30.020) e chiude l'operazione senza leggere il resto del file.

**Approccio 2 — Rispetto alla fine del file (inefficiente e posizione diversa):**

```bash
tail -n 30020 /usr/share/dict/words | head -n 20
```

Usando i riferimenti dal basso si pescano parole completamente diverse.

**L'ottimizzazione del sistemista (hint wc -l):** per estrarre un blocco dal fondo in modo efficiente, si contano prima le righe totali con `wc -l`, si calcola lo scostamento dall'inizio, e si utilizza `head` + `tail` dall'alto.

### Esercizio 4: Estrazione di Colonne (awk)

**Obiettivo:** Estrarre dimensioni e nomi dei file da `ls -l /usr/bin`.

```bash
ls -l /usr/bin | awk '{print $5, $9}'
```

`cut -d' '` fallirebbe a causa degli spazi multipli usati per impaginare l'output di `ls -l`. `awk` risolve il problema alla radice considerando colonne fisse separate da "spazio vuoto".

### Esercizio 5: Ordinamento (sort vs ls)

**Obiettivo:** Ordinare i file di `/usr/bin` per dimensione.

```bash
# Metodo 1 — pipeline completa:
ls -l /usr/bin | sort -n -k5

# Metodo 2 — ottimizzazione nativa:
ls -lS /usr/bin           # dal più grande al più piccolo
ls -lSr /usr/bin          # dal più piccolo al più grande
```

### Esercizio 6: Regex e Shuf

**Task A — Estrarre indirizzi IP della macchina:**

```bash
ip a | grep -E -o '([0-9]{1,3}\.){3}[0-9]{1,3}'
```

`[0-9]{1,3}\.` cerca da 1 a 3 numeri seguiti da un punto; `(...)` raggruppa e `{3}` ripete 3 volte; `[0-9]{1,3}` chiude con gli ultimi numeri senza punto; `-o` estrae solo le sottostringhe matchate.

**Task B — Estrarre una parola casuale formata solo da minuscole, lunga da 4 a 7 caratteri:**

```bash
egrep '^[a-z]{4,7}$' /usr/share/dict/words | shuf | head -n 1
```

`$RANDOM` viene scartato perché, non superando 32.767, non potrebbe pescare le parole nella seconda metà del dizionario. `shuf` mescola l'intero output filtrato; `head -n 1` prende la prima carta dal mazzo.

---

## 27. Appendice: Gestione Dischi e Partizioni

*Note riassuntive sulle procedure di System Administration hardware.*

Per ridimensionare una partizione senza perdere dati, l'ordine logico delle operazioni è **Software → Hardware**:

1. **Smontaggio:** `sudo umount /dev/sdb1`
2. **Controllo integrità:** `sudo fsck -f /dev/sdb1` (obbligatorio prima del resize).
3. **Resize filesystem:** `sudo resize2fs /dev/sdb1 500M` (il software ora considera il disco come 500M).
4. **Resize hardware (fdisk):** Si entra in `fdisk /dev/sdb`, si cancella la partizione 1 e la si ricrea della nuova dimensione (es. `+500M`). *Attenzione cruciale:* deve iniziare dallo **stesso identico settore iniziale** (es. 2048) di prima. Alla domanda sulla rimozione della "signature ext4", rispondere rigorosamente **NO** per conservare i dati.
5. **Montaggio permanente:** Si modifica `/etc/fstab` aggiungendo ad esempio: `/dev/sdb2 /cartella_mount ext4 defaults 0 2`
