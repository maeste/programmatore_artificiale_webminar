# Piano di Sviluppo: Porting del TCK A2A da v0.2.5 a v0.2.6

## Introduzione

Questo documento fornisce un piano dettagliato per l'upgrade del Technology Compatibility Kit (TCK) A2A dalla versione 0.2.5 alla versione 0.2.6. Il progetto mira ad estendere il TCK esistente per supportare e testare le nuove funzionalità introdotte nella specifica A2A 0.2.6.

### Obiettivi del Progetto

1. **Aggiornare il TCK** per testare sistemi che implementano A2A 0.2.6
2. **Mantenere la retrocompatibilità** con i test esistenti per v0.2.5
3. **Implementare nuovi test** per le funzionalità specifiche di v0.2.6
4. **Adattare gli script di esecuzione** per supportare i nuovi trasporti

### Modifiche Principali Identificate in v0.2.6

Basandosi sull'analisi del repository A2A v0.2.6 e sul file `cambiamenti.txt`:

1. **Formalizzazione del supporto multi-trasporto**: gRPC, HTTP+JSON oltre al JSON-RPC esistente
2. **Introduzione del metodo `tasks/list`**: nuovo endpoint per listare i task
3. **`MessageSendConfiguration.acceptedOutputModes`** diventa opzionale
4. **Miglioramenti al file `.proto`**: aggiunte di `json_name` e commenti

---

## FASE 1: Preparazione e Setup dell'Ambiente
**Durata stimata**: 1-2 giorni

### Task 1.1: Setup Ambiente di Sviluppo
- [ ] Verificare la compatibilità delle dipendenze Python attuali
- [ ] Aggiungere dipendenze per gRPC testing (se necessario):
  - `grpcio`
  - `grpcio-tools`
  - `protobuf`
- [ ] Aggiornare `requirements.txt` o `pyproject.toml`
- [ ] Testare che l'ambiente di build funzioni correttamente

**Checkpoint**: Commit con messaggio "setup: prepare environment for A2A v0.2.6 support"

### Task 1.2: Aggiornamento Documentazione di Base
- [ ] Aggiornare `README.md` con informazioni su v0.2.6
- [ ] Aggiornare `CLAUDE.md` se necessario
- [ ] Documentare i nuovi trasporti supportati

**Output**: `working_progress.md` - Fase 1 completata

---

## FASE 2: Estensione dell'Infrastruttura di Test Multi-Trasporto
**Durata stimata**: 4-5 giorni

> **NOTA IMPORTANTE PER LO SVILUPPATORE**:
> I test attualmente implementati in `tests/` sono tutti basati sul trasporto JSON-RPC e utilizzano la classe `SUTClient` che fa chiamate HTTP POST con payload JSON-RPC. Nel porting agli altri trasporti (gRPC e HTTP+JSON), la logica implementata in questi test (verifica di casi corretti, gestione errori, corner cases) **deve fare da guida** anche per i nuovi test. 
> 
> Ogni test esistente che passa su JSON-RPC dovrebbe avere un equivalente che passa sui nuovi trasporti, quando applicabile secondo la specifica v0.2.6.

### Task 2.1: Analisi dei Mapping Trasporti dalla Specifica v0.2.6

Prima di implementare, **devi comprendere esattamente come i metodi JSON-RPC si mappano sui nuovi trasporti**:

- [ ] Creare documento `TRANSPORT_MAPPING.md` che elenca per ogni metodo RPC:

**JSON-RPC → gRPC → HTTP+JSON:**
```
message/send:
  JSON-RPC: POST / {"method": "message/send", "params": {...}}
  gRPC: SendMessage(SendMessageRequest) → SendMessageResponse
  HTTP+JSON: POST /v1/message:send {"message": {...}, "configuration": {...}}

message/stream:
  JSON-RPC: POST / {"method": "message/stream", "params": {...}} → SSE
  gRPC: SendStreamingMessage(SendMessageRequest) → stream StreamResponse
  HTTP+JSON: POST /v1/message:stream {...} → SSE

tasks/get:
  JSON-RPC: POST / {"method": "tasks/get", "params": {"id": "...", "historyLength": 5}}
  gRPC: GetTask(GetTaskRequest{name: "tasks/{id}", history_length: 5}) → Task
  HTTP+JSON: GET /v1/tasks/{id}?historyLength=5 → Task

tasks/list:
  JSON-RPC: NON DISPONIBILE
  gRPC: ListTask() → repeated Task
  HTTP+JSON: GET /v1/tasks → [Task]

tasks/cancel:
  JSON-RPC: POST / {"method": "tasks/cancel", "params": {"id": "..."}}
  gRPC: CancelTask(CancelTaskRequest{name: "tasks/{id}"}) → Task
  HTTP+JSON: POST /v1/tasks/{id}:cancel {"name": "tasks/{id}"} → Task

tasks/resubscribe:
  JSON-RPC: POST / {"method": "tasks/resubscribe", "params": {"id": "..."}} → SSE
  gRPC: TaskSubscription(TaskSubscriptionRequest{name: "tasks/{id}"}) → stream StreamResponse
  HTTP+JSON: POST /v1/tasks/{id}:subscribe {"name": "tasks/{id}"} → SSE
```

### Task 2.2: Creazione dell'Infrastruttura Multi-Trasporto

- [ ] Creare struttura directory:
```
tests/
├── transport/
│   ├── __init__.py
│   ├── conftest.py              # Configurazione trasporti
│   ├── base_client.py           # Interface comune per tutti i client
│   ├── jsonrpc/
│   │   ├── __init__.py
│   │   └── jsonrpc_client.py    # Wrapper per SUTClient esistente
│   ├── grpc/
│   │   ├── __init__.py
│   │   ├── grpc_client.py       # Client gRPC
│   │   └── generated/           # Stub protobuf
│   └── http_json/
│       ├── __init__.py
│       └── rest_client.py       # Client REST
```

- [ ] Implementare `base_client.py` con interface comune:
```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional, AsyncGenerator

class BaseTransportClient(ABC):
    @abstractmethod
    def send_message(self, message: Dict[str, Any], configuration: Optional[Dict] = None) -> Dict[str, Any]:
        """Implementa message/send per il trasporto specifico"""
        pass
    
    @abstractmethod
    async def stream_message(self, message: Dict[str, Any], configuration: Optional[Dict] = None) -> AsyncGenerator[Dict[str, Any], None]:
        """Implementa message/stream per il trasporto specifico"""
        pass
    
    @abstractmethod
    def get_task(self, task_id: str, history_length: Optional[int] = None) -> Dict[str, Any]:
        """Implementa tasks/get per il trasporto specifico"""
        pass
    
    @abstractmethod
    def list_tasks(self) -> List[Dict[str, Any]]:
        """Implementa tasks/list per il trasporto specifico (raise NotImplementedError per JSON-RPC)"""
        pass
    
    @abstractmethod
    def cancel_task(self, task_id: str) -> Dict[str, Any]:
        """Implementa tasks/cancel per il trasporto specifico"""
        pass
```

### Task 2.3: Implementazione Client JSON-RPC (Wrapper Esistente)

- [ ] Creare `tests/transport/jsonrpc/jsonrpc_client.py`:
```python
from tests.transport.base_client import BaseTransportClient
from tck.sut_client import SUTClient
from tck import message_utils

class JSONRPCClient(BaseTransportClient):
    def __init__(self, base_url: str):
        self.sut_client = SUTClient(base_url)
    
    def send_message(self, message: Dict[str, Any], configuration: Optional[Dict] = None) -> Dict[str, Any]:
        params = {"message": message}
        if configuration:
            params["configuration"] = configuration
        
        response = self.sut_client.send_json_rpc("message/send", params=params)
        # Gestire sia successo che errore
        if "error" in response:
            # Convertire errore JSON-RPC in eccezione standardizzata
            raise TransportError(response["error"])
        return response["result"]
    
    def list_tasks(self) -> List[Dict[str, Any]]:
        raise NotImplementedError("tasks/list not available in JSON-RPC transport")
```

### Task 2.4: Implementazione Client gRPC

- [ ] Generare stub gRPC:
```bash
# Dalla directory del progetto
python -m grpc_tools.protoc \
  --proto_path=temp_a2a_026/specification/grpc/ \
  --python_out=tests/transport/grpc/generated/ \
  --grpc_python_out=tests/transport/grpc/generated/ \
  temp_a2a_026/specification/grpc/a2a.proto
```

- [ ] Creare `tests/transport/grpc/grpc_client.py`:
```python
import grpc
from tests.transport.base_client import BaseTransportClient
from tests.transport.grpc.generated import a2a_pb2, a2a_pb2_grpc

class GRPCClient(BaseTransportClient):
    def __init__(self, base_url: str):
        # Estrarre host:port da base_url
        self.channel = grpc.insecure_channel(self._extract_grpc_endpoint(base_url))
        self.stub = a2a_pb2_grpc.A2AServiceStub(self.channel)
    
    def send_message(self, message: Dict[str, Any], configuration: Optional[Dict] = None) -> Dict[str, Any]:
        # Convertire da dict Python a protobuf
        pb_message = self._dict_to_protobuf_message(message)
        pb_config = self._dict_to_protobuf_config(configuration) if configuration else None
        
        request = a2a_pb2.SendMessageRequest(
            msg=pb_message,
            configuration=pb_config
        )
        
        try:
            response = self.stub.SendMessage(request)
            # Convertire da protobuf a dict Python
            return self._protobuf_to_dict(response)
        except grpc.RpcError as e:
            # Convertire errore gRPC in TransportError standardizzato
            raise TransportError({
                "code": e.code().value[0],
                "message": e.details()
            })
    
    def list_tasks(self) -> List[Dict[str, Any]]:
        # gRPC supporta tasks/list
        request = a2a_pb2.ListTaskRequest()
        response = self.stub.ListTask(request)
        return [self._protobuf_to_dict(task) for task in response.tasks]
```

### Task 2.5: Implementazione Client HTTP+JSON

- [ ] Creare `tests/transport/http_json/rest_client.py`:
```python
import requests
from tests.transport.base_client import BaseTransportClient

class HTTPJSONClient(BaseTransportClient):
    def __init__(self, base_url: str):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
    
    def send_message(self, message: Dict[str, Any], configuration: Optional[Dict] = None) -> Dict[str, Any]:
        payload = {"message": message}
        if configuration:
            payload["configuration"] = configuration
        
        response = self.session.post(
            f"{self.base_url}/v1/message:send",
            json=payload,
            headers={"Content-Type": "application/json"}
        )
        
        if response.status_code != 200:
            raise TransportError({
                "code": response.status_code,
                "message": f"HTTP {response.status_code}: {response.text}"
            })
        
        data = response.json()
        # HTTP+JSON restituisce {"task": {...}} o {"message": {...}}
        return data.get("task") or data.get("message")
    
    def get_task(self, task_id: str, history_length: Optional[int] = None) -> Dict[str, Any]:
        url = f"{self.base_url}/v1/tasks/{task_id}"
        params = {}
        if history_length is not None:
            params["historyLength"] = history_length
        
        response = self.session.get(url, params=params)
        # Gestire errori HTTP appropriatamente
        return response.json()
    
    def list_tasks(self) -> List[Dict[str, Any]]:
        response = self.session.get(f"{self.base_url}/v1/tasks")
        return response.json()  # Dovrebbe essere una lista
```

### Task 2.6: Aggiornamento conftest.py per Multi-Trasporto

- [ ] Modificare `tests/conftest.py` principale:
```python
def pytest_addoption(parser):
    parser.addoption("--sut-url", action="store", default=None)
    parser.addoption("--transport", action="store", default="jsonrpc", 
                    choices=["jsonrpc", "grpc", "http_json", "all"])
    # ... altri parametri esistenti

@pytest.fixture(scope="session")
def transport_type(request):
    return request.config.getoption("--transport")

@pytest.fixture(scope="session") 
def client_factory(transport_type):
    def _create_client(sut_url: str, transport: str = None):
        transport = transport or transport_type
        if transport == "jsonrpc":
            return JSONRPCClient(sut_url)
        elif transport == "grpc":
            return GRPCClient(sut_url)
        elif transport == "http_json":
            return HTTPJSONClient(sut_url)
        else:
            raise ValueError(f"Unsupported transport: {transport}")
    return _create_client

@pytest.fixture
def transport_client(client_factory, request):
    sut_url = request.config.getoption("--sut-url")
    return client_factory(sut_url)
```

### Task 2.7: Creazione Test Parametrici per Trasporti

- [ ] Creare `tests/transport/test_transport_compatibility.py`:
```python
import pytest
from tests.markers import mandatory_protocol

# Test che tutti i trasporti dichiarati funzionino
@pytest.mark.parametrize("transport", ["jsonrpc", "grpc", "http_json"])
def test_message_send_basic_all_transports(client_factory, transport, agent_card_data):
    """
    Verifica che message/send funzioni correttamente su tutti i trasporti supportati.
    
    Questo test replica la logica di test_message_send_valid_text ma per ogni trasporto.
    """
    # Skip se il trasporto non è supportato dal SUT
    if not is_transport_supported(agent_card_data, transport):
        pytest.skip(f"Transport {transport} not supported by SUT")
    
    client = client_factory(config.get_sut_url(), transport)
    
    message = {
        "kind": "message",
        "messageId": f"test-{transport}-" + str(uuid.uuid4()),
        "role": "user",
        "parts": [{"kind": "text", "text": f"Hello from {transport} transport!"}]
    }
    
    try:
        result = client.send_message(message)
        
        # La logica di validazione deve essere la stessa per tutti i trasporti
        # ma adattata alla struttura di risposta specifica
        assert_valid_message_send_response(result, transport)
        
    except TransportError as e:
        pytest.fail(f"Transport {transport} failed: {e}")
```

**Checkpoint**: Commit con messaggio "feat: implement multi-transport infrastructure with JSON-RPC, gRPC, and HTTP+JSON clients"

**Output**: `working_progress.md` - Fase 2 completata

---

## FASE 3: Implementazione Test per `tasks/list`
**Durata stimata**: 2-3 giorni

> **RIFERIMENTI PER LO SVILUPPATORE**:
> Basati sui pattern implementati in `test_tasks_get_method.py` per la struttura dei test, la gestione degli errori e la validazione delle risposte. `tasks/list` è disponibile **SOLO** per gRPC e HTTP+JSON, NON per JSON-RPC.

### Task 3.1: Test di Base per `tasks/list`

- [ ] Creare `tests/mandatory/protocol/test_tasks_list.py` seguendo il pattern di `test_tasks_get_method.py`:

```python
import pytest
import uuid
from tests.markers import mandatory_protocol
from tests.transport.base_client import TransportError

@pytest.fixture(scope="module") 
def transport_client_for_list(client_factory, request):
    """
    Fixture specifica per tasks/list che skippa automaticamente 
    se il trasporto non supporta questo metodo.
    """
    transport = request.config.getoption("--transport")
    if transport == "jsonrpc":
        pytest.skip("tasks/list not available in JSON-RPC transport")
    
    sut_url = request.config.getoption("--sut-url")
    return client_factory(sut_url, transport)

@mandatory_protocol
def test_tasks_list_empty(transport_client_for_list):
    """
    MANDATORY: A2A Specification v0.2.6 - Task List (gRPC/HTTP+JSON only)
    
    Il metodo tasks/list DEVE restituire una lista vuota quando 
    non ci sono task attivi, seguendo la specifica v0.2.6.
    
    Failure Impact: Implementation is not A2A v0.2.6 compliant
    """
    try:
        result = transport_client_for_list.list_tasks()
        assert isinstance(result, list), "tasks/list must return a list"
        # Lista può essere vuota all'inizio
        assert len(result) >= 0, "tasks/list returned invalid list"
        
    except TransportError as e:
        # Se il metodo non esiste, questo è un errore di compliance
        if e.code == -32601:  # Method not found
            pytest.fail("tasks/list method not implemented but required for this transport")
        raise

@mandatory_protocol 
def test_tasks_list_with_existing_task(transport_client_for_list):
    """
    MANDATORY: A2A Specification v0.2.6 - Task List with Content
    
    Quando esistono task attivi, tasks/list DEVE includerli nella risposta.
    Segue lo stesso pattern di validazione di test_tasks_get_valid.
    
    Failure Impact: Implementation is not A2A v0.2.6 compliant  
    """
    # Prima creare un task usando message/send
    message = {
        "kind": "message", 
        "messageId": "test-list-task-" + str(uuid.uuid4()),
        "role": "user",
        "parts": [{"kind": "text", "text": "Task for list test"}]
    }
    
    # Creare task che potrebbe non completarsi immediatamente
    task_result = transport_client_for_list.send_message(message)
    
    # Ora ottenere la lista dei task
    task_list = transport_client_for_list.list_tasks()
    assert isinstance(task_list, list), "tasks/list must return a list"
    
    # Se abbiamo creato un task, dovrebbe apparire nella lista
    # (a meno che non si sia completato immediatamente)
    task_ids = [task.get("id") for task in task_list if isinstance(task, dict)]
    
    # Validare che ogni task nella lista abbia la struttura corretta
    for task in task_list:
        assert isinstance(task, dict), "Each task in list must be an object"
        assert "id" in task, "Each task must have an id field"
        assert "status" in task, "Each task must have a status field"
        assert isinstance(task["status"], dict), "Task status must be an object"
        assert "state" in task["status"], "Task status must have a state field"
```

### Task 3.2: Test di Gestione Errori per `tasks/list`

- [ ] Aggiungere test di error handling seguendo il pattern di `test_tasks_get_nonexistent.py`:

```python
@mandatory_protocol
def test_tasks_list_method_available_only_for_supported_transports(client_factory, request):
    """
    MANDATORY: A2A Specification v0.2.6 - Transport Compatibility
    
    tasks/list DEVE essere disponibile solo per gRPC e HTTP+JSON.
    JSON-RPC NON deve supportare questo metodo.
    
    Failure Impact: Transport implementation violates specification
    """
    transport = request.config.getoption("--transport") 
    sut_url = request.config.getoption("--sut-url")
    
    if transport == "jsonrpc":
        # JSON-RPC non deve supportare tasks/list
        jsonrpc_client = client_factory(sut_url, "jsonrpc")
        
        with pytest.raises(NotImplementedError):
            jsonrpc_client.list_tasks()
            
    else:
        # gRPC e HTTP+JSON devono supportarlo
        client = client_factory(sut_url, transport)
        try:
            result = client.list_tasks()
            assert isinstance(result, list)
        except TransportError as e:
            if e.code == -32601:  # Method not found
                pytest.fail(f"tasks/list required for {transport} transport but not implemented")
            raise
```

### Task 3.3: Test Multi-Trasporto per `tasks/list`

- [ ] Creare test parametrici che escludano JSON-RPC:

```python
@pytest.mark.parametrize("transport", ["grpc", "http_json"])  # Escluso jsonrpc
def test_tasks_list_cross_transport_consistency(client_factory, transport, agent_card_data):
    """
    Verifica che tasks/list restituisca risultati consistenti 
    tra gRPC e HTTP+JSON per lo stesso SUT.
    """
    if not is_transport_supported(agent_card_data, transport):
        pytest.skip(f"Transport {transport} not supported by SUT")
    
    client = client_factory(config.get_sut_url(), transport)
    
    # Creare alcuni task per testare
    tasks_created = []
    for i in range(2):
        message = {
            "kind": "message",
            "messageId": f"test-cross-transport-{i}-" + str(uuid.uuid4()),
            "role": "user", 
            "parts": [{"kind": "text", "text": f"Cross transport test {i}"}]
        }
        task = client.send_message(message)
        if isinstance(task, dict) and "id" in task:
            tasks_created.append(task["id"])
    
    # Ottenere lista task
    task_list = client.list_tasks()
    
    # Validazioni di base che devono essere consistenti tra trasporti
    assert isinstance(task_list, list)
    
    for task in task_list:
        # Ogni task deve avere struttura A2A standard
        assert isinstance(task, dict)
        assert "id" in task
        assert "status" in task
        
        # Il formato del status deve essere consistente
        status = task["status"]
        assert isinstance(status, dict)
        assert "state" in status
        assert status["state"] in [
            "submitted", "working", "input_required", 
            "completed", "failed", "canceled"
        ]
```

### Task 3.4: Integrazione con Test Runner Esistente

- [ ] Aggiornare marker per `tasks/list`:

```python
# In tests/markers.py - aggiungere se necessario
mandatory_protocol_v026 = pytest.mark.mandatory_protocol_v026
```

- [ ] Assicurarsi che `run_tck.py` includa i nuovi test quando eseguito con `--transport grpc` o `--transport http_json`

- [ ] Verificare che i nuovi test siano esclusi correttamente quando eseguito con `--transport jsonrpc`

**Checkpoint**: Commit con messaggio "feat: implement tasks/list tests for gRPC and HTTP+JSON transports"

**Output**: `working_progress.md` - Fase 3 completata

---

## FASE 4: Gestione `acceptedOutputModes` Opzionale
**Durata stimata**: 2 giorni

### Task 4.1: Identificazione Test Esistenti
- [ ] Audit dei test esistenti che utilizzano `MessageSendConfiguration`
- [ ] Identificare test che assumono `acceptedOutputModes` come obbligatorio

### Task 4.2: Aggiornamento Test Esistenti
- [ ] Modificare test per gestire l'opzionalità:
  - Test con `acceptedOutputModes` presente
  - Test con `acceptedOutputModes` assente
  - Test che entrambi i casi funzionino correttamente

### Task 4.3: Nuovi Test per Opzionalità
- [ ] Creare test specifici in `tests/mandatory/protocol/test_message_send_config.py`:
  - Comportamento senza `acceptedOutputModes`
  - Comportamento con `acceptedOutputModes` vuoto
  - Retrocompatibilità con implementazioni che richiedono il campo

**Checkpoint**: Commit con messaggio "feat: handle optional acceptedOutputModes in MessageSendConfiguration"

**Output**: `working_progress.md` - Fase 4 completata

---

## FASE 5: Aggiornamento Script di Esecuzione
**Durata stimata**: 2-3 giorni

### Task 5.1: Modifica `run_tck.py`
- [ ] Aggiungere parametro `--transport` con opzioni:
  - `jsonrpc` (default per retrocompatibilità)
  - `grpc`
  - `http_json`
  - `all` (esegue test su tutti i trasporti supportati)

- [ ] Modificare `run_test_category()` per:
  - Passare parametro transport a pytest
  - Gestire report separati per trasporto
  - Aggregare risultati multi-trasporto

### Task 5.2: Adattamento Report di Compliance
- [ ] Modificare `util_scripts/generate_compliance_report.py`:
  - Supportare risultati multi-trasporto
  - Aggregare metriche di compliance per trasporto
  - Generare sezioni separate nel report per ogni trasporto

### Task 5.3: Nuovi Script di Utilità
- [ ] Creare `util_scripts/detect_transports.py`:
  - Auto-detection dei trasporti supportati dal SUT
  - Analisi dell'Agent Card per `additionalInterfaces`
  - Suggerimenti per la selezione del trasporto ottimale

**Checkpoint**: Commit con messaggio "feat: add multi-transport support to test runner and reporting"

**Output**: `working_progress.md` - Fase 5 completata

---

## FASE 6: Test di Integrazione e Validazione
**Durata stimata**: 2-3 giorni

### Task 6.1: Test End-to-End Multi-Trasporto
- [ ] Configurare SUT di test che supporti tutti i trasporti
- [ ] Eseguire suite completa su ogni trasporto
- [ ] Verificare che tutti i test esistenti continuino a passare

### Task 6.2: Test di Regressione
- [ ] Verificare compatibilità con SUT v0.2.5 esistenti
- [ ] Assicurarsi che test JSON-RPC non siano stati compromessi
- [ ] Validare che i report di compliance siano corretti

### Task 6.3: Performance e Stress Testing
- [ ] Test delle performance con trasporti diversi
- [ ] Test di carico per `tasks/list` con molti task
- [ ] Verifica gestione errori per ogni trasporto

**Checkpoint**: Commit con messaggio "test: comprehensive validation of multi-transport A2A v0.2.6 support"

**Output**: `working_progress.md` - Fase 6 completata

---

## FASE 7: Documentazione e Finalizzazione
**Durata stimata**: 1-2 giorni

### Task 7.1: Aggiornamento Documentazione
- [ ] Aggiornare `README.md` con:
  - Esempi di uso multi-trasporto
  - Nuove opzioni di `run_tck.py`
  - Guida alla configurazione gRPC e HTTP+JSON

- [ ] Creare `docs/TRANSPORT_GUIDE.md`:
  - Guida dettagliata ai trasporti supportati
  - Configurazione di SUT per ogni trasporto
  - Troubleshooting comune

### Task 7.2: Aggiornamento Help e Usage
- [ ] Aggiornare help text in `run_tck.py`
- [ ] Aggiornare esempi negli script di utilità
- [ ] Verificare che tutti i messaggi di errore siano chiari

### Task 7.3: Preparazione Release
- [ ] Aggiornare version numbers se necessario
- [ ] Preparare CHANGELOG con modifiche apportate
- [ ] Verificare che tutti i test passino in CI

**Checkpoint**: Commit finale con messaggio "docs: complete A2A v0.2.6 support documentation"

**Output**: `working_progress.md` - Progetto completato

---

## Struttura Directory Finale

```
tests/
├── mandatory/
│   ├── jsonrpc/              # Test JSON-RPC esistenti
│   └── protocol/             # Test protocollo A2A
│       ├── test_tasks_list.py     # Nuovo
│       └── test_message_send_config.py  # Aggiornato
├── transport/                # Nuovo
│   ├── conftest.py          # Configurazione multi-trasporto
│   ├── jsonrpc/             # Client JSON-RPC esistente
│   ├── grpc/                # Client gRPC - Nuovo
│   │   ├── grpc_client.py
│   │   └── generated/       # Stub protobuf generati
│   └── http_json/           # Client HTTP+JSON - Nuovo
│       └── rest_client.py
├── optional/
│   ├── capabilities/
│   ├── quality/
│   └── features/
└── conftest.py              # Aggiornato per multi-trasporto

util_scripts/
├── generate_compliance_report.py  # Aggiornato
├── detect_transports.py          # Nuovo
└── compliance_levels.py

run_tck.py                    # Aggiornato con --transport
```

---

## Considerazioni Importanti per il Programmatore

### Gestione dell'Opzionalità di `tasks/list`
- `tasks/list` è disponibile solo per gRPC e HTTP+JSON, NON per JSON-RPC
- I test devono gestire questa situazione con skip appropriati
- Utilizzare pytest markers per distinguere i test specifici per trasporto

### Retrocompatibilità
- Tutti i test esistenti devono continuare a funzionare con SUT v0.2.5
- Il default deve rimanere JSON-RPC per non rompere workflow esistenti
- I parametri opzionali devono essere gestiti gracefully

### Testing Multi-Trasporto
- Utilizzare pytest parametrization per eseguire gli stessi test su trasporti diversi
- Gestire le differenze semantiche tra trasporti (es. error codes)
- Implementare skip intelligenti per funzionalità non supportate

### Monitoraggio Progresso
- Aggiornare `working_progress.md` alla fine di ogni fase
- Fare commit frequenti con messaggi chiari
- Testare sempre che i test esistenti non si rompano prima di procedere

### Performance
- I test gRPC potrebbero essere più veloci dei JSON-RPC
- HTTP+JSON potrebbe essere più lento per la serializzazione extra
- Considerare timeout appropriati per ogni trasporto

---

## Criteri di Successo

Il progetto sarà considerato completato con successo quando:

1. ✅ Tutti i test esistenti continuano a passare su SUT v0.2.5
2. ✅ I nuovi test per v0.2.6 passano su SUT v0.2.6 
3. ✅ `run_tck.py --transport all` esegue test su tutti i trasporti
4. ✅ I report di compliance aggregano correttamente risultati multi-trasporto
5. ✅ La documentazione è completa e accurata
6. ✅ Il codice rispetta gli standard di qualità esistenti (linting, formattazione)

---

## Note Tecniche

### Generazione Stub gRPC
```bash
python -m grpc_tools.protoc \
  --proto_path=temp_a2a_026/specification/grpc/ \
  --python_out=tests/transport/grpc/generated/ \
  --grpc_python_out=tests/transport/grpc/generated/ \
  temp_a2a_026/specification/grpc/a2a.proto
```

### Configurazione Pytest per Multi-Trasporto
```python
# In conftest.py
def pytest_addoption(parser):
    parser.addoption("--transport", action="store", default="jsonrpc")

@pytest.fixture
def transport(request):
    return request.config.getoption("--transport")
```

### Esempio Test Parametrico
```python
@pytest.mark.parametrize("transport", ["jsonrpc", "grpc", "http_json"])
def test_message_send_basic(transport, client_factory):
    client = client_factory(transport)
    # Test implementation
```

---

**Ricorda**: Questo è un progetto incrementale. Ogni fase può essere interrotta e ripresa. Mantieni sempre il TCK in uno stato funzionante e testa frequentemente con SUT reali.