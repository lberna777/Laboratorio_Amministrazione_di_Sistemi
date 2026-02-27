# **Laboratorio di Sicurezza Informatica — Creazione e gestione VM con Vagrant e SSH**

**Docente:** Prof. Marco Prandini  
**Argomento:** Protocollo Secure Shell (SSH), Autenticazione a chiave pubblica, Infrastructure as Code con Vagrant.

## **1\. Secure Shell (SSH): Concetti Base**

L'amministrazione remota dei sistemi richiede un protocollo sicuro. In passato veniva utilizzato **TELNET**, il quale presentava due criticità fatali:

* Nessuna confidenzialità (il traffico viaggia in chiaro, rendendo le credenziali vulnerabili allo sniffing).  
* Nessuna autenticazione dell'host (rischio elevato di attacchi *Man-in-the-Middle*).

**SSH** risolve questi problemi introducendo un canale cifrato, l'autenticazione dell'host e l'autenticazione attiva dell'utente.

### **L'Autenticazione a Chiave Pubblica (Crittografia Asimmetrica)**

Il meccanismo si basa su un sistema *Challenge-Response*:

* Il **Prover (P)** (l'utente) possiede una **Chiave Privata** (![][image1]) che non deve mai viaggiare in rete. È il vero elemento di identificazione perché non è condivisa con nessuno.  
* Il **Verifier (V)** (il server) possiede la **Chiave Pubblica** di P (![][image2]). È matematicamente legata alla privata, ma non permette di ricavarla.  
* Il server genera una sfida (un *nonce* casuale generato da un PRNG), la cifra con la chiave pubblica dell'utente o chiede all'utente di firmarla.  
* Se l'utente possiede la chiave privata corretta, decifra la sfida, invia la risposta e prova la sua identità senza svelare alcun segreto sul canale.

## **2\. Le 5 Fasi della Connessione SSH**

Quando si avvia un collegamento tra client (ssh) e server (sshd), avvengono in sequenza i seguenti passaggi:

1. **Negoziazione dei cifrari:** Le due macchine concordano gli algoritmi crittografici disponibili.  
2. **Autenticazione dell'host (Server \-\> Client):** Il client verifica l'identità del server per evitare di consegnare credenziali a un impostore.  
   * *Trust on first use:* Non essendoci un'autorità centrale, alla primissima connessione l'utente deve utilizzare un **metodo out-of-band** (es. telefonare all'amministratore, leggere una documentazione fidata) per confermare l'impronta della chiave pubblica del server.  
   * Una volta accettata, la chiave del server viene salvata nel file locale \~/.ssh/known\_hosts per l'autenticazione attiva nelle sessioni future.  
3. **Inizializzazione del canale:** Viene stabilito il canale di comunicazione cifrato.  
4. **Negoziazione dei metodi utente:** Si concorda come l'utente si autenticherà.  
5. **Autenticazione dell'utente (Client \-\> Server):**  
   * *Passiva:* Tramite username e password (viaggia sicura grazie al tunnel cifrato al punto 3).  
   * *Attiva:* Tramite protocollo challenge-response a chiave pubblica (preferita).

*Nota sintattica:* L'identità con cui ci si presenta si definisce con ssh utente@remoteserver. Se omesso, viene assunto lo stesso username dell'operatore sul client locale.

## **3\. Gestione delle Chiavi SSH**

Per utilizzare l'autenticazione attiva (Passwordless login), occorre preparare l'ambiente:

### **1\. Generazione della coppia di chiavi (sul Client)**

ssh-keygen \-t rsa \-b 2048

Genera una chiave privata (id\_rsa) e una pubblica (id\_rsa.pub). La chiave privata può essere protetta da una *passphrase* aggiuntiva: priva della comodità del passwordless login puro, ma è molto più sicura (specie se si amministrano molti server) se il file dovesse essere sottratto.

### **2\. Installazione della chiave pubblica (sul Server)**

Esistono due metodi:

* **Metodo Automatico (consigliato):** ssh-copy-id \[-i chiave\] user@remote  
* **Metodo Manuale (2 step):**  
  1. Copia file sul server: scp .ssh/id\_rsa.pub user@remote:  
     *(Nota fondamentale: il carattere : finale è vitale, dice a scp che si tratta di un host remoto e che il file va piazzato nella home directory di default dell'utente).*  
  2. Inserimento in coda alla lista autorizzati (collegandosi prima via SSH): cat id\_rsa.pub \>\> .ssh/authorized\_keys

**⚠️ Avvertenza sui Permessi:** La segretezza della password è ora sostituita dalla riservatezza della chiave. Spesso il login senza password fallisce semplicemente perché i **permessi di file e directory** sull'host remoto (in particolare la directory .ssh) sono troppo larghi (es. 777). Il server sshd controlla i permessi e "non si fida" del file authorized\_keys se sospetta che altri abbiano potuto modificarlo.

## **4\. SSH come Filtro: L'Esecuzione Remota**

SSH non serve solo per ottenere un terminale interattivo remoto, ma può eseguire comandi al volo ("One-shot") sfruttando il reindirizzamento degli stream I/O sul canale cifrato.  
Esempio:  
ssh root@server "grep pattern"

* I dati forniti attraverso lo **STDIN** locale (al processo ssh) vengono trasportati cifrati e immessi nello **STDIN** del comando grep lanciato sul server.  
* Lo **STDOUT** e lo **STDERR** generati dal processo grep sul server remoto "fuoriescono" dagli analoghi stream sul client locale.

## **5\. Vagrant: Infrastructure as Code**

Vagrant è uno strumento pensato per garantire la **riproducibilità affidabile e la portabilità** di ambienti di test virtualizzati, astraendo dallo specifico motore di virtualizzazione (Provider). Questo è essenziale per replicare i laboratori a casa.  
**Il Modello Vagrant:**

* **Box:** L'immagine di base del sistema operativo, depositata e prelevabile da *Vagrant Cloud*.  
* **Vagrantfile:** Il file di testo dichiarativo che definisce come istanziare la VM (RAM, IP, cartelle).  
* **Provider:** Il sistema di virtualizzazione sottostante (es. *VirtualBox*).

### **Anatomia di un Vagrantfile**

\# \-\*- mode: ruby \-\*-  
Vagrant.configure("2") do |config|  
  \# 1\. Scelta della Box di base  
  config.vm.box \= "debian/bookworm64"  
    
  \# 2\. Rete: IP fisso e mappatura porte Host-\>Guest  
  config.vm.network "private\_network", ip: "192.168.33.10"  
  config.vm.network "forwarded\_port", guest: 80, host: 8080  
    
  \# 3\. Cartelle sincronizzate (Host \-\> Guest)  
  config.vm.synced\_folder "code/", "/app/code"  
    
  \# 4\. Provider e risorse hardware  
  config.vm.provider "virtualbox" do |vb|  
    vb.memory \= 2048  
    vb.cpus \= 1  
  end  
    
  \# 5\. Provisioner: Azioni eseguite dentro la VM al primo avvio  
  config.vm.provision "shell", inline: \<\<-SHELL  
    apt-get update  
    apt-get install \-y apache2  
    service apache2 start  
  SHELL  
end

### **Comandi Vagrant Essenziali**

Operazioni da eseguire nella directory contenente il Vagrantfile:

* vagrant box add autore/versione \[url\]: Scarica preventivamente una box in locale (utile per scaricare grosse VM prima di eseguire file dichiarativi).  
* vagrant init autore/box: Crea un Vagrantfile di default.  
* vagrant up: Se necessario scarica la box, crea la VM tramite il provider e la avvia.  
* vagrant status: Mostra lo stato della/delle VM configurate.  
* vagrant halt: Invia un segnale di arresto pulito (spegnimento).  
* vagrant destroy: Elimina le risorse della VM (il Vagrantfile rimarrà intatto per ricostruirla).

## **6\. L'Ecosistema Vagrant \+ SSH**

L'integrazione di Vagrant con SSH è automatica e indolore:

* Al primo vagrant up, Vagrant **crea automaticamente una chiave privata** sull'host fisico e installa la rispettiva chiave pubblica nella VM appena creata.  
* L'utente configurato di default si chiama vagrant ed è abilitato a eseguire sudo senza richiesta di password.  
* **Accesso Rapido:** vagrant ssh si connette istantaneamente usando le configurazioni autogenerate.

### **Integrare tool esterni (SCP, Nmap, ecc.)**

Ogni volta che la VM si avvia, Vagrant cerca di creare un **port forwarding** attraverso l'interfaccia NAT di VirtualBox, puntando tipicamente dalla porta host 2222 alla porta guest 22\.  
**⚠️ Attenzione:** Se più macchine sono in esecuzione contemporaneamente sul tuo PC (es. un intero laboratorio), Vagrant non può usare sempre la porta 2222 (che sarà occupata), ma assegnerà *dinamicamente* porte diverse (es. 2200, 2201).  
Per aggirare questa dinamicità e usare tool di rete esterni, bisogna lanciare (con la VM accesa):  
vagrant ssh-config

L'output rivelerà le reali coordinate di connessione:  
Host default  
  HostName 127.0.0.1  
  User vagrant  
  Port 2222  \<-- Potrebbe variare\!  
  IdentityFile /path/to/.vagrant/machines/default/virtualbox/private\_key  
  StrictHostKeyChecking no

Questi dati ci permettono di comporre a mano il comando SSH o SCP aggirando il wrapper di Vagrant.  
Comando SSH effettivo ricavato dall'output:  
ssh \-p 2222 \-i /path/to/private\_key vagrant@127.0.0.1

*(Questo passaggio è vitale se nel laboratorio da casa devi usare scp per trasferire file nella VM di test o lanciare altri tool offensivi).*

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAAYCAYAAACmwZ5SAAADJUlEQVR4Xu2XWahPURTGP0OGJ5EyD29IkiHCg6sUyVR4wqMhPFBE8SCUojx4IJnJlMxRSslcMntQpktmD+bM4fv+a+/utuxz67q83P6/+nLP+tY5e5/zX2vvDShTpkxdYhB1h3pKPQ//3qfuUpXUGWoy1SjkD6TuUc9C/pNw/ZC6RW2i+odczwXqESxXz9Y454N3jXoAm4vGlgYHL7KXegzL0zOm/ObWkN3UT2pAEmtCzQrxVUlc+PwGVCdqF/WdGhXint6w+85SDZ03LXj7qXrOE8q/SK2jujmvxuhrv4FN3KOvqZfomMSK8rvCJn3axSMzYf4Cb8AqQ94xbwT6Uid88G9oDxvoiDdIfeoD9YPqHGIx/3C4ThkC81SaOVSW8vt5g7SBeTe9AZvHOdgHrTUTYQPN9QaqXmBLEov5c5JYZAPMm+CNwEvqLf6sDKEy/gLzPaqMpT74t6yHTbKPi6sn1TNaaFol8ZjfK4m1pXZQr6mpSTylO+y+o95I0AKonGZJTL/8Zdia8k9Q+alkF1HzqYWwyatHN1PNq1JLKP8brNekK9RnaidsckXE/p3njYSTsJweSUwL5LDkulbEftQXHBM0kqpA/ovG/INJTOW5nXpFdUninur6N7IVljMiXOtF9cKeFtQa2Lz3hb9171pYZRYyCTbAYhcvIub7/h0e4qqOHOpP9e875Ps3oj7Vc6bDPrheqKhq1GbKTbeoJbA2LGQj7KYKFy8i5ms/TZkd4howR+zfoi0nosOE8pZTy2BtUMQ42OEnRe2ij5rbx0tUUp+oxt4oQPnqbW0TKSorTVQvnmMGzM/tvykqYeVdgp3y/Dgpq6k9ybVyddrL7TYl1G96+ClvFBDzc6usniFPpyWxjWpXZZf2bPk6ylZHPLhoUezpPM8N6gDsY2qxVQ+PRubX1XHwNmyR0a/7Ebb1qERy6ASklVn5yn0PO32lK6cG+kodh5XjyhDXiUvP1jjSC+pq8HI0he0YK7zhaAnL0xw6oLjP/ysaeCw11Bs1RP9Zye0QKeNhe351JV+n0DZ0yAfrIq1he61W5+uw7atMmTJ1mF/SG8sgWqpY9wAAAABJRU5ErkJggg==>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADkAAAAYCAYAAABA6FUWAAAC7ElEQVR4Xu2WWaiNURTH/2bxYEqIuCVThMiDpK4hRUohyRDdPBBP5kLdB4UXRaJM9yKZp3gmSRnixZTxHmQueRBC+P9b+2Ofdc533M49D7fb+dWv01nrO/vsb397rW8DZcqUacyMoY/pa/o2fD6jT2gdvULn09bh+q4h9zzkM+H7qpAXR0Je4z6lF6NcwiH6Evaf8gXsfzWmfnuYjvx7dYk4Sn/T0VGsLV0W4lujuKgO8XUuLprRubCbGBa+56MvbIwzLt6FXgq5sS7XILSKn2gLn4Ct7E/aO4pdgE1CN5GPRXSzDzrmwMZY4hNkOix3zCeKpRdswPM+QZrTz/QXrYhiWpAPSH9KB+kkH3Tsgv3vQJ8gK2G5bT5RLNpaGlADe8bBcrVRbESInYxiHtVjex903KdvfJD0hO2eh7S7yxXNHtikfaH3oTdgtdUtii+HXa96zUcFveqDDo2nMc7STrQz7Iaq6D16AHazJUOrru24nq6BNRN1N23JGtgkYs7BJjjYxRMW0o0+6JgFG+MybGvrpvR5l+6FlVDJSOrxFp0WnEorYd3Vo3r8SN8jvR5r6UQfdKTVoxrfafqDDnG58fQ4vQ1biJ30FF1BW0XX5TAP9mfVLp7GcNj1vu3HqJba+aDjAX3ng4HkKe/2CViJ6GkntKR36OoolsM+2ICVLp7GbNj1a30iMAW22oXoARvjhE8EFsPymptH7/MdLnaT7nexLOroV9rGJ1KYAJuAtohHzSOD9FpNSN6P+RqX5qHGo+0aH0wS1I1nRt+1s77RflEsiwH4V/z1RdsjQ6+5eAdYQ9IN/A8d6fI1rkGwuXyhC1xOqH71uw10Kd1Ct8PuIwet0CNYA9FT1KB6TcyILyrAUFhH1opvgjWB67CtWoga2NlYE5V6KtpJOm29Cp8aq3/yA4e2sc7COnmpYXbMTpceHdi1dbXiWrT6bveGoCOeFqHJoteVXls6nTVJJsO66ndY3Y/KTpcpU6ax8wcb9q6EP1cHyQAAAABJRU5ErkJggg==>