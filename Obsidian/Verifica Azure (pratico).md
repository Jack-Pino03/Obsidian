# Web App con Alta Disponibilità su Azure

Autore: Jacopo Guerandi  
Corso: Microsoft Azure  
Data: Venerdì 30 Maggio 2025

---

## Introduzione

In ambienti cloud moderni è fondamentale garantire la disponibilità continua delle applicazioni, anche in caso di guasti o manutenzioni. Questo esercizio pratico mira a implementare un'infrastruttura in Azure in grado di erogare un'applicazione web semplice attraverso due macchine virtuali Linux configurate in alta disponibilità.

Utilizzando un Azure Load Balancer, possiamo distribuire il traffico tra le VM attive e implementare un failover automatico: se una VM smette di rispondere, il traffico verrà automaticamente reindirizzato verso quella funzionante. Questo approccio è alla base della progettazione di sistemi resilienti e scalabili nel cloud.

---

## Obiettivo

Implementare un’architettura ad alta disponibilità per una web app su Azure, composta da:

- Due macchine virtuali Linux con web server installato
    
- Load Balancer pubblico con health probe sulla porta 80
    
- DNS o IP pubblico raggiungibile
    
- Failover automatico in caso di guasto di una VM
    

---

## 1. Creazione del Resource Group

Da Azure Portal:

- Nome: RG-WebAppHA
    
- Regione: East US (o altra disponibile)
    

---

## 2. Prima VM – macchinabrumbrum1

- Immagine: Ubuntu Server 22.04 LTS
    
- Dimensione: B1s
    
- Autenticazione: chiave SSH
    
- Rete virtuale: VNet-WebApp
    
- Subnet: default
    
- IP pubblico: abilitato
    
- Porte aperte: 22 (SSH), 80 (HTTP)
    
- DNS name label: verificaazure.northeurope.cloudapp.azure.com (esempio)
    

Installazione Apache:

bash

CopiaModifica

`sudo apt update sudo apt install apache2 -y echo "<h1>Web App Attiva - MACCHINA 1</h1>" | sudo tee /var/www/html/index.html`

Verifica nel browser:

http://verificaazure.northeurope.cloudapp.azure.com

---

## 3. Seconda VM – macchinabrumbrum2

Creata con le stesse impostazioni della prima VM:

- Stessa immagine, taglia, rete, porte
    
- Pagina web personalizzata:
    

bash

CopiaModifica

`echo "<h1>Web App Attiva - MACCHINA 2</h1>" | sudo tee /var/www/html/index.html`

---

## 4. Creazione del Load Balancer

Dal Portale Azure:

- Tipo: Load Balancer Pubblico
    
- SKU: Standard
    
- Nome: LB-macchinabrumbrum
    
- IP pubblico: statico (creato durante la procedura)
    
    - Nome IP pubblico: ip-lb-brumbrum
        

---

## 5. Backend Pool

- Nome: brumbrum-pool
    
- VM aggiunte: macchinabrumbrum1, macchinabrumbrum2
    
- Rete: VNet-WebApp
    
- Nic: interfacce delle due VM
    

---

## 6. Health Probe

- Nome: http-probe
    
- Protocollo: HTTP
    
- Porta: 80
    
- Percorso: /
    
- Intervallo: 5 sec
    
- Failures tollerate: 2
    

---

## 7. Regola di Bilanciamento

- Nome: web-rule
    
- Porta frontend: 80
    
- Porta backend: 80
    
- Backend Pool: brumbrum-pool
    
- Health Probe: http-probe
    
- Session persistence: Nessuna
    

---

## 8. Regole di Sicurezza (NSG)

Per entrambe le VM:

- Regola in entrata:
    
    - Nome: allow-http
        
    - Porta: 80
        
    - Protocollo: TCP
        
    - Origine: Any
        
    - Azione: Allow
        

---

## 9. Test del Failover

- Aprire nel browser: http://<IP_statico_del_LoadBalancer>
    
- Spegnere una delle VM
    
- Attendere 15-30 secondi
    
- Ricaricare il browser: il traffico viene inoltrato alla VM attiva
    

---

## Conclusioni

L'infrastruttura realizzata permette di sperimentare una soluzione cloud resiliente con:

- Accessibilità pubblica da Internet
    
- Distribuzione del carico tra due VM Linux
    
- Failover automatico gestito dal Load Balancer
    
- Indirizzo IP statico pubblico raggiungibile
    

Questo tipo di architettura costituisce la base per applicazioni mission-critical, ambienti di test robusti e distribuzioni su larga scala in ambienti Azure.