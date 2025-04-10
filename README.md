# CI/CD con GitHub Actions

Questo repository utilizza **GitHub Actions** per implementare un flusso di integrazione continua (CI) e distribuzione continua (CD), definito tramite due file principali:

- `CI.yml`: Esegue test e verifiche automatiche su ogni `push`
- `CD.yml`: Esegue il deploy automatico in produzione al completamento di una *pull request* sulla branch `main`

---

## 📦 CI - Integrazione Continua (`.github/workflows/CI.yml`)

Il workflow CI viene eseguito automaticamente **a ogni push** e include le seguenti fasi:

### 🔍 Fasi principali

1. **Clonazione del repository**
2. **Installazione tool di sistema**
   - `tree`, `shellcheck`, `perl`, `micro`
3. **Analisi degli script Bash**
   - Usa `shellcheck` ignorando alcuni warning comuni
4. **Installazione dipendenze Perl**
   - Utilizzo di `cpanm` per installare moduli come `Dancer2`, `MongoDB`, `DBIx::Class`, ecc.
5. **Verifica sintassi script Perl**
6. **Setup di Docker**
7. **Test ambienti di base**
   - Sincronizzazione di `/servers/base/` in `/base`
   - Esecuzione dello script `0_step.sh`
8. **Verifica e avvio dei daemon**
   - Inclusi: `mariadb`, `mongod`, `memcached`, `nginx`, `munin-node`
9. **Validazione configurazione di Nagios**

Questo processo garantisce che il codice caricato sia testato in un ambiente il più simile possibile alla produzione.

---

## 🚀 CD - Deploy Continuo (`.github/workflows/CD.yml`)

Il workflow CD viene eseguito **al completamento di una pull request sulla branch `main`** (`pull_request: closed`).

### ⚙️ Funzionalità principali

1. **Checkout del codice**
2. **Elenco dei file presenti nel workspace**
3. **Deploy remoto via rsync**
   - Sincronizza la cartella `servers/base` con il server remoto `207.154.210.208`
   - Utilizza la chiave SSH memorizzata come secret GitHub (`SSH_PRIVATE_KEY`)
   - Impiega l'action [`burnett01/rsync-deployments`](https://github.com/Burnett01/rsync-deployments)

Questo processo consente di distribuire automaticamente il codice testato su un server remoto in modo sicuro e veloce.

---

## 🔐 Sicurezza

Il deploy è protetto tramite l’uso di **chiavi SSH** archiviate nei `Secrets` di GitHub Actions, evitando così l'esposizione di credenziali sensibili.

---

## 📁 Struttura repository consigliata

```
📁 servers/
   └── 📁 base/
         ├── 0_step.sh
         └── altri script e configurazioni
```

---

## ✅ Requisiti

- GitHub Actions abilitato
- Secrets GitHub configurati: `SSH_PRIVATE_KEY`
- Accesso root al server remoto per il deploy
- Supporto per `systemctl` per gestire i servizi

