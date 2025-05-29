#  Documentazione Tecnica del Progetto – CloudTube

**Autore:** Jacopo Guerandi
**Collaboratori**: Acampora Niccolò, Bassani Andrea, Cervadoro Ernesto, Monzani Tommaso, Panzera Federico, 
**Tecnologie:** PHP, MySQL, JavaScript, CSS, HTML  
**Data:** Aprile-Maggio 2025

---
## Introduzione

Questo progetto, che ho chiamato _CloudTube_, è una piattaforma web per la gestione e la visualizzazione di video.  
L'obiettivo era creare un'applicazione semplice ma funzionale per il caricamento e la visualizzazione di contenuti multimediali, organizzati per canali.

Ho utilizzato **PHP** per il backend, **MySQL** per la gestione dei dati, **JavaScript** per interazioni dinamiche e **CSS** per la parte visiva. L'interfaccia è minimale ma funzionale e permette un’esperienza utente fluida.
## Configurazione dell’Ambiente

### Macchine Virtuali su Proxmox

#### VM: `webserver`

- **Nome:** `webserver`
- **Sistema Operativo:** Ubuntu Server 22.04 LTS
- **IP:** `192.168.1.20`
- **Ruolo:** Hosting dell’applicazione CloudTube (Apache2 + PHP)
- **Percorso codice progetto:** `/var/www/html/Cloudtube/`

```bash
#Credenziali di accesso:
Username: ttf
Password: 12345678ttf
```

```bash
#Comandi utili da terminale:
sudo su
Password: 12345678ttf
cd /var/www/html/Cloudtube
```
---


#### **VM: mysql-db**

- Nome: mysql-db
- Sistema Operativo: Ubuntu Server 22.04 LTS
- IP: 192.168.1.10
- Ruolo: Database server (MySQL)
- Database: my_insanej
- Utente: jack
- Password: 1234
## Configurazione dell’accesso remoto a MySQL

Per permettere  la connsessione tra `webserver`  e `mysql-db`, è necessario configurare MySQL affinché accetti connessioni da remoto.

### 1. Modifica del parametro `bind-address`

Aprire il file di configurazione principale di MySQL:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Cercare la riga:

```bash
bind-address = 127.0.0.1
```
Modificare in:

```bash
bind-address = 0.0.0.0
```

> In alternativa, per una configurazione più sicura, è possibile specificare direttamente l’indirizzo IP della VM `webserver`:

```bash

bind-address = 192.168.1.20

#Salvare e chiudere il file (`CTRL + O`, `INVIO`, poi `CTRL + X`).
```
---

### 2. Riavvio del servizio MySQL

Per rendere effettive le modifiche, riavviare il servizio MySQL:

```bash

sudo systemctl restart mysql
```
Verificare che il servizio sia attivo:

```bash
sudo systemctl status mysq
```
---

### 3. Configurazione del firewall (facoltativa ma consigliata)

Per motivi di sicurezza, è consigliato limitare l’accesso alla porta `3306` del database solo alla macchina `webserver`.

Eseguire sulla VM `mysql-db`:

```bash
sudo ufw allow from 192.168.1.20 to any port 3306
```
Verificare che la regola sia attiva:

```bash
sudo ufw status
```
---

### 4. Verifica accesso da remoto

Dal server `webserver`, testare la connessione con:

```bash
mysql -u jack -p -h 192.168.1.10
#Inserire la password quando richiesta (`1234` nel file `dbconnection.php`).
```
MySQL è configurato per accettare connessioni remote modificando bind-address in:
```bash
bind-address = 0.0.0.0
```
**Regola firewall:**
```bash
sudo ufw allow from 192.168.1.20 to any port 3306
```
**Collegamento tra le Macchine:**
La macchina Webserver si connette al database MySQL ospitato su mysql-db all'indirizzo ```192.168.1.10```
Configurazione presente in settings/dbconnection.php:
```php
<?php

try {

$host = '192.168.1.10'; // indirizzo ip della macchina

$dbname = 'my_insanej'; // Nome del database 

$username = 'jack'; // Nome utente 

$password = '1234'; // Password 

$pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);

$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

} catch (PDOException $e) {

die("Errore di connessione al database: " . $e->getMessage());

}

?>
```
---

## Struttura del Progetto

- `index.php`: punto di accesso iniziale al sito. 
	- - `Cloudtube/`
    
    - `home.php`: pagina principale visibile dopo il login.
    
	- `Upload.php`: gestisce il caricamento dei video sul server.
    
	- `displayUploadedVideos.php`: mostra l’elenco dei video caricati.
    
	- `canali.php` e `lista_canali.php`: gestiscono i canali e la loro visualizzazione.
    
	- `settings/`
    
	    - `sessions.php`: gestisce l’avvio e la verifica delle sessioni utente.
        
	    - `dbconnection.php`: configura la connessione al database MySQL.
        
	- `javascript/`
    
	    - `sidebar.js`: permette di mostrare e nascondere la sidebar.
        
	    - `shadows.js`: gestisce effetti visivi sulle ombre.
        
	- `css/`
		- `style.css`: contiene tutto lo stile grafico del sito.`
    
	- `uploads/`: cartella per i video caricati dagli utenti. 
	- `Immagini/`: directory contenente logo, immagini utente e grafiche del sito.
---

## Funzionalità Principali
### Gestione Sessioni (`login.php`)

Controlla se un utente ha una sessione attiva.  
In caso contrario, viene eseguito un redirect automatico alla pagina di login (non inclusa nei file analizzati).  
È una protezione basilare ma fondamentale per evitare accessi non autorizzati.
![login](login.png)
### Homepage (`home.php`)

La homepage rappresenta il punto centrale del sito dopo l’accesso. Integra le funzionalità della sidebar, richiami a pagine come "Carica Video" o "Canali", e fornisce una struttura chiara all’utente autenticato.
![home](home.png)

###  Caricamento Video (`Upload.php`)

Permette all’utente di caricare uno o più file video nel sistema.  
Il file viene spostato nella directory ==**`uploads/`**== e può essere successivamente richiamato per la visualizzazione.  
Vengono eseguiti controlli di validità sui file caricati per verificarne l’estensione e le dimensioni.
![upload](upload.png)
### Visualizzazione Video (`displayUploadedVideos.php`)

Legge la cartella **`uploads/`** e genera dinamicamente la lista dei video presenti.  
Ogni video viene presentato con un player HTML5 incorporato.
![video](videocaricati.png)
### Gestione Canali (`canali.php`, `lista_canali.php`)

Tramite **`canali.php`**![canali](Channels.png) l’utente può accedere alla visualizzazione dei canali esistenti.  
`lista_canali.php`
<div style="text-align: center;">
	<img src="cartella-sito/listacanali.png" >
	<img src="cartella-sito/videosqlcodice.png">
	</div>
	
si collega al database e visualizza dinamicamente i canali e i video associati.  
La gestione dei canali è strutturata per semplificare la categorizzazione dei contenuti.
### Connessione al Database (`dbconnection.php`)

Contiene la configurazione della connessione MySQL, con host, utente, password e nome database.  
È centrale per tutte le operazioni che coinvolgono il database, come la visualizzazione dei canali o dei metadati dei video.
``` php
<?php

try {

$host = '192.168.1.10'; // indirizzo ip della macchina mysql

$dbname = 'my_insanej'; // Nome del database 

$username = 'jack'; // Nome utente 

$password = '1234'; // Password 

$pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);

$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

} catch (PDOException $e) {

die("Errore di connessione al database: " . $e->getMessage());

}

?>
```
### Interazione Frontend (`sidebar.js`, `shadows.js`, `style.css`)

- **`sidebar.js`**: permette di aprire o chiudere la barra laterale in base alle interazioni dell’utente.
```js
const toggleSidebarButton = document.getElementById('toggleSidebar');

const sidebar = document.getElementById('sidebar');

const navbarContent = document.getElementById('navbarContent');

  

// Mostra o nasconde la sidebar

toggleSidebarButton.addEventListener('click', () => {

sidebar.classList.toggle('active');

document.getElementById('mainContent').classList.toggle('sidebar-active');

  

// Sposta il pulsante in alto quando la sidebar è aperta

if (sidebar.classList.contains('active')) {

toggleSidebarButton.style.top = '15px';

} else {

toggleSidebarButton.style.top = '80px';

}

});

  

// Nascondi il pulsante della sidebar quando la navbar è aperta

navbarContent.addEventListener('show.bs.collapse', () => {

toggleSidebarButton.style.opacity = '0';

toggleSidebarButton.style.pointerEvents = 'none';

});

  

// Mostra il pulsante della sidebar quando la navbar è chiusa

navbarContent.addEventListener('hide.bs.collapse', () => {

toggleSidebarButton.style.opacity = '1';

toggleSidebarButton.style.pointerEvents = 'auto';

});
```
    
- `shadows.js`: gestisce effetti dinamici, come le ombre su elementi scrollabili.
```js
window.addEventListener('DOMContentLoaded', () => {

const alerts = document.querySelectorAll('.alert');

alerts.forEach(alert => {

setTimeout(() => {

alert.classList.add('fade-out');

setTimeout(() => alert.remove(), 500); // Attendi che svanisca

}, 2000); // Dopo 2 secondi

});

});
```
- `style.css`: definisce colori, layout, tipografia e altre proprietà CSS che danno un aspetto coerente al sito.
```css
body {

padding-top: 70px; /* Altezza della navbar */

}

  

.sidebar {

width: 250px;

position: fixed;

top: 0;

left: -250px; /* Nascondi la sidebar inizialmente */

height: 100vh;

background-color: #343a40;

padding-top: 70px; /* Altezza della navbar */

transition: left 0.3s ease-in-out;

z-index: 1040;

}

  

.sidebar a {

color: white;

text-decoration: none;

display: block;

padding: 10px;

}

  

.sidebar a:hover {

color: #0dcaf0;

}

  

.main-content {

margin-left: 0; /* Nessun margine iniziale su mobile */

padding: 20px;

transition: margin-left 0.3s ease-in-out;

}

  

/* Pulsante per aprire la sidebar */

#toggleSidebar {

position: fixed;

top: 80px; /* Posizionato sotto la navbar */

left: 15px;

z-index: 1050;

background-color: #343a40;

color: white;

border: none;

border-radius: 5px;

padding: 10px;

cursor: pointer;

transition: top 0.3s ease-in-out, opacity 0.3s ease-in-out; /* Transizione per spostare e nascondere il pulsante */

}

  

#toggleSidebar i {

font-size: 1.5rem;

}

  

/* Mostra la sidebar quando è attiva */

.sidebar.active {

left: 0;

}

  

/* Aggiusta il contenuto principale quando la sidebar è visibile */

.main-content.sidebar-active {

margin-left: 250px;

}

  

/* Sposta il pulsante in alto quando la sidebar è aperta */

.sidebar.active + #toggleSidebar {

top: 15px; /* Posiziona il pulsante in alto */

}

  

/* Nascondi il pulsante quando la navbar è aperta */

.navbar-collapse.show ~ #toggleSidebar {

opacity: 0;

pointer-events: none; /* Disabilita il clic sul pulsante */

}

  

.fade-out {

transition: opacity 0.5s ease-out;

opacity: 0;

}
```
---

## Configurazione e Requisiti

<span style="color:rgb(66, 66, 99)">Per il corretto funzionamento del progetto sono necessari:</span>

- **PHP v7.4 (o superiore)**
    
- **Database MySQL attivo** (macchina accesa)
    
- **Server Apache/Nginx**
    
- **Permessi di scrittura** sulla cartella `uploads/`
    
**```Facoltativo```**
- **Composer**, per la gestione avanzata delle dipendenze PHP
    

Prima dell’esecuzione:

1. Configurare **`settings/dbconnection.php`** con le credenziali corrette del database.
    
2. Assicurarsi che **`uploads/`** sia scrivibile dal server web.
    
3. Caricare eventuali librerie con Composer (è incluso **`composer.phar`** e **`composer.lock`**).    
---