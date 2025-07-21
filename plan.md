# Piano di Sviluppo: Firma Decentralizzata dei Report di Conformità

**Autore:** Gemini
**Data:** 2025-07-20
**Stato:** Da Iniziare

## 1. Panoramica

Questo documento descrive il piano di sviluppo per introdurre un sistema di firma digitale decentralizzato per i report di conformità generati dal TCK. L'obiettivo è permettere agli implementatori di SDK di firmare i propri risultati di test in modo verificabile, garantendo autenticità, integrità e tracciabilità senza la necessità di una chiave privata centralizzata.

Il sistema si baserà su un registro pubblico di chiavi gestito tramite Pull Request sul repository del TCK.

## 2. Requisiti

| ID | Requisito | Priorità |
|:---|:---|:---|
| REQ-1 | Gli implementatori di SDK devono poter firmare un report di conformità usando la propria chiave privata. | Alta |
| REQ-2 | Chiunque deve poter verificare l'autenticità di un report firmato usando strumenti forniti dal TCK. | Alta |
| REQ-3 | Il report firmato deve contenere metadati non ripudiabili, inclusi i commit hash del TCK e del SUT. | Alta |
| REQ-4 | Il TCK deve mantenere un registro pubblico e affidabile delle chiavi pubbliche degli implementatori di SDK. | Alta |
| REQ-5 | Il processo di aggiunta di una nuova chiave pubblica al registro deve essere gestito tramite Pull Request. | Media |
| REQ-6 | Il sistema deve essere documentato chiaramente per tutti gli attori coinvolti (implementatori, verificatori). | Media |

## 3. Architettura Proposta

Il flusso di lavoro sarà il seguente:
1.  **Generazione Report (Non Firmato):** Lo script `generate_compliance_report.py` viene modificato per produrre un `unsigned_report.json` contenente i risultati e i metadati di certificazione (hash dei commit, timestamp, ecc.).
2.  **Firma (Implementatore SDK):** L'implementatore usa un nuovo script `util_scripts/sign_report.py`, la propria chiave privata e il proprio `providerId` per firmare il report non firmato, producendo un `signed_compliance_report.json`.
3.  **Registro Pubblico:** Un file `compliance_registry.json` nel repository del TCK mappa i `providerId` alle rispettive chiavi pubbliche. Questo file viene aggiornato tramite un processo di revisione via Pull Request.
4.  **Verifica (Pubblica):** Chiunque può usare un nuovo script `util_scripts/verify_report.py` per verificare un report firmato. Lo script usa il `providerId` nel report per trovare la chiave pubblica corretta nel registro e validare la firma.

## 4. Piano di Sviluppo per Fasi

Questo piano è destinato a uno sviluppatore junior. Per favore, segui le fasi in ordine, esegui commit frequenti per ogni piccolo passo completato e assicurati che tutti i test passino prima di ogni push.

**Tracciamento:** Tieni traccia del tuo lavoro aggiornando il file `WORKING_PROGRESS.md` che creerai nella Fase 1.

---

### Fase 1: Setup Iniziale e Infrastruttura del Registro

**Obiettivo:** Creare le fondamenta del nuovo sistema.

| Task ID | Descrizione | Note |
|:---|:---|:---|
| T-1.1 | Crea un nuovo file `WORKING_PROGRESS.md` alla radice del progetto per documentare i tuoi progressi. | Aggiungi una checklist delle fasi e spunta le voci man mano che le completi. |
| T-1.2 | Crea il file `compliance_registry.json` alla radice del progetto. | Inserisci una struttura JSON di esempio con un array vuoto `sdkProviders`, come discusso nell'architettura. |
| T-1.3 | Aggiungi la libreria `cryptography` al file `requirements.txt`. | Esegui `pip install cryptography` e aggiorna il file con la versione corretta. |
| T-1.4 | Crea uno script di utility `util_scripts/generate_keys.py` per generare coppie di chiavi (privata/pubblica). | Lo script dovrebbe salvare `private_key.pem` e `public_key.pem`. Aggiungi `*.pem` al `.gitignore`. |

---

### Fase 2: Modifica della Generazione del Report

**Obiettivo:** Adattare lo script esistente per produrre report non firmati.

| Task ID | Descrizione | Note |
|:---|:---|:---|
| T-2.1 | Identifica e analizza l'attuale script di generazione dei report (probabilmente `util_scripts/generate_compliance_report.py`). | Comprendi come raccoglie i dati e formatta l'output. |
| T-2.2 | Modifica lo script per raccogliere i metadati necessari: commit hash del TCK, commit hash del SUT, e timestamp. | Usa `git rev-parse HEAD` in Python tramite `subprocess` per ottenere gli hash. |
| T-2.3 | Modifica lo script affinché produca come output un file `unsigned_report.json`. | L'output deve contenere le sezioni `reportPayload` e `certificationData` con tutti i dati raccolti. Rimuovi qualsiasi logica di firma preesistente. |

---

### Fase 3: Creazione dello Strumento di Firma

**Obiettivo:** Fornire agli implementatori uno script per firmare i loro report.

| Task ID | Descrizione | Note |
|:---|:---|:---|
| T-3.1 | Crea un nuovo file `util_scripts/sign_report.py`. | Lo script deve accettare argomenti da riga di comando: `--in-file`, `--out-file`, `--key-file`, `--provider-id`. |
| T-3.2 | Implementa la logica per caricare il report non firmato e la chiave privata. | Usa la libreria `cryptography`. |
| T-3.3 | Implementa la logica di firma. Serializza i dati in modo canonico, calcola l'hash (SHA-256) e firma l'hash. | Assicurati che la serializzazione sia deterministica (es. chiavi ordinate, no spazi). |
| T-3.4 | Implementa la logica per salvare il `signed_compliance_report.json` finale. | Il file deve contenere i dati originali più la nuova sezione `signature` con `providerId` e il valore della firma. |

---

### Fase 4: Creazione dello Strumento di Verifica

**Obiettivo:** Permettere a chiunque di verificare un report firmato.

| Task ID | Descrizione | Note |
|:---|:---|:---|
| T-4.1 | Crea un nuovo file `util_scripts/verify_report.py`. | Lo script deve accettare come argomento il percorso del report firmato da verificare. |
| T-4.2 | Implementa la logica per caricare il report firmato e il `compliance_registry.json`. | |
| T-4.3 | Implementa la logica di verifica: estrai il `providerId`, trova la chiave pubblica corrispondente nel registro e verifica la firma. | Usa la stessa logica di serializzazione e hashing dello script di firma. |
| T-4.4 | Stampa un output chiaro e inequivocabile per l'utente (SUCCESSO/ERRORE). | In caso di successo, mostra i dati certificati (commit hash, provider, ecc.). |

---

### Fase 5: Test e Documentazione

**Obiettivo:** Garantire l'affidabilità e l'usabilità del sistema.

| Task ID | Descrizione | Note |
|:---|:---|:---|
| T-5.1 | Crea test di integrazione per il flusso di firma e verifica. | I test dovrebbero coprire il caso di successo, una firma manomessa e un provider non registrato. |
| T-5.2 | Aggiorna il `README.md` o un documento in `docs/` per spiegare il nuovo processo. | Descrivi i passaggi per implementatori e verificatori. |
| T-5.3 | Rivedi tutto il codice, aggiungi commenti dove necessario e assicurati che rispetti le convenzioni del progetto. | La pulizia del codice è importante. |

## 5. Timeline Stimata

Si stima che questo piano richieda circa **3-5 giorni di lavoro** per uno sviluppatore junior, a seconda della sua familiarità con la codebase e con i concetti di crittografia.
