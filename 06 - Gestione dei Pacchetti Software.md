# **Laboratorio di Amministrazione di Sistemi — Gestione dei Pacchetti Software**

**Docente:** Prof. Marco Prandini
**Argomento:** Ciclo di vita del software, installazione manuale e assistita, sistemi a pacchetti (deb/rpm), repository, creazione di pacchetti e repo personalizzati, Virtual Environments e Containers.

---

## **1. Il Ciclo di Vita del Software**

La gestione del software su un sistema in produzione si articola in tre fasi fondamentali: **installazione**, **aggiornamento** e **disinstallazione**. Ciascuna di queste operazioni solleva problematiche trasversali che l'amministratore deve affrontare: la verifica dei prerequisiti hardware e del sistema operativo, la gestione delle **dipendenze** (sia da altri componenti software sia di altri componenti dal software in questione) e la corretta **configurazione** post-installazione.

---

## **2. Installazione Manuale**

### **2.1 Da Binari**

L'installazione da binari precompilati consiste nella semplice copia dei file eseguibili nei percorsi appropriati del filesystem. Richiede tuttavia una **verifica manuale** della compatibilità con l'architettura della macchina e del soddisfacimento di tutte le dipendenze.

### **2.2 Da Sorgente**

L'installazione da codice sorgente impone un passaggio di **compilazione**, ma offre in compenso l'indipendenza dall'architettura e una maggior flessibilità nel soddisfacimento delle dipendenze, poiché i sorgenti possono disporre di diverse interfacce per adeguarsi a ciò che è presente sul sistema.

### **2.3 Dipendenze e Pacchetti di Sviluppo**

Nel caso di installazione da binari, è necessario disporre non solo dei software indicati come prerequisiti, ma anche che essi siano di una **versione specifica**. Nel caso di installazione da sorgente si ha un certo grado di flessibilità, ma occorre disporre non solo dei componenti runtime, bensì anche delle **librerie di sviluppo** (prototipi, interfacce, librerie per collegamento statico). In un sistema ideale, dove si dispone di tutti i sorgenti, questi elementi sono sempre presenti. Nelle distribuzioni reali, per flessibilità, ogni pacchetto software ha un corrispondente pacchetto **`-dev`** (su sistemi deb) o **`-devel`** (su sistemi rpm).

---

## **3. Installazione Manuale Tipica in Linux**

Il caso più comune è quello di software distribuito per mezzo di un archivio `tar.gz`, scritto in C e predisposto alla compilazione tramite **autoconf**. Lo strumento `autoconf` svolge le seguenti funzioni: verifica se sono soddisfatti tutti i prerequisiti, rileva le versioni e le collocazioni dei pacchetti sul sistema, accetta dall'utente la specifica di varianti (attivazione/disattivazione di funzionalità, preferenze architetturali) e genera i **Makefile** sulla base delle specificità del sistema e delle scelte operate.

### **3.1 I Passi Tipici**

La procedura standard si articola in sei fasi sequenziali:

1. **Reperimento del software** — Download dell'archivio sorgente.
2. **Estrazione del pacchetto** — L'archivio si presenta solitamente come `tar.gz`. È buona prassi determinare una collocazione sensata per i sorgenti (es. `/usr/local/src`) ed estrarre in tale directory. È prudente **testare l'archivio** prima dell'estrazione (`tar tvzf`) per verificare la gerarchia di directory che genera.
3. **Esame delle scelte disponibili** — Si entra nella directory generata dall'estrazione e si esaminano i file **README** e **INSTALL**. Se presente un eseguibile `configure`, lo si lancia con `--help` per ottenere la lista dei parametri di configurazione (collocazione del software, attivazione/disattivazione di sottocomponenti, predisposizione dei moduli dinamicamente caricabili).
4. **Configurazione dei sorgenti** — Si lancia nuovamente `configure` con i parametri scelti. Si risolvono i problemi evidenziati (tipicamente l'assenza di pacchetti prerequisiti). Il tool `configure` non è a prova d'errore e genera un `config.log` utile per il debugging.
5. **Compilazione** — Si lancia `make` o si seguono le indicazioni nell'output del passo precedente.
6. **Installazione** — Si lancia `sudo make install`.

**⚠️ Avvertenza sulla Sicurezza:** Solo l'ultima operazione (installazione) può richiedere i diritti di superutente. Si deve pertanto evitare di compiere le operazioni precedenti come `root`. Sono noti casi di **malware** che sfruttano la cattiva abitudine di eseguire una o più delle operazioni preliminari con diritti eccessivi.

### **3.2 Autenticazione del Software Scaricato**

La prima cautela da adottare quando si scarica software è verificarne l'**autenticità** tramite firma digitale. Per verificare una firma serve una chiave pubblica fidata.

**Verifica con firma GPG:**

```bash
gpg --verify FILE.asc FILE.tar.gz          # mostra il key id
gpg --keyserver pgpkeys.mit.edu --recv-key <KEY_ID>
gpg --verify FILE.asc FILE.tar.gz          # ripetere la verifica
```

L'autenticità della chiave in questo caso deriva dalla fonte da cui la si è ottenuta: occorre **valutare caso per caso** e seguire le indicazioni specifiche del progetto.

**Verifica con fingerprint (sha256):**

Come minimo, dovrebbe essere disponibile un fingerprint. Basta collocare il file `.sha256` nella stessa directory del file da verificare e lanciare:

```bash
sha256sum -c FILE.sha256
```

---

## **4. Riflessioni sull'Installazione da Sorgenti**

L'installazione da sorgenti offre la possibilità teorica di **verificare il codice**. Tuttavia, se non lo si fa realmente, si genera un falso senso di sicurezza: fidarsi della firma sull'archivio sorgente non è diverso dal verificare la firma su un binario. L'installazione da sorgenti è inoltre più difficile da manutenere e richiede **molti componenti ausiliari** (header e librerie, processori di macro, compilatori e linker), tutti elementi che possono potenzialmente avvantaggiare un attaccante (si veda il celebre **Ken Thompson Hack**).

Entrano dunque in scena le **distribuzioni e i pacchetti**, che offrono: chiavi di verifica installate una volta per tutte (comodo, ma rappresenta un *Single Point of Failure*), gestione automatica delle dipendenze e garanzia di compatibilità binaria tra tutti gli elementi del set.

---

## **5. Installazione Assistita: Il Sistema a Pacchetti**

L'installazione assistita è comunemente effettuata per mezzo di **software ausiliari**: i **package manager** specifici della distribuzione Linux (rpm/yum, dpkg/apt, ...) o gli installer per Windows.

Un tool di installazione può farsi carico delle verifiche relative alle dipendenze, può generare dinamicamente dati specifici, ma non può configurare ogni dettaglio del sistema in modo specifico.

### **5.1 Struttura di un Pacchetto**

Le distribuzioni di Linux organizzano il software in **pacchetti** e dispongono di un package manager per la loro gestione. Un pacchetto si presenta sotto forma di singolo file che contiene in forma compatta l'insieme di: software precompilato, criteri per la verifica della compatibilità e dei prerequisiti, e procedure di pre/post-installazione.

La garanzia della compatibilità con un determinato sistema può essere data solo a patto di vincolare con precisione: **architettura**, **versione della distribuzione** e **versione del software** contenuto nel pacchetto.

### **5.2 Formati dei Pacchetti**

**Formato .deb** (Debian e derivate, es. Ubuntu):
La nomenclatura segue lo schema `nome-versione_software-versione_pacchetto_architettura.deb`. Ad esempio: `aptitude-0.2.15.9-2_i386.deb`.

**Formato .rpm** (Red Hat e derivate, es. CentOS, Fedora):
La nomenclatura segue lo schema `nome-versione_software-versione_pacchetto.distro.architettura.rpm`. Ad esempio: `httpd-2.4.6-45.el7.centos.x86_64.rpm`.

### **5.3 Pacchetti Base e Development**

Per ogni libreria (es. `zlib1g`) il pacchetto base contiene solo il file `.so` (shared object) per il **linking dinamico** a runtime. Il pacchetto corrispondente di sviluppo (es. `zlib1g-dev`) contiene invece la libreria statica `.a` per il **linking statico**, e i file header `.h` (prototipi per il compilatore).

Con questa suddivisione si risparmia molto spazio sui sistemi che non sono usati per sviluppare codice. Su sistemi deb si parla di pacchetti **"-dev"**, su sistemi rpm di pacchetti **"-devel"**.

La verifica delle dipendenze dinamiche di un binario si effettua con il comando `ldd`:

```bash
ldd /usr/sbin/sshd
```

---

## **6. Distribuzioni: Criteri per la Scelta**

### **6.1 Architetture Supportate**

Tutte le distribuzioni supportano i processori Intel 32bit, la maggior parte quelli a 64bit; alcune sono disponibili per tutte le architetture su cui è stato portato il kernel. I pacchetti di terze parti potrebbero non essere disponibili per tutte le architetture supportate.

### **6.2 Stabilità vs. Aggiornamento**

Il rilascio frequente e continuo del software nel mondo GNU/Linux ha come conseguenza che le versioni più aggiornate possano essere meno stabili. Vi sono distribuzioni con filosofia di inclusione dei pacchetti più recenti (a costo di minor robustezza) e altre che garantiscono l'inclusione del solo software ben collaudato.

### **6.3 Version vs. Rolling**

Alcune distribuzioni sono **"versionate"**: durante il ciclo di vita di una versione vengono forniti solo aggiornamenti correttivi, mentre le novità vengono accumulate per la pubblicazione della versione successiva. Altre sono **"rolling"**: ogni novità viene testata e distribuita incrementalmente, mantenendo il sistema costantemente alla versione più recente.

### **6.4 Supporto e Durata**

La disponibilità di supporto garantito è tipica delle distribuzioni commerciali, ma anche quelle gratuite più diffuse beneficiano di ampie comunità. Per installazioni server esistono le varianti **LTS** (*Long Term Support*): per 5–7 anni i maintainer garantiscono che gli aggiornamenti non modifichino le API, tipicamente fornendo solo il backporting dei security fix.

### **6.5 Ampiezza del Set di Pacchetti**

Si va dai circa 1.500 pacchetti delle distribuzioni minimali ai 26.000 di Debian. Una scelta intelligente di distribuzione mette tutto l'essenziale in un singolo supporto di installazione.

---

## **7. Le Due Famiglie: Debian e Red Hat**

Debian e Red Hat sono le due distribuzioni capostipite da cui sono state derivate quasi tutte le varianti più diffuse. I due sistemi di gestione dei pacchetti presentano molte somiglianze architetturali e si articolano su tre livelli funzionali:

- **Tool di basso livello** per la gestione dei singoli pacchetti (`dpkg` / `rpm`).
- **Tool intermedi** per la gestione coordinata di pacchetti e dipendenze.
- **Tool per il reperimento automatico** da **repository** dei pacchetti necessari (`apt` / `yum`→`dnf`).

### **7.1 Repository**

I pacchetti possono essere scaricati e gestiti singolarmente, ma normalmente si usano i **repository** (repo): raccolte indicizzate di pacchetti che possono essere online o su filesystem locali. I package manager leggono per ogni repo l'indice e i metadati dei pacchetti, conoscono quali versioni sono disponibili e come risolvere le dipendenze.

**Collocazione delle liste di repo:**

| Famiglia | File di configurazione |
|---|---|
| Debian/Ubuntu | `/etc/apt/sources.list` e `/etc/apt/sources.list.d/*` |
| Red Hat/CentOS | `/etc/yum.conf` e `/etc/yum.repos.d/*.repo` |

---

## **8. Gestione dei Pacchetti .deb (Debian/Ubuntu)**

| Operazione | Comando |
|---|---|
| Database location | `/var/lib/dpkg`, `/var/lib/apt` |
| Sources file | `/etc/apt/sources.list` |
| Update sources | `apt update` |
| Key management | Chiavi gpg direttamente in `/etc/apt/trusted.gpg.d/` (apt-key è **deprecato**) |
| Search | `apt search keywords` |
| Install (locale) | `dpkg -i filename.deb` |
| Install (da repo) | `apt install packagenames` |
| Upgrade | `apt upgrade [packagenames]` |
| Remove | `dpkg -r packagename` oppure `apt remove packagenames` |

---

## **9. Gestione dei Pacchetti .rpm (Red Hat/CentOS/Fedora)**

| Operazione | Comando |
|---|---|
| Database location | `/var/lib/rpm` |
| Sources file | `/etc/yum.conf` |
| Update sources | `yum update` |
| Key management | `rpm --import keyfile` |
| Search | `yum search keywords` |
| Install (locale) | `rpm -i filename.rpm` |
| Install (da repo) | `yum install packagenames` |
| Upgrade | `yum upgrade [packagenames]` |
| Verify integrity | `rpm -V [packagenames\|a]` |
| Remove | `rpm -e packagenames` oppure `yum remove packagenames` |

*Nota:* `yum` è mostrato perché ancora ampiamente utilizzato, ma è stato deprecato in favore di **`dnf`** (e **`zypper`** su Suse), con sintassi molto simili.

---

## **10. Verifica dell'Autenticità dei Pacchetti**

La firma dei pacchetti è gestita centralmente. I maintainer di una distribuzione forniscono le chiavi di verifica nei media di installazione ufficiali o sui repository online. I set di chiavi possono essere gestiti con **GnuPG** (es. i sistemi deb-based collocano i gpg keyring in `/etc/apt/trusted.gpg.d/`), ma è più comune usare strumenti forniti dalla distribuzione stessa:

| Famiglia | Gestione chiavi |
|---|---|
| .deb | `apt-key {add file \| list \| del keyid \| adv --recv-key keyid \| ... }` |
| .rpm | `rpm {--import \| -e \| -q[ai] \| ...}` |

Nel mondo rpm, le chiavi sono trattate come se fossero pacchetti: si possono usare gli stessi comandi per interrogarle, eliminarle, ecc.

---

## **11. Lavorare con Repository Esterni**

Un'esigenza comune è installare software ben supportato ma non incluso nei canali ufficiali della distribuzione. Si aggiunge semplicemente il repository all'elenco.

**Esempio Apt (deb):**

```
/etc/apt/sources.list.d/virtualbox.list :
deb http://download.virtualbox.org/virtualbox/debian xenial contrib
```

**Esempio Yum (rpm):**

```
/etc/yum.repos.d/epel.repo :
[epel]
name=Epel Linux
baseurl=http://mirror.example.com/repo/epel5_x86_64
enabled=1
gpgcheck=0
```

### **11.1 Gestire la Provenienza dei Pacchetti**

Si può generare confusione se un pacchetto con lo stesso nome è presente in versioni diverse in repository differenti. I package manager, di default, scelgono sempre la versione più avanzata. Ciò comporta un rischio di **software injection**: supponiamo di aver aggiunto un repo semisconosciuto per installare un'applicazione innocua; se a tale repo viene aggiunto un pacchetto "core" dichiarato più recente della versione ufficiale, il package manager lo installerà in modo silente.

**Controllo della provenienza:**

| Famiglia | Comando |
|---|---|
| Yum | `repoquery -i [package name]` |
| Apt | `apt-cache showpkg [package name]` |

**Elenco dei pacchetti provenienti da un repo:**

| Famiglia | Comando |
|---|---|
| Yum | `yum list installed \| grep [repo name]` |
| Apt | Vari comandi per estrarre manualmente le info dai file della cache |

### **11.2 Limitare le Modifiche Automatiche (Version Locking/Pinning)**

Per evitare problemi in sistemi con dipendenze complesse (ad esempio mix di pacchetti installati manualmente e via package manager), è possibile bloccare la versione di specifici pacchetti.

**Apt:** Editare i file in `/etc/apt/preferences.d/*`.

**Yum:** Installare il plugin `yum-plugin-versionlock`, quindi usare `yum versionlock [package name]` oppure editare manualmente `/etc/yum/pluginconf.d/versionlock.list`.

---

## **12. Creazione di Pacchetti e Repository Personalizzati**

I package manager sono molto utili per "tenere in ordine" il sistema. È sconsigliabile mischiare installazioni manuali con pacchetti. Non è difficile **pacchettizzare le proprie applicazioni**.

### **12.1 Build Your Own Repo (RPM)**

**Configurazione dell'ambiente** (per un utente non root) tramite il file `~/.rpmmacros`:

```
%packager    Marco Prandini <marco.prandini@unibo.it>
%vendor      DISI
%_topdir     /home/prandini/rpmbuild
%_signature  gpg
%_gpg_name   Marco Prandini (Unibo) <marco.prandini@unibo.it>
```

Sotto `_topdir` devono essere presenti le cartelle: **SPECS** (file di specifica dei pacchetti), **SOURCE** (sorgenti da compilare/includere), **BUILD** (contenitore per il pacchetto "aperto"), **SRPMS** (destinazione dei pacchetti sorgente) e **RPMS** (destinazione dei pacchetti binari).

Il **file di specifiche** (`.spec`) contiene: l'elenco dei sorgenti (`Sources0`, `Sources1`...), le dipendenze esplicite (`Requires`), le dipendenze di build (`BuildRequires`) e il `BuildRoot` (dove il build viene eseguito).

```bash
rpmbuild -ba --rmsource --sign test.spec
```

Il comando genera i pacchetti in `RPMS/architettura/` e `SRPMS/`. Si copiano i file rpm sul server web (es. `/var/www/repos/myrepo/RPMS`) e si lancia:

```bash
createrepo /var/www/repos/myrepo/RPMS
```

### **12.2 Build Your Own Repo (DEB)**

Il processo è simile ma con alcune differenze: la generazione di pacchetti è più semplice (nessuna automazione della compilazione), ma la gestione dei repository è più complessa. Non vi è una struttura di file obbligatoria.

**Procedura:**

1. Creare la gerarchia di directory: `~/distro/` con sottocartelle `sources/`, `packages/` e `apt-repo/`.
2. In `sources/` creare il programma (es. uno script bash `hello-world`).
3. In `packages/hello-world_0.0.1-1_all/` riprodurre la gerarchia di directory del sistema target (`mkdir -p usr/bin && cp ~/distro/sources/hello-world usr/bin/`).
4. Creare la directory `DEBIAN` con il file `control`:

```
Package: hello-world
Version: 0.0.1
Maintainer: Marco Prandini <marco.prandini@unibo.it>
Depends: bash
Architecture: all
Homepage: http://example.com
Description: A program that prints hello
```

5. Creare il pacchetto `.deb`:

```bash
dpkg-deb -Zgzip --build ./packages/hello-world_0.0.1-1_all
```

6. Verificare il risultato:

```bash
dpkg-deb --info ./packages/hello-world_0.0.1-1_all.deb
dpkg-deb --contents ./packages/hello-world_0.0.1-1_all.deb
```

**Firma del pacchetto:** Si genera una coppia di chiavi con `gpg --full-generate-key` e la si pubblica (tramite `gpg --export --armor > pubkey-appsec.txt` o su un keyserver con `gpg --keyserver pgp.mit.edu --send-key <KEY_ID>`).

**Gestione del repository con Aptly:**

```bash
aptly config show
aptly repo create -distribution=jammy -component=main apt-repo
aptly repo add apt-repo ./packages/
aptly repo show -with-packages apt-repo
aptly snapshot create hello-1.0 from repo apt-repo
aptly publish -architectures=all snapshot hello-1.0
```

**Uso del custom repo sui client:**

Creare un file `/etc/apt/sources.list.d/hello.list` contenente:

```
deb file:<HOMEDIR>/distro/apt-repo/public jammy main
```

Sui client la chiave pubblica deve essere importata. Se ottenuta da file:

```bash
gpg -o pubkey-appsec.gpg --dearmor pubkey-appsec.txt
sudo cp pubkey-appsec.gpg /etc/apt/trusted.gpg.d/
```

Test finale con `apt update && apt install hello-world`.

---

## **13. Problematiche di Aggiornamento**

Quando si aggiorna un pacchetto software già in uso, si deve tener conto di potenziali problemi derivanti da:

- **Prerequisiti:** I pacchetti da cui dipende il candidato devono esistere e potrebbe non essere facile aggiornarli senza causare problemi ad altri software che li utilizzano.
- **Configurazione:** Eventuali modifiche incompatibili al formato delle direttive di configurazione già messe a punto per la versione funzionante.
- **Dipendenze di altri software e test di non regressione:** Le modifiche apportate alle interfacce o alle funzionalità del software potrebbero influire sul funzionamento di altri software. Occorre curare la configurazione del `PATH` per impostare l'ordine di ricerca degli eseguibili nelle directory e predisporre configurazioni di test per far coesistere le due versioni durante le fasi di verifica (es. binding a socket, porte, IP diversi).
- **Configurazione del loader:** Per far convivere differenti versioni di librerie dinamiche, si veda la man page `ld(1)` (sezioni sui parametri `-rpath` e `-rpath-link`). Si possono modificare i settaggi di default in `/etc/ld.so.conf` (da applicare con `ldconfig`) o usare le variabili `LD_LIBRARY_PATH` in fase di loading e `LD_RUN_PATH` in fase di linking.

---

## **14. Disinstallazione**

La disinstallazione presenta gli stessi problemi dell'aggiornamento in termini di eventuale dipendenza di altri software dal pacchetto che si sta per rimuovere. In entrambi i casi può essere molto difficile prevedere gli effetti sul sistema se la gestione è manuale. Il **grafo delle dipendenze** è quindi il valore aggiunto più significativo dei sistemi a pacchetti. Può essere molto utile sfruttare la possibilità offerta dai package manager di creare i propri pacchetti anche per il software installato manualmente, così da gestirlo tramite il sistema automatico di verifica delle dipendenze.

---

## **15. Snap e Flatpak: Pacchetti Autocontenuti**

Per software che deve essere particolarmente longevo, indipendente dalle librerie, basato su funzioni non standard di una distribuzione, o distribuito in modo indipendente dai canali ufficiali, esistono formati di pacchetto autocontenuti:

- **snap** (Ubuntu)
- **flatpak** (vendor neutral)

Questi formati includono all'interno del pacchetto tutte le dipendenze necessarie all'esecuzione.

---

## **16. Virtual Environments e Containers**

### **16.1 Il Problema delle Macchine Virtuali**

Le macchine virtuali rispondono a necessità comuni quali preconfigurare sistemi, isolare diversi ambienti di esecuzione e definire limiti di uso delle risorse. Tuttavia sono spesso eccessivamente pesanti: impongono l'installazione di un intero SO (con inutile replicazione di librerie e utilità e overhead di memoria in esecuzione) e limitano l'accesso *bare metal* alle risorse.

### **16.2 La Soluzione: Isolamento a Livello di Processo**

La soluzione consiste nell'isolare l'ambiente di esecuzione di gruppi di processi **condividendo il sistema operativo**. L'isolamento non è totale (non si possono eseguire applicazioni incompatibili col sistema), ma l'accesso all'hardware è diretto e le risorse condivise non sono replicate. Questo paradigma è implementato da tecnologie quali OpenBSD Jails, Solaris Zones e **Linux namespaces**.

### **16.3 Le Tre Funzionalità del Kernel**

Il kernel Linux fornisce tre meccanismi fondamentali a supporto dei Virtual Environments:

**Control Groups (cgroups):** Misurano e limitano la quantità di risorse (memoria, CPU, I/O, rete) che un processo può consumare. Controllano inoltre l'accesso ai device (via `/dev`).

**Namespaces:** Mostrano a un processo istanze di risorse fisiche condivise; un processo in un namespace non vede la risorsa fisica reale, e l'istanza gli appare come a suo uso esclusivo. Ogni processo vive in un namespace di ogni tipo tra: `mnt` (mount points, filesystems), `pid` (processes), `net` (network stack), `ipc` (System V IPC), `uts` (hostname) e `user` (UIDs).

**Union-capable filesystems:** Combinano due supporti per mostrare un filesystem unico (es. snapshot).

### **16.4 Containers**

Usando il partizionamento di risorse si possono definire i **containers**: definiscono in modo coerente cgroups e namespaces, specificano come esporre le risorse viste dal processo interno al sistema e includono il software necessario all'esecuzione del processo. La gestione dei container è semplificata da strumenti come **LXC** (LinuX Containers), **Docker**, **CoreOS rkt** e **Apache Mesos**.
