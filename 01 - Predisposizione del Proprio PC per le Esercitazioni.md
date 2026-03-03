# Predisposizione del Proprio PC per le Esercitazioni

*Corso di (Laboratorio di) Amministrazione di Sistemi — Prof. Marco Prandini & Alessandro Vannini, Università di Bologna*

---

## INDICE

1. [Opzioni di Predisposizione per Sistema Operativo Host](#1-opzioni-di-predisposizione-per-sistema-operativo-host)
2. [Creazione di una Chiavetta USB Avviabile](#2-creazione-di-una-chiavetta-usb-avviabile)
3. [Dual Boot: Ridimensionamento della Partizione Windows](#3-dual-boot-ridimensionamento-della-partizione-windows)
4. [Il Processo di Avvio: BIOS, UEFI e Secure Boot](#4-il-processo-di-avvio-bios-uefi-e-secure-boot)
5. [Installazione su Windows tramite WSL2](#5-installazione-su-windows-tramite-wsl2)
6. [Installazione su macOS Silicon (Apple M1/M2/M3)](#6-installazione-su-macos-silicon-apple-m1m2m3)
7. [Installazione su macOS Intel](#7-installazione-su-macos-intel)
8. [Installazione su Host GNU/Linux](#8-installazione-su-host-gnulinux)

---

## 1. Opzioni di Predisposizione per Sistema Operativo Host

L'obiettivo finale è disporre di un ambiente in grado di eseguire **VirtualBox**, **Vagrant** e **Ansible** sul proprio computer personale, replicando fedelmente la configurazione del LAB3.

### GNU/Linux (opzione preferita)

È l'opzione raccomandata dal corso. Se non si dispone già di un host GNU/Linux, è possibile installarlo in due modalità:

- **Dual boot**: si affianca Windows sullo stesso disco, ridimensionando la partizione esistente per ricavare circa 30 GB liberi. Al boot si sceglie il sistema operativo desiderato.
- **USB live con persistenza**: si avvia il sistema operativo direttamente dalla chiavetta USB senza modificare il disco interno. Utile per provare l'ambiente, ma le performance sono inferiori.

### macOS

Sia la variante **Apple Silicon** (M1/M2/M3) sia la variante **Intel** sono supportate, con alcune differenze nelle box Vagrant disponibili (si vedano le sezioni dedicate).

### Windows con WSL2

È la soluzione per chi non può o non vuole modificare la propria installazione Windows. Vagrant viene eseguito all'interno del sottosistema Linux (WSL2), mentre le VM vengono comunque gestite da VirtualBox installato sul lato Windows. Questa configurazione richiede alcuni passaggi aggiuntivi di configurazione e presenta qualche limitazione (si veda la sezione dedicata).

---

## 2. Creazione di una Chiavetta USB Avviabile

### Immagine ISO da utilizzare

Il corso utilizza **Xubuntu 24.04.4 LTS** (architettura AMD64):

```
xubuntu-24.04.4-desktop-amd64.iso
```

### Scrittura dell'immagine da GNU/Linux

Il comando `dd` copia l'immagine ISO byte per byte sul dispositivo USB. **Attenzione**: il parametro `of=` deve puntare all'intero dispositivo (es. `/dev/sdb`), non a una sua partizione (es. `/dev/sdb1`). Verificare con `lsblk` il nome corretto del dispositivo prima di procedere.

```bash
sudo dd if=xubuntu-24.04.4-desktop-amd64.iso of=/dev/sdb bs=4M oflag=sync
```

- `bs=4M` — imposta la dimensione del blocco di lettura/scrittura a 4 MiB per ottimizzare le prestazioni.
- `oflag=sync` — forza la sincronizzazione dei buffer sul dispositivo prima di restituire il controllo, garantendo che tutti i dati siano stati effettivamente scritti.

### Scrittura dell'immagine da Windows

Su Windows è possibile utilizzare uno dei seguenti strumenti grafici:

- **Rufus** — https://rufus.ie/ (raccomandato, molto diffuso)
- **Etcher** — https://etcher.balena.io/ (interfaccia più semplice)

### Avviso: Intel RST (Rapid Storage Technology)

Alcuni PC Windows con storage NVMe gestiscono i dischi tramite il driver **Intel RST**. In questo caso, Linux potrebbe non riconoscere il disco durante l'installazione o l'avvio. Prima di procedere, verificare nelle impostazioni del BIOS/UEFI se la modalità SATA/NVMe è impostata su *RAID* (RST) o su *AHCI*. Per poter installare Linux in dual boot occorre passare alla modalità *AHCI*, operazione che tuttavia può compromettere l'avvio del Windows esistente se non eseguita correttamente.

---

## 3. Dual Boot: Ridimensionamento della Partizione Windows

Per installare Linux affiancato a Windows è necessario liberare spazio sul disco riducendo la partizione Windows. Si stima un fabbisogno minimo di circa **30 GB**.

### Procedura con Gestione Disco di Windows

1. Aprire **Gestione Disco** (tasto destro su "Questo PC" → Gestisci, oppure `diskmgmt.msc`).
2. Fare clic con il tasto destro sulla partizione Windows (solitamente la più grande, etichettata C:) e selezionare **Riduci volume...**.
3. Indicare la quantità di spazio da sottrarre in MB (es. 30720 per 30 GB).
4. Confermare: lo spazio liberato apparirà come "Non allocato" e sarà disponibile per il programma di installazione di Linux.

---

## 4. Il Processo di Avvio: BIOS, UEFI e Secure Boot

Comprendere il processo di avvio è fondamentale per diagnosticare problemi di installazione e configurare correttamente il dual boot.

### Sequenza di avvio classica

```
Firmware (BIOS/UEFI)
        ↓
  Boot Loader (es. GRUB)
        ↓
   Kernel Linux
        ↓
    init / systemd
```

### BIOS (Basic Input/Output System)

Il firmware tradizionale, presente sui PC precedenti al 2010 circa. Legge il **MBR** (Master Boot Record) nei primi 512 byte del disco per trovare il boot loader.

### UEFI (Unified Extensible Firmware Interface)

Il firmware moderno, che sostituisce il BIOS. Introduce la **EFI System Partition (ESP)**: una piccola partizione FAT32 che funge da "mini sistema operativo" contenente i boot loader di tutti i sistemi installati. La struttura tipica della ESP è:

```
/EFI/
  Boot/         ← bootloader di fallback (BOOTX64.EFI)
  Microsoft/    ← bootloader di Windows
  ubuntu/       ← bootloader di Ubuntu/Xubuntu
  refind/       ← refind (boot manager alternativo, se installato)
```

UEFI non legge più l'MBR ma cerca direttamente i file `.efi` nella ESP.

### Secure Boot

Secure Boot è una funzionalità di UEFI che verifica la firma crittografica del boot loader prima di eseguirlo, impedendo l'avvio di software non autorizzato. Tutti i boot loader devono essere firmati con una chiave fidata.

Per consentire l'avvio di Linux su sistemi con Secure Boot attivo, **The Linux Foundation** ha negoziato con Microsoft una chiave di firma per un componente detto **pre-bootloader** (shim). In questo modo, GRUB (firmato dal pre-bootloader) è considerato affidabile da Secure Boot senza che sia necessario disattivarlo nel firmware.

### Avvio da USB con UEFI

Per avviare il PC dalla chiavetta USB su sistemi Windows con UEFI, è possibile utilizzare il seguente comando da un prompt PowerShell o CMD con privilegi amministrativi:

```
shutdown.exe /r /fw
```

Questo riavvia il sistema direttamente nelle impostazioni del firmware UEFI, dove è possibile modificare l'ordine di avvio.

---

## 5. Installazione su Windows tramite WSL2

### 5.1 Installazione di WSL2 e Ubuntu

Aprire PowerShell o il Prompt dei comandi come amministratore ed eseguire:

```powershell
wsl --install -d Ubuntu
```

Questo comando installa il sottosistema WSL2 e la distribuzione Ubuntu. Al termine riavviare il sistema e completare la configurazione dell'utente Ubuntu al primo avvio.

### 5.2 Installazione di VirtualBox (lato Windows)

VirtualBox deve essere installato sul sistema Windows nativo (non dentro WSL). Scaricarlo dal sito ufficiale: https://www.virtualbox.org/

### 5.3 Configurazione di Vagrant dentro WSL2

Aprire il terminale Ubuntu (WSL2) e procedere come segue.

**Abilitare i repository universe e multiverse** (necessari per alcuni pacchetti):

```bash
sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo apt update
```

**Aggiungere il repository HashiCorp e installare Vagrant:**

```bash
# Scaricare e aggiungere la chiave GPG di HashiCorp
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Aggiungere il repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt-get install vagrant
```

**Installare Ansible:**

```bash
sudo apt install ansible ansible-lint
```

**Installare il plugin Vagrant per WSL2:**

```bash
vagrant plugin install virtualbox_WSL2
```

### 5.4 Configurazione delle variabili d'ambiente

Aggiungere al file `~/.bashrc` le seguenti righe per consentire a Vagrant (eseguito in WSL) di comunicare con VirtualBox (installato su Windows):

```bash
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"
```

Ricaricare la configurazione:

```bash
source ~/.bashrc
```

### 5.5 Utilizzo di Vagrant da WSL2

Le VM Vagrant devono essere gestite da una directory che risiede nel filesystem Windows (accessibile da WSL sotto `/mnt/c/`), non nel filesystem interno di WSL. Ad esempio:

```
/mnt/c/Users/<username>/vagrant-vms/
```

**Configurare il mount di /etc/wsl.conf** per gestire correttamente i metadati dei file (necessario affinché Vagrant possa impostare i permessi corretti):

```ini
# /etc/wsl.conf
[automount]
options="metadata"
```

**Scaricare la box Debian Bookworm per la prima volta:**

```bash
vagrant box add debian/bookworm64
```

### 5.6 Fix rete WSL2

VirtualBox crea delle schede di rete virtuali che potrebbero essere bloccate dal firewall di Windows. Per risolvere eventuali problemi di connettività delle VM, disabilitare la regola del firewall relativa alla scheda virtuale WSL.

### 5.7 Fix SSH per Vagrant su WSL2

Se si riscontrano problemi con la connessione SSH alle VM, aggiungere la seguente direttiva al `Vagrantfile` **prima** di eseguire `vagrant up`:

```ruby
config.ssh.insert_key = false
```

Il `Vagrantfile` di base si genera con:

```bash
vagrant init debian/bookworm64
```

---

## 6. Installazione su macOS Silicon (Apple M1/M2/M3)

### 6.1 Installare Homebrew

Se non già presente, installare il package manager Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 6.2 Installare VirtualBox per Apple Silicon

VirtualBox ha rilasciato supporto sperimentale per Apple Silicon. Installarlo tramite brew (verificare la disponibilità della versione aggiornata sul sito ufficiale):

```bash
brew install --cask virtualbox
```

### 6.3 Installare Vagrant

```bash
brew install --cask vagrant
```

### 6.4 Installare Ansible

```bash
brew install ansible
ansible --version   # per verificare la corretta installazione
```

### 6.5 Box Vagrant per Apple Silicon

Poiché le VM devono girare in architettura ARM64, occorre utilizzare una box compatibile. Il corso raccomanda:

```bash
vagrant box add KrampusBoxes/debian12
```

---

## 7. Installazione su macOS Intel

La procedura è analoga a quella per Apple Silicon, con l'unica differenza che si può utilizzare la box Debian standard (architettura AMD64).

### 7.1 Installare Homebrew, VirtualBox, Vagrant

```bash
# Homebrew (se non presente)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# VirtualBox e Vagrant
brew install --cask virtualbox
brew install --cask vagrant
```

### 7.2 Installare Ansible

```bash
brew install ansible
ansible --version   # per testare la corretta installazione
```

### 7.3 Box Vagrant per macOS Intel

Su Intel è possibile utilizzare la box ufficiale Debian standard oppure cercarne una compatibile sul portale HashiCorp:

```bash
vagrant box add debian/bookworm64
```

Per selezionare box alternative compatibili: https://portal.cloud.hashicorp.com/vagrant/discover?sort=created_at%20desc

---

## 8. Installazione su Host GNU/Linux

### 8.1 Installazione dei pacchetti

È possibile installare VirtualBox, Vagrant e Ansible tramite il package manager della propria distribuzione. Per ottenere le versioni più aggiornate si consiglia tuttavia di seguire le istruzioni dei rispettivi siti ufficiali:

- **VirtualBox**: https://www.virtualbox.org/
- **Vagrant**: https://www.vagrantup.com/ (repository HashiCorp)
- **Ansible**: https://docs.ansible.com/

```bash
# Esempio generico (adattare alla propria distribuzione)
sudo apt install virtualbox vagrant ansible ansible-lint
```

Per Ansible, installare anche i componenti aggiuntivi necessari al corso:

```bash
# ansible-lint (linter per playbook Ansible)
sudo apt install ansible-lint

# Collection posix di Ansible Galaxy
ansible-galaxy collection install 'ansible.posix:==1.5.4'
```

### 8.2 Prima configurazione: scaricare la box Debian

Sul proprio computer, la prima operazione da compiere è scaricare l'immagine del sistema operativo che verrà usata da Vagrant per istanziare le VM:

```bash
vagrant box add debian/bookworm64
```

> **Nota:** In laboratorio questo passaggio è già stato eseguito dallo script `setup_new.sh` fornito dal corso; sul proprio PC personale occorre eseguirlo manualmente la prima volta.

---

*Nota: tutte le procedure descritte in questo documento presuppongono un sistema host a 64 bit con supporto alla virtualizzazione hardware (Intel VT-x o AMD-V) abilitato nel BIOS/UEFI.*
