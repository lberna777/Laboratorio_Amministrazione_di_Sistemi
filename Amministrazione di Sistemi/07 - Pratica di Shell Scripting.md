# Pratica di Shell Scripting

Laboratorio di Amministrazione di Sistemi — Prof. Prandini

---

## Indice

1. [Quoting: Protezione dei Caratteri Speciali](#1-quoting-protezione-dei-caratteri-speciali)
2. [Il Comando echo](#2-il-comando-echo)
3. [Pathname Expansion (Globbing)](#3-pathname-expansion-globbing)
4. [Brace Expansion](#4-brace-expansion)
5. [Variabili](#5-variabili)
6. [Variabili di Ambiente](#6-variabili-di-ambiente)
7. [Variabili Notevoli di Bash](#7-variabili-notevoli-di-bash)
8. [Variabili Posizionali](#8-variabili-posizionali)
9. [Parameter Expansion: Assegnazione Condizionale](#9-parameter-expansion-assegnazione-condizionale)
10. [Parameter Expansion: Manipolazione delle Stringhe](#10-parameter-expansion-manipolazione-delle-stringhe)
11. [Accesso Indiretto alle Variabili](#11-accesso-indiretto-alle-variabili)
12. [Array Numerici](#12-array-numerici)
13. [Array Associativi](#13-array-associativi)
14. [Il Builtin read](#14-il-builtin-read)
15. [Aritmetica in Bash](#15-aritmetica-in-bash)
16. [Strutture di Controllo: Sequenze e Funzioni](#16-strutture-di-controllo-sequenze-e-funzioni)
17. [Valutazione delle Condizioni](#17-valutazione-delle-condizioni)
18. [Istruzione if](#18-istruzione-if)
19. [Istruzione case](#19-istruzione-case)
20. [Cicli for](#20-cicli-for)

---

## 1. Quoting: Protezione dei Caratteri Speciali

Bash attribuisce un significato speciale a numerosi caratteri che, se non protetti, vengono interpretati dalla shell prima di essere passati al comando:

```
[ ] ! * ? $ { } ( ) " ' \ | > < ;
```

Per trattare questi caratteri come letterali, bash offre tre meccanismi di quoting:

| Meccanismo | Sintassi | Comportamento |
|------------|----------|---------------|
| **Backslash** | `\c` | Protegge il singolo carattere `c`. |
| **Apici singoli** | `'testo'` | Protegge tutto il contenuto letteralmente, senza eccezioni. Non è possibile includere un apice singolo all'interno. |
| **Virgolette doppie** | `"testo"` | Protegge tutto il contenuto, tranne `$`, `` ` `` (backtick) e `\`. Le espansioni di variabile e di comando rimangono attive. |

È possibile **concatenare** frammenti protetti in modi diversi: il risultato è una singola stringa. Ad esempio:

```bash
echo "protetto da virgolette"\*'o da apici'
# Output: protetto da virgolette*o da apici
```

Il carattere `*` è letterale (protetto dal backslash), mentre `$` all'interno delle virgolette doppie rimane attivo.

---

## 2. Il Comando echo

Il comportamento di `echo` illustra bene come la shell espande i suoi argomenti prima di passarli al comando:

```bash
echo PATH        # stampa la stringa letterale "PATH"
echo $PATH       # stampa il contenuto della variabile PATH
echo *           # stampa i nomi di tutti i file nella directory corrente
```

Nel terzo caso, `*` viene espanso dalla shell (pathname expansion) prima che `echo` venga invocato: il comando riceve già la lista dei file come argomenti separati.

---

## 3. Pathname Expansion (Globbing)

La **pathname expansion** (o *globbing*) espande i pattern in elenchi di file corrispondenti nel filesystem. Il confronto avviene con i nomi dei file nella directory corrente (o nel percorso specificato). I file vengono restituiti in ordine **alfabetico**.

Se nessun file corrisponde al pattern, il pattern rimane **letterale** (non viene espanso).

### Pattern disponibili

| Pattern | Significato |
|---------|-------------|
| `*` | Qualsiasi sequenza di caratteri (anche vuota), escluso `/`. |
| `?` | Esattamente un carattere qualsiasi. |
| `[SET]` | Esattamente un carattere nell'insieme specificato. |

### Varianti di `[SET]`

- `[abc]` — uno tra `a`, `b`, `c`.
- `[a-z]` — un carattere nell'intervallo da `a` a `z`.
- `[!abc]` o `[^abc]` — un carattere **non** in `{a, b, c}`.
- `[[:classe:]]` — una classe POSIX: `[:alpha:]`, `[:digit:]`, `[:alnum:]`, `[:space:]`, `[:upper:]`, `[:lower:]`, ecc.

### Esempi

```bash
echo *                    # tutti i file nella directory corrente
echo [a-p,1-7]*[cfd]?     # file che iniziano con a-p o 1-7, finiscono con c/f/d + 1 char
echo \*                   # stampa l'asterisco letterale
echo *[!\*\?]*            # file che contengono almeno un carattere che non è * né ?
echo /*/*/*               # file a profondità 3 dal root
```

---

## 4. Brace Expansion

La **brace expansion** genera stringhe a partire da un pattern con graffe, **senza consultare il filesystem**. Avviene prima della pathname expansion e può essere usata per generare sequenze arbitrarie di stringhe.

### Forma a lista

```bash
Pre{elem1,elem2,elem3}Post
```

```bash
echo a{b,c,d}e        # abe ace ade
mkdir progetto/{src,include,build,doc}
```

### Forma a sequenza

```bash
{inizio..fine[..incremento]}
```

```bash
echo {1..5}           # 1 2 3 4 5
echo {a..e}           # a b c d e
echo {1..10..2}       # 1 3 5 7 9
echo {01..05}         # 01 02 03 04 05  (zero-padding automatico)
```

### Forme avanzate

```bash
# Prodotto cartesiano
echo {a,b}{1,2}       # a1 a2 b1 b2

# Espansioni annidate
echo {a{1,2},b{3,4}}  # a1 a2 b3 b4
```

---

## 5. Variabili

In bash, le variabili sono **stringhe** per impostazione predefinita. L'assegnazione non ammette spazi attorno al segno `=`:

```bash
pippo=valore          # corretto
pippo = valore        # ERRORE: bash interpreta "pippo" come un comando
```

Per accedere al valore di una variabile si usa `$NOME` o la forma `${NOME}` (necessaria per disambiguare i confini del nome):

```bash
echo $pippo           # stampa il valore
echo ${pippo}_suffisso  # obbligatorio per aggiungere testo dopo il nome

pippo="primo valore"
echo "$pippo"         # buona pratica: virgolette per gestire spazi
```

### Interazione con il quoting

```bash
x="ciao mondo"
echo $x               # ciao mondo (word splitting: 2 argomenti)
echo "$x"             # ciao mondo (1 argomento, spazio preservato)
```

---

## 6. Variabili di Ambiente

Le **variabili di ambiente** sono variabili che vengono passate ai processi figli. In bash, una variabile viene esportata nell'ambiente con il comando `export`:

```bash
export pippo          # esporta la variabile pippo già definita
export pippo=valore   # definisce ed esporta in un passo solo
```

Per visualizzare l'ambiente corrente:

```bash
set                   # mostra tutte le variabili (ambiente + locali shell)
env                   # mostra solo le variabili di ambiente
```

È possibile passare variabili di ambiente **solo per la durata di un singolo comando** anteponendo l'assegnazione al comando:

```bash
LANG=C ls --help      # ls usa LANG=C, ma la shell non modifica il proprio LANG
```

---

## 7. Variabili Notevoli di Bash

Bash definisce un insieme di variabili speciali utili negli script:

| Variabile | Descrizione |
|-----------|-------------|
| `BASHPID` | PID del processo bash corrente. |
| `$$` | PID del processo *capostipite* (utile negli script per nomi di file temporanei univoci). |
| `PPID` | PID del processo genitore. |
| `HOSTNAME` | Nome dell'host corrente. |
| `RANDOM` | Numero pseudo-casuale tra 0 e 32767 (si rigenera ad ogni accesso). |
| `UID` | User ID dell'utente corrente. |
| `HOME` | Directory home dell'utente corrente. |
| `LC_*` | Variabili di localizzazione (lingua, formato numeri, ecc.). |
| `PS0` | Prompt mostrato prima di ogni comando (bash 4.4+). |
| `PS1` | Prompt primario (quello mostrato quando bash è pronto per input). |
| `PS2` | Prompt secondario (per comandi su più righe). |
| `PS4` | Prompt usato con `set -x` (debug). |

---

## 8. Variabili Posizionali

Le **variabili posizionali** contengono gli argomenti passati allo script (o alla funzione):

| Variabile | Contenuto |
|-----------|-----------|
| `$0` | Nome dello script (o della shell). |
| `$1`, `$2`, … | Primo, secondo, … argomento. |
| `$*` | Tutti gli argomenti come **un'unica stringa** (separati dal primo carattere di `$IFS`). |
| `$@` | Tutti gli argomenti come **parole separate**. |
| `"$*"` | Tutti gli argomenti come **un'unica stringa quotata**. |
| `"$@"` | Tutti gli argomenti come **stringhe separate e quotate** individualmente (da preferire). |
| `$#` | Numero di argomenti. |

### Il comando shift

```bash
shift [n]
```

`shift` scarta i primi `n` argomenti posizionali (default: 1), spostando quelli restanti: `$2` diventa `$1`, `$3` diventa `$2`, ecc. Utile per processare argomenti in un ciclo.

---

## 9. Parameter Expansion: Assegnazione Condizionale

Bash offre forme speciali di parameter expansion per gestire variabili non definite o vuote:

| Sintassi | Comportamento |
|----------|---------------|
| `${VAR:-default}` | Se `VAR` non è definita o è vuota, restituisce `default` (senza modificare `VAR`). |
| `${VAR:=default}` | Se `VAR` non è definita o è vuota, assegna `default` a `VAR` e lo restituisce. |
| `${VAR:?messaggio}` | Se `VAR` non è definita o è vuota, stampa `messaggio` su stderr e termina lo script. |
| `${VAR:+alternativa}` | Se `VAR` è definita e non vuota, restituisce `alternativa`; altrimenti restituisce stringa vuota. |

### Esempi pratici

```bash
# Usa "/tmp" se il primo argomento non è fornito
DIR=${1:-"/tmp"}

# Assegna un valore di default alla home se non impostata
WORKDIR=${HOME:=/tmp}

# Termina lo script se il secondo argomento manca
FILE=${2:?"Errore: specificare un nome file"}
```

---

## 10. Parameter Expansion: Manipolazione delle Stringhe

Bash permette di manipolare il contenuto delle variabili direttamente nell'espansione, senza invocare comandi esterni come `sed` o `cut`:

| Sintassi | Operazione |
|----------|-----------|
| `${#nome}` | Lunghezza della stringa. |
| `${nome:n}` | Sottostringa dall'indice `n` alla fine. |
| `${nome:n:l}` | Sottostringa dall'indice `n` di lunghezza `l`. |
| `${nome#pattern}` | Rimuove la corrispondenza **più corta** dal **fronte** (prefisso). |
| `${nome##pattern}` | Rimuove la corrispondenza **più lunga** dal **fronte** (prefisso). |
| `${nome%pattern}` | Rimuove la corrispondenza **più corta** dalla **coda** (suffisso). |
| `${nome%%pattern}` | Rimuove la corrispondenza **più lunga** dalla **coda** (suffisso). |
| `${nome/pattern/stringa}` | Sostituisce la **prima** occorrenza di `pattern` con `stringa`. |
| `${nome//pattern/stringa}` | Sostituisce **tutte** le occorrenze di `pattern` con `stringa`. |

### Esempio pratico: rinomina file

```bash
FN="documento.bad"
mv "${FN}" "${FN/.bad/.good}"
# Sposta "documento.bad" in "documento.good"
```

### Altri esempi

```bash
stringa="ciao mondo"
echo ${#stringa}          # 10
echo ${stringa:5}         # mondo
echo ${stringa:0:4}       # ciao

percorso="/usr/local/bin/bash"
echo ${percorso##*/}      # bash  (basename)
echo ${percorso%/*}       # /usr/local/bin  (dirname)
```

---

## 11. Accesso Indiretto alle Variabili

Bash supporta l'**accesso indiretto** a una variabile tramite la sintassi `${!CHIAVE}`: il valore di `CHIAVE` viene interpretato come il nome di un'altra variabile, e viene restituito il valore di quest'ultima.

```bash
pippo="ciao"
chiave="pippo"
echo ${!chiave}    # stampa "ciao" (valore di $pippo)
```

Questo meccanismo è utile per costruire nomi di variabili dinamicamente all'interno degli script.

---

## 12. Array Numerici

Bash supporta array monodimensionali con indici numerici interi (anche **sparsi**):

### Creazione

```bash
declare -a A                  # dichiarazione esplicita (facoltativa)
A[0]="primo"                  # assegnazione per indice
A[1]="secondo"
MYVECTOR=("alfa" "beta" "gamma")  # assegnazione in blocco
```

### Accesso

```bash
echo ${A[0]}                  # accesso all'elemento 0
echo ${MYVECTOR[2]}           # elemento di indice 2: "gamma"
echo ${A[*]}                  # tutti gli elementi (come unica stringa)
echo ${A[@]}                  # tutti gli elementi (come parole separate)
echo "${A[@]}"                # forma raccomandata per preservare gli spazi negli elementi
```

### Metadati

```bash
echo ${!A[@]}                 # lista degli indici definiti (utile per array sparsi)
echo ${#A[*]}                 # numero di elementi
```

### Inizializzazione con IFS

```bash
IFS=: read -ra PARTI <<< "/usr/local/bin"
# PARTI[0]="" PARTI[1]="usr" PARTI[2]="local" PARTI[3]="bin"
```

---

## 13. Array Associativi

Gli **array associativi** (disponibili da bash 4) usano stringhe come indici:

```bash
declare -A rubrica              # la dichiarazione è obbligatoria

rubrica["mario"]="0512345678"
rubrica["luigi"]="0519876543"

echo ${rubrica["mario"]}        # 0512345678
echo ${!rubrica[@]}             # mario luigi  (lista delle chiavi)
echo ${#rubrica[@]}             # 2  (numero di coppie)
```

---

## 14. Il Builtin read

Il builtin **`read`** legge una riga da stdin (o da un file descriptor specificato), la tokenizza usando `$IFS` e assegna i token alle variabili specificate:

```bash
read [opzioni] [nome...]
```

| Opzione | Effetto |
|---------|---------|
| `-p "prompt"` | Mostra `prompt` prima di leggere. |
| `-u fd` | Legge dal file descriptor `fd` invece di stdin. |
| `-a array` | Tokenizza la riga e assegna i token a un array. |
| `-r` | Modalità raw: il backslash non ha significato speciale. |

### Il problema di read nelle pipeline

Poiché ogni comando in una pipeline viene eseguito in una **subshell**, le variabili assegnate con `read` non sono visibili nella shell genitrice:

```bash
echo "ciao mondo" | read A B
echo $A   # vuoto! A era nella subshell
```

**Soluzione 1**: usare una subshell esplicita e accedere alle variabili all'interno:

```bash
echo "ciao mondo" | ( read A B; echo "A=$A B=$B" )
```

**Soluzione 2**: aprire un fd aggiuntivo connesso al terminale e usare `-u`:

```bash
exec 3<$(tty)
echo "ciao mondo" | { read A B; read -u 3 C; echo "A=$A C=$C"; }
```

---

## 15. Aritmetica in Bash

Bash tratta le variabili come **stringhe** per impostazione predefinita. Per effettuare calcoli aritmetici è necessario utilizzare meccanismi specifici:

### declare -i

Dichiara una variabile come intera: ogni assegnazione viene valutata aritmeticamente.

```bash
declare -i n
n=3+4       # n vale 7 (non la stringa "3+4")
```

### let

```bash
let "n = 3 + 4"
let n++
```

### (( )) — valutazione aritmetica

La forma preferita per condizioni e assegnazioni:

```bash
(( n = 3 + 4 ))    # assegnazione
(( n++ ))          # incremento
(( a > b ))        # confronto: exit code 0 se vero, 1 se falso
```

All'interno di `(( ))`, le variabili sono riconosciute **per nome** senza il prefisso `$` (anche se `$` è accettato).

> **Attenzione**: `(( 0 ))` restituisce exit code 1 (falso), `(( 1 ))` restituisce 0 (vero). Questo comportamento è l'opposto della convenzione degli exit code Unix.

### $(( )) — sostituzione aritmetica

Restituisce il risultato come stringa:

```bash
echo $(( 3 + 4 ))       # 7
n=$(( n * 2 ))
```

### Operatori aritmetici

Bash supporta un insieme di operatori simile al C:

| Categoria | Operatori |
|-----------|-----------|
| Incremento/decremento | `++`, `--` (pre e post) |
| Aritmetici | `+`, `-`, `*`, `/`, `**` (potenza), `%` |
| Logici | `!`, `&&`, `\|\|`, operatore ternario `?:` |
| Bitwise | `~`, `&`, `\|`, `^`, `<<`, `>>` |
| Assegnazione | `=`, `*=`, `/=`, `%=`, `+=`, `-=`, `<<=`, `>>=`, `&=`, `^=`, `\|=` |
| Confronto | `<`, `>`, `<=`, `>=`, `==`, `!=` |

### Basi numeriche

```bash
echo $(( 16#ff ))     # 255 (esadecimale)
echo $(( 2#1010 ))    # 10  (binario)
echo $(( 8#17 ))      # 15  (ottale)
```

---

## 16. Strutture di Controllo: Sequenze e Funzioni

### Sequenze

In bash esistono due modi per raggruppare comandi in una sequenza:

```bash
# Subshell: ambiente separato
( cmd1 ; cmd2 ; cmd3 )

# Stesso ambiente della shell corrente
{ cmd1 ; cmd2 ; cmd3 ; }   # nota: spazio dopo { e ; o newline prima di }
```

La differenza fondamentale: le modifiche a variabili e directory effettuate dentro `{ }` sono visibili all'esterno; quelle in `( )` no.

### Funzioni

Le funzioni in bash si definiscono con:

```bash
function NOME() {
    SEQUENZA
}

# Oppure (sintassi POSIX):
NOME() {
    SEQUENZA
}
```

Le funzioni:

- Ricevono i propri argomenti come **variabili posizionali** (`$1`, `$2`, …), che oscurano temporaneamente quelle dello script chiamante.
- Hanno accesso a `$FUNCNAME` (nome della funzione corrente).
- Eseguono nel **contesto della shell chiamante** (non in una subshell), quindi le modifiche a variabili sono visibili all'esterno — a meno di non dichiararle con `local`.

```bash
function saluta() {
    local messaggio="Ciao, $1!"   # locale alla funzione
    echo "$messaggio"
}
saluta "Lorenzo"    # Ciao, Lorenzo!
echo $messaggio     # vuoto: local non è visibile all'esterno
```

---

## 17. Valutazione delle Condizioni

In bash, le strutture di controllo (`if`, `while`, `until`) valutano **comandi** (non espressioni booleane). L'exit code del comando determina la condizione: **0 = vero**, **qualsiasi altro valore = falso**.

La variabile `$?` contiene l'exit code dell'ultimo comando eseguito.

### I comandi test, [ ] e [[ ]]

| Comando | Descrizione |
|---------|-------------|
| `test espressione` | Valuta l'espressione; exit code 0 se vera. |
| `[ espressione ]` | Sinonimo di `test` (gli spazi sono obbligatori). |
| `[[ espressione ]]` | Versione estesa bash-specifica. |

#### Operatori unari (test/[ ])

| Operatore | Significato |
|-----------|-------------|
| `-z stringa` | True se la stringa è vuota. |
| `-n stringa` | True se la stringa è non vuota. |
| `-e file` | True se il file esiste. |
| `-f file` | True se è un file regolare. |
| `-d file` | True se è una directory. |
| `-s file` | True se il file esiste e ha dimensione > 0. |

#### Operatori binari (test/[ ])

| Tipo | Operatori |
|------|-----------|
| **Stringhe** | `=`, `!=`, `<`, `>` (ordine lessicografico) |
| **Numerici** | `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge` |
| **File** | `-nt` (più recente), `-ot` (più vecchio) |

#### Estensioni di [[ ]]

`[[ ]]` aggiunge rispetto a `[ ]`:

- **Pattern matching**: `[[ "$VAR" == *.txt ]]` (glob sul lato destro non quotato).
- **Regex**: `[[ "$VAR" =~ ^[0-9]+$ ]]` (espressione regolare POSIX estesa).
- Non richiede il quoting delle variabili per gestire spazi (ma rimane buona pratica).

#### Combinare condizioni

Con `test`/`[ ]`: `-a` (AND), `-o` (OR), `!` (NOT).

Con `[[ ]]`: `&&` (AND), `||` (OR), `!` (NOT).

Tramite exit code dei processi (si applica a qualsiasi comando, non solo a `test`):

```bash
cmd1 && cmd2    # esegue cmd2 solo se cmd1 ha successo (exit 0)
cmd1 || cmd2    # esegue cmd2 solo se cmd1 fallisce (exit ≠ 0)
! cmd1          # inverte l'exit code
```

---

## 18. Istruzione if

```bash
if COMANDO1; then
    BLOCCO1
elif COMANDO2; then
    BLOCCO2
else
    BLOCCO3
fi
```

`COMANDO1` e `COMANDO2` sono comandi qualsiasi: l'esecuzione di `BLOCCO1` avviene se `COMANDO1` restituisce exit code 0.

```bash
# Esempi tipici
if [ -f "$file" ]; then
    echo "Il file esiste"
elif [ -d "$file" ]; then
    echo "È una directory"
else
    echo "Non esiste"
fi

if grep -q "pattern" file.txt; then
    echo "Pattern trovato"
fi
```

---

## 19. Istruzione case

`case` confronta una stringa con una serie di pattern (glob), eseguendo il blocco corrispondente al primo match:

```bash
case STRINGA in
    pattern1)
        COMANDI ;;
    pattern2|pattern3)
        COMANDI ;;
    *)
        COMANDI_DEFAULT ;;
esac
```

- I pattern supportano i metacaratteri del globbing (`*`, `?`, `[SET]`).
- `|` separa pattern alternativi.
- `*)` funge da default (corrisponde a tutto).
- `;;` termina ogni clausola.

```bash
case "$1" in
    start)   echo "Avvio..." ;;
    stop)    echo "Arresto..." ;;
    restart) echo "Riavvio..." ;;
    *.conf)  echo "File di configurazione" ;;
    *)       echo "Uso: $0 {start|stop|restart}" ;;
esac
```

---

## 20. Cicli for

### for foreach (stile bash)

```bash
for NOME [in PAROLE...]; do
    COMANDI
done
```

Se `in PAROLE` è omesso, itera sugli argomenti posizionali (`"$@"`).

Le `PAROLE` possono provenire da:

```bash
# Pathname expansion
for f in *.txt; do echo "$f"; done

# Argomenti posizionali
for arg in "$@"; do echo "$arg"; done

# Command substitution
for utente in $(cut -d: -f1 /etc/passwd); do echo "$utente"; done

# Brace expansion
for i in {1..10}; do echo "$i"; done
```

### for C-style

```bash
for (( init ; condizione ; incremento )); do
    COMANDI
done
```

```bash
for (( i=0; i<10; i++ )); do
    echo "$i"
done

# Con più variabili
for (( i=0, j=0; i+j < 10; i++, j+=2 )); do
    echo "i=$i j=$j"
done
```

### Sequenze con $(seq)

`$(seq start [increment] end)` genera sequenze flessibili:

```bash
for n in $(seq 1 2 20); do
    echo "$n"     # 1 3 5 7 ... 19
done

for n in $(seq 10 -1 1); do
    echo "$n"     # 10 9 8 ... 1
done
```

---
