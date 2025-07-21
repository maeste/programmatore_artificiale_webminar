# Piano di Sviluppo: Aggiornamento TCK per Specifica A2A v0.2.6

## 1. Introduzione

Questo documento delinea il piano di sviluppo per l'aggiornamento del Test Compatibility Kit (TCK) della specifica A2A alla versione `v0.2.6`. La versione `v0.2.6` introduce modifiche significative, in particolare la formalizzazione del supporto per trasporti alternativi (gRPC e HTTP+JSON) e l'introduzione di un nuovo metodo RPC (`tasks/list`).

L'obiettivo di questo piano è fornire istruzioni chiare e incrementali per un programmatore Junior, garantendo che il lavoro proceda per fasi ben definite e che ogni checkpoint sia verificabile. Il programmatore dovrà mantenere un file `plan-wip.md` per tracciare l'avanzamento.

## 2. Obiettivi

L'obiettivo principale è estendere la copertura del TCK per includere i nuovi requisiti introdotti nella specifica A2A `v0.2.6`, garantendo la compatibilità con le implementazioni che utilizzano i nuovi trasporti e il nuovo metodo.

In particolare:
*   Aggiungere test per i metodi RPC esistenti tramite i trasporti gRPC e HTTP+JSON.
*   Aggiungere test per il nuovo metodo `tasks/list` tramite gRPC e REST.
*   Aggiornare i test esistenti per riflettere la modifica di `MessageSendConfiguration.acceptedOutputModes`.

## 3. Prerequisiti

Il programmatore Junior deve avere familiarità con:
*   Python e il framework `pytest`.
*   Concetti di base di JSON-RPC, gRPC e HTTP/REST.
*   Utilizzo di Git per il controllo di versione.

## 4. Fasi del Piano

Il lavoro sarà suddiviso nelle seguenti fasi incrementali:

### Fase 0: Setup Iniziale e Comprensione della Specifica

**Obiettivo:** Preparare l'ambiente di lavoro e acquisire una comprensione approfondita delle modifiche introdotte nella `v0.2.6`.

**Istruzioni:**
1.  **Creazione del file `plan-wip.md`:** Crea un nuovo file chiamato `plan-wip.md` nella root del progetto. Questo file servirà per annotare l'avanzamento, le decisioni prese e eventuali blocchi.
2.  **Clonazione del repository della specifica:** Clona il repository della specifica A2A e fai il checkout del tag `v0.2.6` in una directory temporanea. Questo ti permetterà di consultare la specifica aggiornata durante lo sviluppo.
    ```bash
    git clone https://github.com/a2aproject/A2A /tmp/a2a_spec_v026
    cd /tmp/a2a_spec_v026
    git checkout v0.2.6
    ```
3.  **Revisione delle modifiche:** Rileggi attentamente il file `docs/specification.md` e `types/src/types.ts` all'interno della directory clonata (`/tmp/a2a_spec_v026`). Presta particolare attenzione alle sezioni relative ai trasporti (Sezione 3 e 7) e alla definizione di `AgentCard` e `MessageSendConfiguration`.
4.  **Aggiornamento `plan-wip.md`:** Annota nel file `plan-wip.md` le tue prime impressioni e le aree che ritieni più complesse o che richiedono maggiore attenzione.

**Checkpoint:** Repository della specifica clonata e file `plan-wip.md` creato con le prime annotazioni.

**Commit:**
```bash
git add plan-wip.md
git commit -m "feat: Inizio sviluppo TCK v0.2.6 - Setup iniziale e clonazione specifica"
```

### Fase 1: Adattamento dei Test Esistenti per `MessageSendConfiguration`

**Obiettivo:** Modificare i test esistenti per gestire il campo `acceptedOutputModes` che è diventato opzionale in `MessageSendConfiguration`.

**Istruzioni:**
1.  **Identificazione dei test:** Cerca nel codebase del TCK tutti i test che costruiscono o validano oggetti `MessageSendConfiguration` o che interagiscono con il campo `acceptedOutputModes`.
2.  **Analisi dell'impatto:** Per ogni test identificato, valuta se la modifica da obbligatorio a opzionale influisce sulla logica del test. Potrebbe essere necessario:
    *   Rimuovere l'assunzione che il campo sia sempre presente.
    *   Aggiungere casi di test in cui `acceptedOutputModes` è assente.
    *   Assicurarsi che la validazione dei payload gestisca correttamente l'opzionalità.
3.  **Implementazione delle modifiche:** Applica le modifiche necessarie ai test. Assicurati che i test continuino a passare dopo le modifiche.
4.  **Esecuzione dei test:** Esegui i test modificati e tutti i test di regressione pertinenti per assicurarti di non aver introdotto nuove problematiche.
5.  **Aggiornamento `plan-wip.md`:** Documenta le modifiche apportate e i risultati dei test.

**Checkpoint:** Tutti i test relativi a `MessageSendConfiguration` sono stati adattati e passano correttamente.

**Commit:**
```bash
git add <file_modificati>
git commit -m "refactor(tck): Adattato test per MessageSendConfiguration.acceptedOutputModes opzionale"
```

### Fase 2: Implementazione del Supporto per il Trasporto HTTP+JSON

**Obiettivo:** Aggiungere la capacità al TCK di testare agenti che espongono un'interfaccia HTTP+JSON.

**Istruzioni:**
1.  **Analisi dell'architettura del TCK:** Comprendi come il TCK attuale gestisce le interazioni con gli agenti (es. `sut_client.py`). Identifica i punti in cui è possibile estendere il supporto per un nuovo trasporto.
2.  **Definizione del client HTTP+JSON:** Crea un nuovo modulo o estendi un modulo esistente per implementare un client HTTP+JSON che possa inviare richieste e ricevere risposte secondo la specifica A2A per questo trasporto. Presta attenzione ai percorsi (`/v1/message:send`, `/v1/tasks`, ecc.) e ai formati di payload (JSON).
3.  **Creazione di test di base:** Inizia con un piccolo set di test per i metodi RPC fondamentali (`message/send`, `tasks/get`) utilizzando il nuovo client HTTP+JSON. Questi test dovrebbero verificare la corretta comunicazione e la validazione dei payload.
4.  **Integrazione nel framework di test:** Integra il nuovo client nel framework di test del TCK, in modo che i test possano essere eseguiti specificando il trasporto HTTP+JSON.
5.  **Aggiornamento `plan-wip.md`:** Documenta l'implementazione del client HTTP+JSON e i risultati dei test iniziali.

**Checkpoint:** Client HTTP+JSON implementato e test di base per `message/send` e `tasks/get` passano correttamente.

**Commit:**
```bash
git add <nuovi_file_e_modificati>
git commit -m "feat(tck): Aggiunto supporto base per trasporto HTTP+JSON"
```

### Fase 3: Implementazione del Supporto per il Trasporto gRPC

**Obiettivo:** Aggiungere la capacità al TCK di testare agenti che espongono un'interfaccia gRPC.

**Istruzioni:**
1.  **Generazione del codice gRPC:** Utilizza i file `.proto` della specifica A2A (`/tmp/a2a_spec_v026/specification/grpc/a2a.proto`) per generare il codice Python client gRPC. Assicurati di utilizzare la versione corretta di `protoc` e dei plugin Python.
2.  **Definizione del client gRPC:** Crea un nuovo modulo o estendi un modulo esistente per implementare un client gRPC che possa inviare richieste e ricevere risposte secondo la specifica A2A per questo trasporto. Presta attenzione ai nomi dei servizi e dei metodi definiti nel `.proto`.
3.  **Creazione di test di base:** Inizia con un piccolo set di test per i metodi RPC fondamentali (`SendMessage`, `GetTask`) utilizzando il nuovo client gRPC. Questi test dovrebbero verificare la corretta comunicazione e la validazione dei payload (Protobuf).
4.  **Integrazione nel framework di test:** Integra il nuovo client nel framework di test del TCK, in modo che i test possano essere eseguiti specificando il trasporto gRPC.
5.  **Aggiornamento `plan-wip.md`:** Documenta l'implementazione del client gRPC e i risultati dei test iniziali.

**Checkpoint:** Client gRPC implementato e test di base per `SendMessage` e `GetTask` passano correttamente.

**Commit:**
```bash
git add <nuovi_file_e_modificati>
git commit -m "feat(tck): Aggiunto supporto base per trasporto gRPC"
```

### Fase 4: Estensione della Copertura dei Test per i Nuovi Trasporti

**Obiettivo:** Estendere la copertura dei test per tutti i metodi RPC supportati da gRPC e HTTP+JSON.

**Istruzioni:**
1.  **Mappatura dei metodi:** Per ogni metodo RPC definito nella Sezione 7 di `docs/specification.md`, identifica se è supportato da gRPC e/o REST (HTTP+JSON).
2.  **Scrittura dei test:** Scrivi test specifici per ogni metodo RPC e per ogni trasporto supportato. Questi test dovrebbero coprire:
    *   Casi di successo (richieste valide, risposte attese).
    *   Casi di errore (parametri non validi, stati di errore, ecc.).
    *   Test di streaming (`message/stream`, `tasks/resubscribe`) per gRPC e HTTP+JSON, se applicabile.
3.  **Riuso del codice e guida dai test esistenti:** I test esistenti per il trasporto JSON-RPC rappresentano una guida preziosa. Analizzali per comprendere la logica di test, la struttura delle asserzioni e i casi d'uso coperti. Riusa il più possibile il codice e la logica di test esistente, adattandola ai nuovi client di trasporto (gRPC e HTTP+JSON).
4.  **Esecuzione e debug:** Esegui i nuovi test e debugga eventuali fallimenti. Assicurati che tutti i test passino.
5.  **Aggiornamento `plan-wip.md`:** Documenta l'avanzamento della copertura dei test per i nuovi trasporti.

**Checkpoint:** Tutti i metodi RPC supportati da gRPC e HTTP+JSON hanno test dedicati e passano correttamente.

**Commit:**
```bash
git add <nuovi_test_e_modificati>
git commit -m "feat(tck): Estesa copertura test per gRPC e HTTP+JSON"
```

### Fase 5: Implementazione e Test del Metodo `tasks/list`

**Obiettivo:** Aggiungere test per il nuovo metodo `tasks/list`.

**Istruzioni:**
1.  **Implementazione del client:** Aggiungi il supporto per il metodo `tasks/list` nei client gRPC e HTTP+JSON, seguendo le definizioni nella specifica.
2.  **Scrittura dei test:** Scrivi test specifici per `tasks/list` per entrambi i trasporti (gRPC e REST). Questi test dovrebbero verificare:
    *   La corretta invocazione del metodo.
    *   La struttura della risposta (lista di `Task`).
    *   Casi limite (lista vuota, errori).
3.  **Esecuzione e debug:** Esegui i nuovi test e debugga eventuali fallimenti.
4.  **Aggiornamento `plan-wip.md`:** Documenta l'implementazione e i risultati dei test per `tasks/list`.

**Checkpoint:** Metodo `tasks/list` implementato e testato per gRPC e REST, e tutti i test passano.

**Commit:**
```bash
git add <nuovi_file_e_modificati>
git commit -m "feat(tck): Aggiunto e testato metodo tasks/list per gRPC e REST"
```

### Fase 6: Adattamento degli Script di Esecuzione e Generazione Report

**Obiettivo:** Modificare `run_tck.py` per supportare l'esecuzione dei test per i nuovi trasporti e generare report separati per ciascuno.

**Istruzioni:**
1.  **Analisi di `run_tck.py`:** Comprendi come `run_tck.py` esegue attualmente i test e come vengono generati i report. Identifica i punti in cui è possibile aggiungere la logica per selezionare il trasporto e per generare report specifici per trasporto.
2.  **Aggiunta di opzioni per il trasporto:** Modifica `run_tck.py` per accettare un parametro che specifichi il trasporto da testare (es. `--transport jsonrpc`, `--transport grpc`, `--transport http+json`).
3.  **Esecuzione selettiva dei test:** Implementa la logica per eseguire solo i test pertinenti al trasporto selezionato. Potrebbe essere necessario utilizzare i marcatori di `pytest` (es. `@pytest.mark.jsonrpc`, `@pytest.mark.grpc`, `@pytest.mark.http_json`) per etichettare i test in base al trasporto.
4.  **Generazione di report separati:** Modifica la logica di generazione dei report per creare un report distinto per ogni trasporto. Ad esempio, `report_jsonrpc.html`, `report_grpc.html`, `report_http_json.html`.
5.  **Aggiornamento `plan-wip.md`:** Documenta le modifiche apportate agli script di esecuzione e generazione report.

**Checkpoint:** `run_tck.py` è in grado di eseguire test per trasporti specifici e generare report separati.

**Commit:**
```bash
git add run_tck.py <altri_file_modificati>
git commit -m "feat(tck): Aggiunto supporto multi-trasporto a run_tck.py e generazione report"
```

### Fase 7: Pulizia e Documentazione Finale

**Obiettivo:** Finalizzare il lavoro, pulire il codice e aggiornare la documentazione.

**Istruzioni:**
1.  **Revisione del codice:** Rivedi tutto il codice aggiunto e modificato per garantire che sia pulito, leggibile e segua le convenzioni del progetto.
2.  **Rimozione della directory temporanea:** Elimina la directory temporanea contenente la specifica A2A (`/tmp/a2a_spec_v026`).
    ```bash
    rm -rf /tmp/a2a_spec_v026
    ```
3.  **Aggiornamento della documentazione del TCK:** Se necessario, aggiorna la documentazione interna del TCK per riflettere il nuovo supporto per i trasporti e i metodi.
4.  **Revisione finale di `plan-wip.md`:** Assicurati che il file `plan-wip.md` sia completo e rifletta accuratamente tutto il lavoro svolto.

**Checkpoint:** Codice pulito, directory temporanea rimossa, documentazione aggiornata.

**Commit:**
```bash
git add <file_modificati>
git commit -m "docs(tck): Finalizzazione e pulizia per TCK v0.2.6"
```

## 5. Considerazioni Aggiuntive

*   **Test di integrazione:** Durante lo sviluppo, è consigliabile eseguire regolarmente l'intera suite di test del TCK per catturare eventuali regressioni.
*   **Comunicazione:** In caso di dubbi o blocchi, non esitare a chiedere chiarimenti al team.
*   **Performance:** Se l'aggiunta di nuovi test dovesse impattare significativamente le performance della suite di test, valuta l'introduzione di marcatori (`pytest.mark`) per consentire l'esecuzione selettiva dei test.

Questo piano fornisce una roadmap chiara. Procedi con attenzione e documenta ogni passo.