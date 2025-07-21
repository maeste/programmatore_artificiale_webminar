# Scaletta webinar – AI‑first developer workflow

*(Durata totale: \~45 min + 5‑10 min Q&A)*

## 1. Opening & Mindset (5′)

- Scherziamo sulla ricerca di MERC su 16 sviluppatori
- Scopo del webinar: **potenziare lo sviluppo "da zero" con l’AI**, non sostituirlo.
- Concetto di *AI‑first developer workflow*: l’AI come **partner**.
- Call to action: *competenza + AI > solo AI*.
- Call to action: competenza sull'AI E sul codice 
  - Capire l'architettura software e saper fare debugging
  - Saper scegliere il modello e il context engineering
  - Creare il software e la documentazione LLM first

## 2. Il PRD/plan come fondamento (7′)

- Perché il **Product Requirements Document** è la mappa per l’agente.
- Struttura raccomandata del PRD (vedi prd\_template.md).
- Integrare **checkpoint Git** e ciclo "agente (junior) → revisione umano".

## 3. Generare il PRD con Chatbot & Gemini CLI (6′)

- Uso di Claude/ChatGPT per architettura e outline. 
- Un template di prompt (vedi prompt\_*template\_*for\_*prd\_*generation.md)
- **Gemini CLI** per brainstorming, explain‑code, refactor interattivo. La possibilità di aggiungere facilmente contesto con comandi @. Usabile anche per vibe coding, ma io lo uso principalmente in fase prd
- Best practice: far spiegare prima il codice sconosciuto. Fornire documentazione di progetto e di piataforma
- L'importanza del deign del progetto sia a livello architetturale che designed-for-llm con script e documentazione chiara.
- L'importanza del TDD

## 4. Cursor IDE: dal PRD al codice (9′)

- Cursor per il code completion
- CTRL-K
- Cursor in modalità ask
- Cursor in modalità agent

### 4.1 Cursor Rules

- File `.cursorrules` e repo **awesome‑cursor‑rules**.
- Benefici: coerenza di stile, standard di team.
- **Documentazione ufficiale Cursor Rules:** [https://docs.cursor.com/context/rules](https://docs.cursor.com/context/rules)

### 4.2 Memory

- Nuova feature "Memory": preferenze salvate per progetto.
- Documentazione ufficiale memory: [https://docs.cursor.com/context/memories](https://docs.cursor.com/context/memories)
- **Confronto e best practice**
- **Cursor Rules (statiche)**: file `.cursor/rules/*.mdc` versionati in Git che contengono linee guida permanenti ➜ stile di codice, architettura, naming, step di build. Tre modalità di trigger (Always / Auto / Manual) più *Agent Requested*; ideali per team governance.
- **Memory (dinamica)**: archivio per‑progetto che si popola in automatico dal  ➜ fatti emersi in chat, comandi preferiti, percorsi file chiave. Introdotta in **Cursor 1.0 (4 giu 2025)**, si abilita da *Settings → Rules → Memories (beta)* e si può editare/flushare.
- **Quando usare cosa**: configuri le Rules all’avvio per fornire «guard‑rail» costanti; lasci che Memory aggiunga contesto operativo man mano che il progetto evolve. Ripulisci la Memory dopo refactor major per evitare drift.
- **Suggerimento slide**: mostra una tabella di confronto (Auto‑update, Scope, Versioning, Autore, Use‑case) + un frammento di Regola (`---
  globs: ["*.py"]
  ...`).

## 5. Agenti autonomi (8′)

- Vantaggi e svantaggi
  - Possibilità di andare i nparallello
  - Task lunghi, multipli e ben separati
  - Comparazione di risultati multi modello, ma anche necessità multi modello
  - Spostare la gestione della complessità in fase di PR review

### 5.1 Cursor Background Agent e Web Agent

- Come si avviano, monitoraggio, branch isolati.
- Costi e Max Mode

### 5.2 Google Jules

- Differenze: scope singolo PR, etichetta `assign-to-jules`, modello Gemini 2.5 Pro.
- Tabella di confronto rapida.

## 6. Vibe Coding vs AI‑assisted su grandi codebase (5′)

- **Vibe Coding rapido**: prompt → app full‑stack giocosa / MVP.
- **Codebase estesa**: digestione codice, regole, agenti in background, checkpoint Git.

## 7. Workflow end‑to‑end consigliato (3′)

1. Planning con chatbot + Gemini CLI → PRD.
2. Commit `plan.md` su branch `planning/...`.
3. Apri Cursor, carica `.cursorrules`, abilita Memory.
4. Lancia Background Agent sull’issue #1; segui diff/PR.
5. Valida, merge, checkpoint Git.
6. Itera task‑per‑task; aggiorna Rules/Memory.

## 8. Risorse & Q&A (2′)

- Repo *awesome‑cursor‑rules* (GitHub: [https://github.com/PatrickJS/awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules))
- Cursor Changelog / Docs Background Agent
- Gemini CLI su GitHub

---

### TODO

-

---

> **Nota:** Questa scaletta è un punto di partenza. Sentiti libero di annotare, tagliare o espandere sezioni direttamente qui nel canvas.

