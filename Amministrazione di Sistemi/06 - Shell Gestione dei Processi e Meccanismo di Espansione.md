# Shell: Gestione dei Processi e Meccanismo di Espansione

Laboratorio di Amministrazione di Sistemi — Prof. Prandini

---

## Indice

1. [Principi dello Scripting con Bash](#1-principi-dello-scripting-con-bash)
2. [Fork ed Exec](#2-fork-ed-exec)
3. [Stream, Shell, Terminale e Avvio dei Programmi](#3-stream-shell-terminale-e-avvio-dei-programmi)
4. [Stream e Ridirezione](#4-stream-e-ridirezione)
5. [Ridirezioni Speciali e Permanenti](#5-ridirezioni-speciali-e-permanenti)
6. [La Pipe in Bash: Costruzione Interna](#6-la-pipe-in-bash-costruzione-interna)
7. [Pipe con Builtin e Funzioni](#7-pipe-con-builtin-e-funzioni)
8. [Subshell](#8-subshell)
9. [Interazione con i Processi: PID e Job ID](#9-interazione-con-i-processi-pid-e-job-id)
10. [Segnali](#10-segnali)
11. [Disposizione dei Segnali e trap](#11-disposizione-dei-segnali-e-trap)
12. [Invio di Segnali: kill e Scorciatoie da Terminale](#12-invio-di-segnali-kill-e-scorciatoie-da-terminale)
13. [sleep e Interazione con i Segnali](#13-sleep-e-interazione-con-i-segnali)
14. [Processi in Background: &, wait, jobs, fg](#14-processi-in-background--wait-jobs-fg)
15. [Modificatori per il Background: nohup, nice, disown](#15-modificatori-per-il-background-nohup-nice-disown)
16. [Il Meccanismo di Espansione della Shell](#16-il-meccanismo-di-espansione-della-shell)

---

## 1. Principi dello Scripting con Bash

Bash è al tempo stesso una shell interattiva e un interprete di script. La sua funzione principale è gestire due categorie di risorse del sistema operativo: **file** e **processi**. Tutta la potenza di bash deriva dalla capacità di orchestrare comandi (processi) e di collegare i loro flussi di I/O (file descriptor) attraverso i meccanismi di pipe e ridirezione.

Un aspetto fondamentale da interiorizzare è che bash è un interprete **espansivo**: prima di eseguire un comando, la riga digitata dall'utente viene sottoposta a una serie ordinata di trasformazioni (le *espansioni*). Comprendere questo meccanismo è indispensabile per prevedere il comportamento degli script e per evitare errori sottili.

---

## 2. Fork ed Exec

Ogni nuovo processo in Unix nasce attraverso la coppia di system call **`fork()`** e **`exec()`**.

### fork()

`fork()` **duplica** il processo chiamante (*processo padre*), creando un processo figlio (*child*) che eredita:

- una copia dell'**immagine di memoria** (stack, heap, variabili globali)
- la **tabella dei file descriptor** aperti (stdin, stdout, stderr e tutti gli altri)
- le **strutture dell'utente** (UID, GID, directory corrente, ecc.)

Padre e figlio condividono inizialmente il **segmento di testo** (codice), che viene duplicato solo se uno dei due lo modifica (meccanismo *copy-on-write*). Dopo `fork()`, entrambi i processi riprendono l'esecuzione dalla stessa istruzione, distinguendosi dal valore di ritorno: `0` nel figlio, PID del figlio nel padre.

### exec()

`exec()` **sostituisce** l'immagine del processo corrente con un nuovo programma: il codice, lo stack e l'heap del chiamante vengono rimpiazzati da quelli del nuovo eseguibile. I file descriptor aperti vengono conservati (a meno che non abbiano il flag `FD_CLOEXEC`). Il PID rimane invariato.

Il pattern tipico in bash è: `fork()` per creare un figlio, poi `exec()` nel figlio per eseguire il comando richiesto. Il padre attende la terminazione del figlio con `wait()`.

---

## 3. Stream, Shell, Terminale e Avvio dei Programmi

Quando un utente accede al sistema, si instaura una catena di processi collegati:

```
terminale (hardware o emulatore) → /dev/tty* o /dev/pts/* → bash → comando
```

Il kernel espone i dispositivi di I/O attraverso file speciali in `/dev`: `/dev/tty*` per i terminali fisici (console), `/dev/pts/*` per gli pseudoterminali (finestre di terminale, SSH). Quando bash lancia un comando, il figlio eredita i **file descriptor** 0 (stdin), 1 (stdout) e 2 (stderr) di bash, che a loro volta puntano al dispositivo terminale.

Questa catena di fork/exec è il motivo per cui i programmi "vedono" automaticamente il terminale come loro I/O standard: lo ereditano dal processo bash che li ha lanciati.

---

## 4. Stream e Ridirezione

I tre stream standard sono:

| File Descriptor | Nome | Descrizione |
|-----------------|------|-------------|
| `0` | stdin | Input standard |
| `1` | stdout | Output standard |
| `2` | stderr | Output di errore |

Le **ridirezioni** modificano il collegamento di questi file descriptor prima dell'esecuzione del comando:

| Sintassi | Effetto |
|----------|---------|
| `cmd > file` | Ridirezione stdout su `file` (sovrascrive) |
| `cmd >> file` | Ridirezione stdout su `file` (aggiunge in coda) |
| `cmd 2> file` | Ridirezione stderr su `file` (sovrascrive) |
| `cmd 2>> file` | Ridirezione stderr su `file` (aggiunge in coda) |
| `cmd 2>&1` | Ridirezione stderr sullo stesso fd di stdout |
| `cmd < file` | Ridirezione stdin da `file` |

L'ordine delle ridirezioni è significativo: `cmd > file 2>&1` ridireige prima stdout su `file`, poi stderr sullo stesso fd di stdout (quindi su `file`). L'ordine inverso `cmd 2>&1 > file` produce un effetto diverso.

---

## 5. Ridirezioni Speciali e Permanenti

### Here Document

Consente di fornire un blocco di testo multiriga come stdin di un comando, all'interno dello script:

```bash
cat <<MARKER
Prima riga
Seconda riga
MARKER
```

Il testo tra i due `MARKER` viene inviato a stdin di `cat`. Le espansioni di variabili sono attive all'interno del here document (a meno di non quotare il marker di apertura: `<<'MARKER'`).

### Here String

Versione compatta del here document per fornire una singola stringa come stdin:

```bash
cat <<< "testo da passare a stdin"
```

### Ridirezioni Permanenti con exec

`exec` senza un comando da eseguire modifica i file descriptor della shell corrente in modo permanente per il resto dello script:

```bash
exec 2>/dev/null            # da qui in poi, stderr va in /dev/null
exec 3< filein              # fd 3 aperto in lettura su filein
exec 4> fileout             # fd 4 aperto in scrittura su fileout
exec 5<> filerw             # fd 5 aperto in lettura e scrittura
```

Questo meccanismo è utile per redirigere stderr dell'intero script o per aprire file descriptor aggiuntivi per comunicazioni tra parti dello script.

---

## 6. La Pipe in Bash: Costruzione Interna

La pipe (`|`) è uno dei meccanismi più potenti di Unix: consente di collegare lo stdout di un processo allo stdin del successivo. La sua costruzione interna in bash segue esattamente **6 passi**:

**Esempio**: `ls | sort`

1. **`pipe(fd[])`** — bash chiama la system call `pipe()`, che crea un canale unidirezionale nel kernel e restituisce due file descriptor: `fd[0]` (estremità di lettura) e `fd[1]` (estremità di scrittura).

2. **`fork()` × 2** — bash si duplica due volte, creando due processi figli: `bash_c1` (per `ls`) e `bash_c2` (per `sort`).

3. **`dup2(fd[1], 1)` nel figlio 1** — `bash_c1` sostituisce il proprio stdout (fd 1) con l'estremità di scrittura della pipe (`fd[1]`). Da questo momento, tutto ciò che `ls` scriverà su stdout finirà nella pipe.

4. **`dup2(fd[0], 0)` nel figlio 2** — `bash_c2` sostituisce il proprio stdin (fd 0) con l'estremità di lettura della pipe (`fd[0]`). Da questo momento, `sort` leggerà da stdin ciò che arriva dalla pipe.

5. **Chiusura di `fd[0]` e `fd[1]`** — entrambi i figli chiudono le estremità originali (non duplicate) della pipe, in modo che il canale si chiuda correttamente quando il produttore termina.

6. **`exec("ls")` e `exec("sort")`** — i figli eseguono i comandi richiesti. Quando `ls` termina, l'estremità di scrittura della pipe si chiude; `sort` riceve EOF su stdin e può completare l'ordinamento.

---

## 7. Pipe con Builtin e Funzioni

Quando uno dei comandi in una pipeline è un **builtin** bash o una **funzione**, non può essere eseguito con `exec()` (che sostituisce il processo). In questo caso il passo 6 è diverso:

- il figlio corrispondente al builtin/funzione **non chiama `exec()`**, ma esegue direttamente il codice del builtin nella propria copia di bash (che diventa una *subshell*).

Questa distinzione ha importanti conseguenze: le modifiche a variabili effettuate all'interno di un builtin in pipeline (ad esempio `read` in `... | read VAR`) avvengono in una subshell e non sono visibili nella shell genitrice.

---

## 8. Subshell

Una **subshell** è un processo figlio di bash che esegue un gruppo di comandi in un ambiente separato. Si crea con la sintassi:

```bash
( cmd1 ; cmd2 ; cmd3 )
```

La subshell **eredita** tutto l'ambiente della shell genitrice (variabili, file descriptor aperti, directory corrente), ma ogni modifica effettuata al suo interno (assegnazione di variabili, cambio di directory, ridirezioni permanenti) è **locale** alla subshell e non si propaga alla genitrice.

Le subshell sono utili per isolare effetti collaterali:

```bash
# La directory corrente della shell genitrice non cambia
(cd /tmp && ls)
echo "Sono ancora in: $PWD"
```

---

## 9. Interazione con i Processi: PID e Job ID

Ogni processo nel sistema è identificato da un **PID** (*Process ID*), un numero intero assegnato dal kernel al momento della creazione. I processi in esecuzione sotto la stessa sessione bash sono organizzati in **job**, identificati da un **Job ID** progressivo.

I processi vengono eseguiti con i **privilegi dell'utente** che ha lanciato la shell, il che determina i file accessibili e le operazioni permesse.

La variabile speciale `$!` contiene il PID dell'ultimo processo lanciato in background con `&`.

---

## 10. Segnali

I **segnali** sono eventi asincroni che il kernel invia ai processi per notificare condizioni particolari (errori, richieste di terminazione, interrupt da tastiera, ecc.). Ogni segnale è identificato da un numero intero e da un nome simbolico.

Caratteristiche fondamentali:

- I segnali sono **asincroni**: possono arrivare in qualunque momento, indipendentemente da ciò che il processo sta facendo.
- Un segnale può trovarsi in stato **pending** (consegnato ma non ancora gestito) mentre il processo è occupato in una system call bloccante.
- Ciascun processo può registrare un **handler** (gestore) per ogni segnale, ovvero una funzione da eseguire alla ricezione del segnale.

---

## 11. Disposizione dei Segnali e trap

La risposta di un processo a un segnale è detta **disposizione** (*disposition*). Le disposizioni possibili sono:

| Disposizione | Descrizione |
|--------------|-------------|
| **terminate** | Il processo termina (comportamento predefinito per molti segnali). |
| **ignore** | Il segnale viene ignorato. |
| **stop** | Il processo viene sospeso (fermato). |
| **cont** | Il processo sospeso riprende l'esecuzione. |

Due segnali sono **non intercettabili e non ignorabili**:
- `SIGKILL` (9): termina il processo incondizionatamente.
- `SIGSTOP` (19/17): sospende il processo incondizionatamente.

### Il builtin trap

In bash, il builtin **`trap`** consente di registrare handler per i segnali:

```bash
trap [-lp] [[codice] segnale...]
```

- `trap 'echo "Intercettato SIGINT"' INT` — registra un handler per SIGINT.
- `trap '' INT` — ignora SIGINT.
- `trap - INT` — ripristina la disposizione predefinita.
- `trap -l` — elenca i segnali disponibili.
- `trap -p` — mostra gli handler correntemente registrati.

Bash definisce anche **pseudo-segnali** che non corrispondono a segnali reali del kernel, ma possono essere intercettati con `trap`:

| Pseudo-segnale | Quando viene attivato |
|----------------|----------------------|
| `DEBUG` | Prima di ogni comando. |
| `RETURN` | Al ritorno da una funzione o da uno script sorgente. |
| `ERR` | Ogni volta che un comando restituisce exit code ≠ 0. |
| `EXIT` | All'uscita dalla shell (normale o per segnale). |

> **Nota**: gli handler registrati con `trap` nella shell genitrice **non vengono ereditati** dai processi figli.

---

## 12. Invio di Segnali: kill e Scorciatoie da Terminale

### Il comando kill

```bash
kill [opzioni] <pid> [...]
```

Nonostante il nome, `kill` non uccide necessariamente un processo: invia un segnale qualsiasi al PID specificato. Per impostazione predefinita invia `SIGTERM` (15), che può essere intercettato e ignorato. Le opzioni principali sono:

- `kill -9 <pid>` — invia SIGKILL (non intercettabile).
- `kill -l` — elenca i segnali disponibili.
- `kill -s SIGNAME <pid>` — invia il segnale specificato per nome.

### Scorciatoie da terminale

Il terminale intercetta alcune combinazioni di tasti e le converte in segnali inviati al processo in foreground:

| Combinazione | Segnale | Effetto predefinito |
|--------------|---------|---------------------|
| `Ctrl+C` | `SIGINT` | Interruzione |
| `Ctrl+\` | `SIGQUIT` | Terminazione con core dump |
| `Ctrl+Z` | `SIGTSTP` | Sospensione (stop) |
| `Ctrl+D` | (EOF) | Chiusura stdin |
| `Ctrl+Q` | — | Riprende output terminale (XON) |
| `Ctrl+S` | — | Sospende output terminale (XOFF) |

---

## 13. sleep e Interazione con i Segnali

Il comando **`sleep`** è un processo esterno (non un builtin) che sospende l'esecuzione per un intervallo di tempo. Supporta i suffissi:

```bash
sleep 10      # 10 secondi
sleep 2m      # 2 minuti
sleep 1h      # 1 ora
sleep 0.5d    # 0.5 giorni
```

Poiché `sleep` è un processo esterno, **risponde ai segnali**: un `SIGINT` (Ctrl+C) lo interrompe immediatamente. Questo comportamento è rilevante negli script che utilizzano `sleep` come timer: se il processo bash genitore intercetta un segnale mentre è in `wait`, il comportamento dipende dalla registrazione degli handler.

---

## 14. Processi in Background: &, wait, jobs, fg

### Avvio in background con &

Aggiungendo `&` al termine di un comando, bash lo avvia in background senza attenderne il completamento:

```bash
comando &
# Output: [1] 12345
```

Il numero tra parentesi quadre è il **Job ID**, il numero successivo è il **PID**. Il PID viene anche memorizzato nella variabile speciale `$!`.

Per sospendere un processo in foreground e inviarlo in background: `Ctrl+Z` (invia SIGTSTP, sospende il processo), poi `bg %job_id` per riprenderne l'esecuzione in background.

### wait

Il builtin **`wait`** blocca la shell corrente fino al completamento dei job in background specificati:

```bash
wait [pid_o_job_id...]
```

Senza argomenti, attende tutti i job in background. `wait` risponde ai segnali durante l'attesa.

### jobs e fg

- **`jobs`**: elenca i job attivi con il loro stato (Running, Stopped, Done).
- **`fg %job_id`**: riporta in foreground il job specificato.

```bash
# Esempio
sleep 100 &    # avvia in background (Job 1)
sleep 200 &    # avvia in background (Job 2)
jobs           # mostra: [1] Running, [2] Running
fg %1          # riporta "sleep 100" in foreground
```

---

## 15. Modificatori per il Background: nohup, nice, disown

Questi tre comandi/builtin modificano il comportamento di processi in background in modo complementare:

### nohup

Esegue un comando **immune a SIGHUP** (il segnale inviato ai processi quando la sessione termina). Utile per processi che devono sopravvivere alla disconnessione dell'utente:

```bash
nohup comando [argomenti] &
```

`nohup` ridireige automaticamente stdout e stderr su `nohup.out` (nella directory corrente) se non già ridirezionati esplicitamente.

### nice

Modifica la **priorità di scheduling** del processo (il valore di *niceness*). Valori più alti di niceness significano priorità più bassa (il processo è "gentile" verso gli altri):

```bash
nice -n 10 comando    # avvia con niceness +10 (meno prioritario)
```

Il valore predefinito di `nice` è 10. Solo root può impostare valori di niceness negativi (maggiore priorità).

### disown

Rimuove un job dalla **tabella dei job** di bash, in modo che bash non gli invii SIGHUP alla chiusura della sessione:

```bash
disown %job_id
```

A differenza di `nohup` (che si usa prima di avviare il processo), `disown` può essere applicato a un processo già in esecuzione.

---

## 16. Il Meccanismo di Espansione della Shell

Prima di eseguire qualsiasi comando, bash sottopone la riga di input a una sequenza ordinata di **trasformazioni**. Comprendere quest'ordine è fondamentale per scrivere script corretti.

### Ordine di elaborazione della riga di comando

Bash processa la riga in questo ordine:

1. **Accantonamento delle assegnazioni** — le assegnazioni `NOME=valore` all'inizio della riga vengono messe da parte temporaneamente.
2. **Espansione degli elementi** — gli elementi non-assegnazione vengono espansi secondo i passi descritti sotto.
3. **Impostazione delle ridirezioni** — le ridirezioni vengono applicate dopo le espansioni.
4. **Ripristino delle assegnazioni** — le assegnazioni accantonate vengono applicate all'ambiente del processo figlio.

### I passi dell'espansione

**Passo 1 — Tokenizzazione**: la riga viene divisa in *token* usando i metacaratteri `SPACE TAB NEWLINE ; ( ) < > | &`.

**Passo 2 — Espansione degli alias**: se il primo token corrisponde a un alias definito, viene sostituito e la tokenizzazione riparte dall'inizio.

**Passo 3 — Riconoscimento delle keyword**: se il primo token è una keyword bash (`if`, `while`, `function`, `{`, `(`, …), viene attivato il parsing strutturato corrispondente.

**Passo 4 — Brace expansion**: le espressioni tra graffe vengono espanse. Due forme:
- **Lista**: `Pre{a,b,c}Post` → `Prea Preb Prec`
- **Sequenza**: `{1..5}` → `1 2 3 4 5`

Questa espansione avviene *prima* delle altre e non dipende dal filesystem.

**Passo 5 — Tilde expansion**: `~` viene espanso nella home directory dell'utente corrente; `~username` nella home di `username`.

**Passo 6 — Parameter expansion**: `$NOME` o `${NOME}` viene sostituito con il valore della variabile.

**Passo 7 — Command substitution**: `$(comando)` viene sostituito con l'output del comando eseguito (rimuovendo il newline finale).

**Passo 8 — Arithmetic substitution**: `$((espressione))` viene valutato aritmeticamente e sostituito con il risultato.

**Passo 9 — Process substitution**: `<(comando)` o `>(comando)` sostituisce il comando con un file descriptor (supportato solo in bash, non in sh).

**Passo 10 — Word splitting**: il risultato delle espansioni 6–8 viene suddiviso in parole separate usando i caratteri in `$IFS` (per impostazione predefinita: spazio, tab, newline).

**Passo 11 — Pathname expansion** (globbing): i pattern contenenti `*`, `?` o `[SET]` vengono espansi nella lista dei file corrispondenti (in ordine alfabetico). Se nessun file corrisponde, il pattern rimane letterale.

**Passo 12 — Command lookup**: il primo token risultante viene cercato come funzione, builtin o comando esterno nel `PATH`.

### Quoting: inibire l'espansione

I meccanismi di **quoting** permettono di proteggere i caratteri speciali dall'espansione:

| Meccanismo | Effetto |
|------------|---------|
| `\c` | Protegge il singolo carattere `c` |
| `'testo'` | Protegge tutto il contenuto letteralmente (nessuna eccezione) |
| `"testo"` | Protegge tutto tranne `$`, `` ` `` e `\` |

Le sequenze protette possono essere concatenate: `"valore=$VAR"'\*'` produce la stringa `valore=<contenuto di VAR>` seguita da `\*` letterale.

---
