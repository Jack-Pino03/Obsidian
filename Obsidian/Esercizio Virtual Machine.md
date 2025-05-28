# Esercizio Virtual Machine con Ubuntu 25, SSH, MySQL e LAMP

## 1. Creazione della Virtual Machine

### Sistema:

- Ho scaricato l'**ISO** di Ubuntu 25 (desktop/server)
    
- Ho utilizzato **VMware**
    

### Configurazioni base:

- RAM: 4 GB
    
- CPU: 2 core
    
- Disco: 20 GB dinamico
    
- Rete: Bridge (indirizzo ip: 192.168.1.34)
    

---

## 2. Abilitazione SSH

Dopo aver avviato la macchina, ho aggiornato i pacchetti e installato il server SSH con i seguenti comandi:

``` bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

Ho poi verificato che il servizio SSH fosse attivo con:

``` bash
sudo systemctl status ssh
```

---

##  3. Connessione via SSH dal MacBook

Dal mio MacBook, ho aperto il terminale e mi sono connesso alla VM utilizzando il comando:

``` bash
ssh nome_utente@localhost -p 2222
```

(Sostituendo `nome_utente` con il mio utente Ubuntu)

---

##  4. Installazione e Configurazione di MySQL

Una volta connesso, ho installato il server MySQL:

``` bash
sudo apt install mysql-server -y
```

Ho verificato che il servizio fosse attivo con:

``` bash
sudo systemctl status mysql
```

### Creazione utente MySQL

Sono entrato nel client MySQL con:

``` bash
sudo mysql
```

E ho creato un utente chiamato `jack` con tutti i privilegi:

```
CREATE USER 'jack'@'%' IDENTIFIED BY 'password_sicura';
GRANT ALL PRIVILEGES ON *.* TO 'jack'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Poi ho modificato la configurazione per consentire connessioni da remoto:

``` bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Ho sostituito `bind-address` con:

``` bash
bind-address = 0.0.0.0
```

Infine, ho riavviato MySQL e aperto la porta nel firewall:

``` bash
sudo systemctl restart mysql
sudo ufw allow 3306
```

---

##  5. Test con MySQL Workbench

Con **MySQL Workbench** installato sul mio MacBook, mi sono collegato al server utilizzando:

- **Hostname**: `localhost`
    
- **Porta**: `3306`
    
- **Username**: `jack`
    
- **Password**: `password_sicura`
    

Ho eseguito alcune query per testare la configurazione:

``` bash
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE utenti (id INT PRIMARY KEY, nome VARCHAR(50));
INSERT INTO utenti VALUES (1, 'Mario'), (2, 'Anna');
SELECT * FROM utenti;
```

---

##  6. Snapshot e LAMP Stack

###  Creazione snapshot

Prima di procedere con altre modifiche, ho creato uno snapshot della VM chiamato **“Prima installazione LAMP”**.

### Installazione dello stack LAMP

Ho installato Apache, PHP e i moduli necessari con:

``` bash
sudo apt install apache2 php libapache2-mod-php php-mysql -y
```

Ho verificato che Apache fosse attivo:

``` bash
sudo systemctl status apache2
```

Per testare PHP, ho creato un file nella root di Apache:

``` bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Poi ho aperto il browser all'indirizzo: [http://localhost:8080/info.php](http://localhost:8080/info.php)

### 🧹 Eliminazione dello snapshot

Dopo aver verificato che tutto funzionasse correttamente, ho eliminato lo snapshot direttamente dall'interfaccia dell'hypervisor.

---
---
##  Parte 1: Ho installato Docker e Docker Compose

Prima di iniziare, ho voluto verificare se avevo già Docker installato sulla mia macchina. Ho aperto il terminale e digitato:
```
docker --version
docker-compose --version
```
Purtroppo non avevo ancora niente installato, quindi ho proceduto con l'installazione seguendo questi passi sul mio Ubuntu:

``` bash

# installazione docker
sudo apt install docker.io -y

# installazione docker-compose
sudo apt install docker-compose -y
```
Una cosa importante che ho imparato: dovevo aggiungere il mio utente al gruppo docker per evitare di dover usare sempre `sudo`:

``` bash

sudo usermod -aG docker $USER
newgrp docker
```
Per completare, ho avviato il servizio Docker:

``` bash

sudo systemctl enable docker
sudo systemctl start docker
```
## Parte 2: creazione ambiente di lavoro

Mi piace mantenere tutto organizzato, quindi ho creato una cartella dedicata al progetto:

``` bash

mkdir mio_progetto_mysql
cd mio_progetto_mysql
```
##  Parte 3: configurazione del container MySQL

Questa è stata la parte più interessante! Ho creato un file `docker-compose.yml` con questo contenuto:
``` yaml

version: "3.8"
services:
  mysql_db:
    image: mysql:latest
    container_name: mysql_container
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: la_mia_password_sicura
      MYSQL_DATABASE: es2db
      MYSQL_USER: jack-pino
      MYSQL_PASSWORD: password_utente
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```
Ho fatto particolare attenzione a:

- Usare l'ultima versione di MySQL
    
- Esporre la porta 3306 per le connessioni esterne
    
- Creare un database di default (`es2db`)
    
- Creare un utente dedicato (`jack-pino`)
    
- Usare un volume per persistenza dei dati
    
--- 
Una volta configurato il mysql all'interno di Docker, ho dovuto interrompere il mysql server, quello creato nell'esercizio 1, poiché la porta d'uscita era la medesima del Docker, in questo modo posso utilizzarla per connettermi tramite WorkBench
``` bash
# Blocco/stop di mysql-server

systemctl stop mysql-server

# Verifica spegnimento

systemctl stasus mysql-server

```
##  Parte 4: Ho avviato e testato il container

Con tutto configurato, ho lanciato il container con:

``` bash
docker-compose up -d
```
Per verificare che tutto funzionasse, ho controllato i log:

```
docker logs mysql_container
```
Poi mi sono connesso al database usando il client MySQL da terminale:
```bash

mysql -h 127.0.0.1 -u mario_rossi -p
```
Dopo aver inserito la password, ho potuto verificare che il database `mio_database` era effettivamente presente!

## Parte 5: Ho documentato tutto il processo

Come buona pratica, ho tenuto traccia di:

- Tutti i comandi eseguiti
    
- Le configurazioni applicate
    
- Un paio di problemi incontrati e come li ho risolti:
    
    - All'inizio dimenticavo di esporre la porta 3306
        
    - Una volta ho sbagliato l'indentazione nel file YAML
        

Ho anche testato la connessione da MySQL Workbench sul mio computer host, verificando che potessi effettivamente interagire con il database dal mio client preferito.