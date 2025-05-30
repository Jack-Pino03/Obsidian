# Deploy di Web App ad Alta Disponibilità su Microsoft Azure
---

<font color="#4bacc6">Autore: Jacopo Guerandi  </font>
<font color="#4bacc6">Corso: Microsoft Azure  </font>
<font color="#4bacc6">Data: Venerdì 30 Maggio 2025</font>

---
## Introduzione

In ambienti cloud moderni è fondamentale garantire la disponibilità continua delle applicazioni, anche in caso di guasti o manutenzioni. 
Utilizzando un **Azure Load Balancer**, possiamo distribuire il traffico tra le VM attive e implementare un **failover** automatico: se una VM smette di rispondere, il traffico verrà automaticamente reindirizzato verso quella funzionante. Questo approccio è alla base della progettazione di sistemi resilienti e scalabili nel cloud.

## Obiettivo

Realizzare un'infrastruttura Azure composta da:
- Due VM Linux con web server Apache,
- Load Balancer pubblico con health probe,
- IP statico pubblico,
- Regole di sicurezza e failover automatico.
## Architettura di riferimento

```mermaid
graph TD
    User --> LoadBalancer
    LoadBalancer --> VM1[macchinabrumbrum1]
    LoadBalancer --> VM2[macchinabrumbrum2]
```

---

## Procedura dettagliata

### 1. Creazione del Resource Group

1. Accedi al Portale Azure.
2. Vai su `"Resource Groups"` → `"Create"`.
3. Imposta:
   - **Name:** `"inserisci nome risorsa"`
   - **Region:** `North Europe (o uno a scelta)`
1. Completa la creazione.

---

### 2. Provisioning delle Virtual Machine (VM)

Per ciascuna VM (macchinabrumbrum1 e macchinabrumbrum2):

- **OS:** `Ubuntu Server 22.04 LTS`  
- **Size:** `B1s ` 
- **Authentication:** `Password o SSH`  
- **Virtual Network:** `VNet-WebApp`
- **Subnet:** `default`
- **Public IP:**` Statico, abilitato`  
- **Ports:** `22 (SSH), 80 (HTTP) ` 
- **DNS label:** `"nome a scelta".northeurope.cloudapp.azure.com`

---

### 3. Installazione Web Server sulle VM

Accedi via SSH a ciascuna VM:

**Macchina n1**
```bash
sudo apt update && apt upgrade -y
sudo apt install apache2 -y
echo "<h1>["inserire un contenuto a scelta come test"]</h1>" | sudo tee /var/www/html/index.html
```

**Macchina n2**
```bash
sudo apt update && apt upgrade -y
sudo apt install apache2 -y
echo "<h1>["inserire un contenuto a scelta come test"]</h1>" | sudo tee /var/www/html/index.html
```

Verifica l’accesso via browser agli IP pubblici/DNS di ciascuna VM.

| VM1                                       | VM2                                       |
| ----------------------------------------- | ----------------------------------------- |
| ![macchina1](Cartella-sito/macchina1.png) | ![macchina2](Cartella-sito/macchina2.png) |
| **Accesso:** `indirizzo ip`               | **Accesso:** `DNS`                        |

---

### 4. Configurazione del Load Balancer

1. Vai su `"Create a resource"` → `"Networking"` → `"Load Balancer"`.
2. Imposta:
   - **Type:** `Public`
   - **SKU:** `Standard`
   - **Name:** `"inserire nome`
   - **IP Address:** `Statico`
3. Associa il Load Balancer alla VNet creata.

---

### 5. Configurazione dei Backend Pool

- **Name:** `"inserisci nome"`
- **Aggiungi:** `la-prima-macchina, la-seconda-macchina` 
- **Network:** `VNet-WebApp`

---

### 6. Configurazione HealthProbe

- **Name:** `"inserisci nome"`
- **Protocol:** `HTTP`
- **Port:** `80`
- **Path:** `/`
- **Interval:** `5s`
- **Unhealthy Threshold:** `2`

---

### 7. Regole di Bilanciamento

- **Name:** `"inserisci nome"`
- **Frontend port:** `80`
- **Backend port:** `80`
- **Backend pool:** `Inserisci la backend-pool creata in precedenza`
- **Health probe:** `Inserisci la probe creata in precedenza`
- **Session persistence:** `None`

---

### 8. Configurazione Network Security Group (NSG)

Per ogni VM:
- **Inbound Rule:**  
  - **Name:** `allow-http`
  - **Port:** `80`  
  - **Protocol:** `TCP`
  - **Source:** `Any`
  - **Action:** `Allow`

---

### 9. Test di Failover e Verifica

1. Accedi a `http://<IP_statico_del_LoadBalancer>` dal browser.
2. Spegni una delle due VM dal portale Azure.
3. Dopo 15-30 secondi, aggiorna il browser: il traffico sarà servito dalla VM attiva.
4. Riaccendi la VM spenta e verifica il ritorno del traffico bilanciato.

---
