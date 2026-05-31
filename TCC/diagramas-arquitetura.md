# Diagramas de Arquitetura — EzTrip

## 1. Visão Macro (System Design)

```mermaid
flowchart TB
    subgraph Usuario["Usuario"]
        Browser["Navegador Web"]
    end

    subgraph FrontendSPA["Frontend"]
        Nginx["NGINX - Reverse Proxy (porta 80)"]
        ReactApp["React SPA (React 19 + TypeScript)"]
    end

    subgraph API["Backend API (.NET 10)"]
        ApiLayer["Camada Api - Controllers + SignalR + Middlewares"]
        AppLayer["Camada Application - CQS + MediatR"]
        DomainLayer["Camada Domain - Entidades + Regras de Negocio"]
        InfraLayer["Camada Infrastructure - Dapper + Email + Cache"]
        Shared["Shared Kernel - Result + Settings"]
        ApiLayer --> AppLayer
        AppLayer --> DomainLayer
        AppLayer --> InfraLayer
        InfraLayer --> DomainLayer
        Shared -.- ApiLayer
        Shared -.- AppLayer
        Shared -.- DomainLayer
        Shared -.- InfraLayer
    end

    subgraph Infra["Infraestrutura (Docker)"]
        PG[("PostgreSQL 18 - Banco")]
        Valkey[("Valkey - Cache + Backplane")]
        MinIO[("MinIO - Storage S3")]
        MailPit[("MailPit - SMTP Dev")]
        Aspire[("Aspire Dashboard - Logs + Tracing")]
    end

    subgraph Ext["Sistemas Externos"]
        Gemini("Google Gemini AI")
        Google("Google OAuth")
        Gmail("Gmail / AWS SES")
    end

    Browser -- HTTPS --> Nginx
    Nginx -- arquivos estaticos --> ReactApp
    Nginx -- proxy reverso /api/* --> ApiLayer
    Nginx -- WebSocket Upgrade SignalR --> ApiLayer
    ApiLayer -- SQL Dapper porta 5432 --> PG
    ApiLayer -- TCP porta 6379 --> Valkey
    ApiLayer -- S3 API porta 9000 --> MinIO
    ApiLayer -- SMTP porta 1025 --> MailPit
    ApiLayer -- HTTPS --> Gemini
    ApiLayer -- HTTPS --> Google
    ApiLayer -- OTLP gRPC porta 18889 --> Aspire
    MailPit -.-> Gmail
```

## 2. Fluxo de Requisicao (Pipeline)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant N as Nginx
    participant SPA as Frontend React
    participant API as Controller
    participant MP as MediatR Pipeline
    participant H as Handler
    participant DB as PostgreSQL

    U->>N: GET /trips/123/expenses
    N->>SPA: arquivo estatico index.html
    SPA->>API: GET /api/trips/123/expenses (JWT)
    Note over API: Middleware verifica permissoes
    API->>MP: mediator.Send(query)
    Note over MP: Logging + OpenTelemetry
    MP->>H: Handle(query)
    H->>DB: Dapper SELECT query
    DB-->>H: dados
    H-->>MP: Result Ok
    MP-->>API: Lista de gastos
    API-->>SPA: JSON 200 OK
    Note over SPA: TanStack Query atualiza cache
```

## 3. Fluxo de Escrita com Transacao

```mermaid
flowchart LR
    Form["Formulario React"] --> Ctrl["Controller"]
    Ctrl --> Cmd["Command Handler"]
    Cmd --> TX["Transaction UnitOfWork"]
    TX --> DB[("PostgreSQL")]
    DB --> TX
    TX --> Events["Domain Events"]
    Events --> SignalR["SignalR Hub"] --> WS["WebSocket Push"]
    Events --> Hangfire["Hangfire Job"] --> Email["MailKit Envio"]
    Cmd -.-> OTEL["OpenTelemetry"]
    Ctrl -.-> OTEL
    OTEL -.-> Dashboard["Aspire Dashboard"]
    TX -- erro --> Ctrl
    Ctrl -- erro --> Form
```

## 4. Stack Tecnologica

```mermaid
flowchart LR
    subgraph FrontendTech["Frontend"]
        React["React 19"] --> TS["TypeScript 5"]
        React --> Vite["Vite 7"]
        React --> Router["TanStack Router"]
        React --> Query["TanStack Query"]
        React --> Zustand["Zustand"]
        React --> RHF["React Hook Form"]
        RHF --> Zod["Zod"]
        React --> Axios["Axios"]
        React --> SignalR["@microsoft/signalr"]
        React --> Tailwind["Tailwind CSS 4"]
        React --> i18n["i18next"]
    end

    subgraph BackendTech["Backend"]
        Net[".NET 10"] --> MediatR["MediatR"]
        Net --> Dapper["Dapper"]
        Net --> FM["FluentMigrator"]
        Net --> MailKit["MailKit"]
        Net --> Hangfire["Hangfire"]
        Net --> JWT["JWT Auth"]
        Net --> OTEL["OpenTelemetry"]
        Net --> Serilog["Serilog"]
        Net --> Gemini["Mscc.GenerativeAI"]
        Net --> Redis["StackExchange.Redis"]
        Net --> S3["AWSSDK.S3"]
    end

    subgraph InfraTech["Infraestrutura"]
        Docker["Docker Compose"]
        PG["PostgreSQL 18"]
        Valkey["Valkey Redis"]
        MinIO["MinIO S3"]
        MailPit["MailPit SMTP"]
        Aspire["Aspire Dashboard"]
    end

    BackendTech --> InfraTech
    FrontendTech --> Axios
    FrontendTech --> SignalR
```

---

> **Nota:** Diagramas baseados no codigo-fonte do EzTrip.
