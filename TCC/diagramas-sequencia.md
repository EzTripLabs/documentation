# Diagramas de Sequencia — EzTrip

## UC01 — Criar Viagem

```mermaid
sequenceDiagram
    participant User as Usuario
    participant Form as Formulario React
    participant API as Nginx + Controller
    participant Auth as Auth Middleware
    participant MediatR as MediatR Pipeline
    participant Handler as CreateTripHandler
    participant DB as PostgreSQL
    participant Cache as Valkey

    User->>Form: Preenche nome, destino, datas e permissoes
    Note over Form: Validacao Zod (campos obrigatorios)
    Form->>API: POST /api/trips (JWT no header)
    API->>Auth: Valida token JWT
    Auth-->>API: Usuario autenticado (userId)
    API->>MediatR: mediator.Send(CreateTripCommand)
    Note over MediatR: Logging + OpenTelemetry
    MediatR->>Handler: Handle(command)

    Handler->>Handler: Trip.Create(nome, destino, datas, userId)
    Handler->>Handler: TripPermissions.Create(tripId, permissoes)
    Handler->>Handler: Participant.CreateAsAdmin(tripId, userId)
    Handler->>Handler: ParticipantPermissions.Create(adminId, permissoesPadrao)

    Handler->>DB: BEGIN TRANSACTION
    Handler->>DB: INSERT trip
    Handler->>DB: INSERT trip_permissions
    Handler->>DB: INSERT participant
    Handler->>DB: INSERT participant_permissions
    DB-->>Handler: OK
    Handler->>DB: COMMIT
    Note over Handler: Se erro → ROLLBACK

    Handler->>Handler: Dispara domain events
    Handler-->>MediatR: Result(Guid) sucesso
    MediatR-->>API: TripId
    API-->>Form: 201 Created (tripId)
    Form->>User: Redireciona para pagina da viagem
```

## UC02 — Registrar Gasto com Notificacao em Tempo Real

```mermaid
sequenceDiagram
    participant User as Usuario A
    participant Form as Formulario React
    participant API as Nginx + Controller
    participant Perm as Permission Middleware
    participant MediatR as MediatR Pipeline
    participant Handler as AddExpenseHandler
    participant DB as PostgreSQL
    participant Domain as Expense Entity
    participant Sender as RealTimeEventsHandler
    participant SignalR as SignalR Hub
    participant WS as WebSocket
    participant UserB as Usuario B (navegador)

    User->>Form: Preenche titulo, valor, categoria, participantes
    Note over Form: Validacao: valor > 0, participantes validos
    Form->>API: POST /api/trips/456/expenses (JWT)
    API->>Perm: Verifica permissao "Expenses.Add"
    Perm-->>API: Permitido
    API->>MediatR: mediator.Send(AddExpenseCommand)
    MediatR->>Handler: Handle(command)

    Handler->>DB: SELECT participantes da viagem 456
    DB-->>Handler: Lista de userIds validos
    Note over Handler: Valida se todos os participantes pertencem a viagem

    Handler->>Domain: Expense.Create(titulo, valor, moeda, participantes, ...)
    Domain->>Domain: raise ExpenseCreatedDomainEvent

    Handler->>DB: BEGIN TRANSACTION
    Handler->>DB: INSERT expense
    DB-->>Handler: OK
    Handler->>DB: COMMIT

    Handler->>Domain: GetDomainEvents()
    Handler->>Handler: DispatchDomainEventsAsync()

    Note over Sender: Recebe ExpenseCreatedDomainEvent
    Sender->>DB: SELECT expense completo
    DB-->>Sender: Expense com dados
    Sender->>SignalR: Send ExpenseCreatedMessage

    alt Gasto Privado
        SignalR->>WS: Envia apenas para usuarios envolvidos
    else Gasto Publico
        SignalR->>WS: Envia para todo o grupo da viagem
    end

    WS-->>Form: Atualiza cache (queryClient.setQueryData)
    WS-->>UserB: Notificacao em tempo real
    Form-->>User: 201 Created (expenseId)
    Note over User: Gasto aparece na lista sem recarregar
```

---

> **Legenda:** As setas representam chamadas sincronas (->>) e assincronas (-->>). As notas (Note over) indicam validacoes e processamentos internos importantes.
