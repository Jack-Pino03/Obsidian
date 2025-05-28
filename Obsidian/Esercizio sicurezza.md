#  Analisi OSINT – Superficie di Attacco Pubblica del Gruppo FS Italiane

##  Obiettivo
Analizzare la superficie di attacco pubblicamente visibile del Gruppo Ferrovie dello Stato Italiane (FS), identificando informazioni accessibili via OSINT che un attaccante potrebbe aggregare per pianificare un attacco, *senza effettuare intrusioni*.

---

## 1. Struttura del Gruppo

*Fonti utilizzate:* Wikipedia, fsitaliane.it, LinkedIn

•⁠  ⁠Holding pubblica italiana (100% MEF)
•⁠  ⁠Filiali principali:
  - Trenitalia
  - RFI (infrastrutture)
  - Italferr (engineering)
  - Ferservizi, Mercitalia, Busitalia
•⁠  ⁠Settori: trasporto ferroviario, infrastrutture, logistica, servizi IT

---

##  2. Presenza Online

### Domini principali:

•⁠  ⁠⁠ fsitaliane.it ⁠
•⁠  ⁠⁠ trenitalia.com ⁠
•⁠  ⁠⁠ rfi.it ⁠
•⁠  ⁠⁠ busitalia.it ⁠
•⁠  ⁠⁠ italferr.it ⁠
•⁠  ⁠⁠ mercitalia.it ⁠

### Sottodomini identificati:

•⁠  ⁠⁠ www.fsitaliane.it ⁠
•⁠  ⁠⁠ portaleferservizi.fsitaliane.it ⁠
•⁠  ⁠⁠ vpn.fsitaliane.it ⁠
•⁠  ⁠⁠ sso.fsitaliane.it ⁠
•⁠  ⁠⁠ jira.fsitaliane.it ⁠
•⁠  ⁠⁠ cloud.fsitaliane.it ⁠
•⁠  ⁠⁠ dev.trenitalia.fsitaliane.it ⁠

*Strumenti usati:* Amass, DNSDumpster, crt.sh, Google Dorking

---

## 3. Dati di Registrazione Domini

*Esempio - fsitaliane.it:*

•⁠  ⁠*Registrar:* Aruba S.p.A.
•⁠  ⁠*DNS:* dns1.ferroviedellostato.it, dns2…
•⁠  ⁠*Creazione:* 2000
•⁠  ⁠*Scadenza:* ~2030
•⁠  ⁠*SPF/DMARC:* parzialmente configurati

*Strumenti usati:* Whois, dig, ViewDNS.info, MXToolbox

---

## 4. Analisi della Superficie di Attacco

### Obiettivi:

•⁠  ⁠Individuare sottodomini non standard
•⁠  ⁠Verificare uso di ambienti di test/dev
•⁠  ⁠Identificare esposizione a servizi cloud

*Sottodomini sensibili (esempi):*

| Sottodominio | Descrizione Presunta |
|--------------|----------------------|
| ⁠ jira.fsitaliane.it ⁠ | Gestione ticketing (Atlassian) |
| ⁠ dev.trenitalia.fsitaliane.it ⁠ | Ambiente di test |
| ⁠ vpn.fsitaliane.it ⁠ | Accesso remoto |
| ⁠ cloud.fsitaliane.it ⁠ | Possibile portale cloud privato |

*Strumenti usati:* Sublist3r, Shodan, Censys, Wayback Machine

---

##  5. Analisi Servizi e Versioni

*Obiettivi:* identificare software esposti, versioni obsolete

| Dominio | Tecnologia Rilevata | Versione |
|--------|----------------------|----------|
| ⁠ fsitaliane.it ⁠ | Apache | 2.4.29 (obsoleta) |
| ⁠ jira.fsitaliane.it ⁠ | Atlassian Jira | 8.x |
| ⁠ cloud.fsitaliane.it ⁠ | Liferay | sconosciuta |

*Strumenti usati:* WhatWeb, Nmap, Wappalyzer, Censys, Shodan

---

## 6. Ricerca Persone e Tecnologie

*Fonti:* LinkedIn, Google Dorks, Hunter.io

### Profili rilevanti:

•⁠  ⁠*Mario R.* – IT Security Manager, Ferservizi
•⁠  ⁠*Luisa G.* – Project Manager, Italferr
•⁠  ⁠*Stefano B.* – System Engineer, Trenitalia

### Informazioni trovate:

•⁠  ⁠Uso di strumenti: Splunk, ServiceNow, Atlassian, Office 365
•⁠  ⁠Post pubblici che menzionano progetti, strumenti e ambienti

*Google Dork esempio:*
```text
site:linkedin.com/in "ferrovie dello stato" AND ("cybersecurity" OR "IT")