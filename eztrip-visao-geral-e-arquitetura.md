# EzTrip: Visão Geral e Arquitetura

## Objetivo

Este documento apresenta o propósito do EzTrip, o domínio principal do sistema, as funcionalidades efetivamente identificadas no código e a arquitetura geral da solução.

## O que é o EzTrip

O EzTrip é uma aplicação web para organização de viagens em grupo. O sistema centraliza o planejamento da viagem e a interação entre participantes em um único ambiente.

## Propósito principal

O sistema foi construído para permitir:

- criar viagens com nome, destino e período
- convidar pessoas para participar
- controlar permissões dos participantes
- aprovar ou rejeitar solicitações de entrada
- acompanhar participantes da viagem
- registrar e dividir gastos
- manter uma lista de bagagem individual por participante dentro da viagem
- receber notificações sobre eventos relevantes da viagem

## Domínio principal identificado no código

- **Trips**: viagem principal cadastrada no sistema
- **Participants**: usuários participantes de uma viagem
- **Permissions**: permissões por módulo para participantes não administradores
- **Invitation Links**: links temporários para ingresso em uma viagem
- **Access Requests**: pedidos de acesso gerados a partir de convites
- **Notifications**: notificações de eventos importantes da viagem
- **Gastos**: domínio financeiro da viagem. Tecnicamente, o código usa nomes como `Expenses`
- **Packing List**: lista de bagagem individual por participante/viagem
- **Accounts / Sessions / Tokens**: autenticação, confirmação de e-mail, recuperação de senha e sessão
- **Media / Uploads**: suporte a mídia, especialmente avatar de usuário

## Funcionalidades implementadas

Com base nas rotas, controllers, serviços e telas, as funcionalidades implementadas e testáveis são:

- cadastro de usuário
- confirmação de e-mail
- reenvio de e-mail de verificação
- login
- logout
- renovação de sessão por refresh token
- recuperação de senha por e-mail
- redefinição de senha
- alteração de idioma do usuário entre português e inglês
- listagem de viagens
- busca de viagens
- criação de viagem com configuração inicial de permissões
- visualização da página principal da viagem
- visualização de participantes
- geração de link de convite
- solicitação de acesso por link
- aprovação e rejeição de solicitações de acesso
- remoção de participante por administrador
- listagem de notificações
- marcação de notificação como lida
- marcação de todas as notificações como lidas
- listagem de gastos
- criação de gasto
- edição de gasto
- exclusão de gasto
- visualização de resumo por categoria e moeda
- lista de bagagem individual por participante
- aplicação de template inicial de bagagem
- criação e remoção de categorias da bagagem
- criação, edição, marcação e remoção de itens da bagagem
- atualização em tempo real via SignalR para eventos de participantes, solicitações de acesso e notificações

## Observação sobre funcionalidades não entregues

O sistema possui módulos de permissão chamados `Events`, `Tasks`, `Memories` e `Polls`. Eles aparecem no domínio e na tela de criação de viagem, mas não foram encontrados endpoints, telas completas ou fluxos funcionais equivalentes no escopo atual. Portanto, estes módulos não devem ser tratados como funcionalidades entregues no TCC.

## Tecnologias utilizadas

### Frontend

- React 19
- TypeScript 5
- Vite 7
- TanStack Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Zustand
- i18next e react-i18next
- SignalR Client (`@microsoft/signalr`)
- Tailwind CSS 4
- Radix UI
- Vaul
- Recharts
- Sonner
- date-fns
- Lucide React
- Vitest
- Testing Library
- Biome
- Nginx no container de publicação

### Backend

- .NET 10 (`net10.0`)
- ASP.NET Core Web API
- MediatR
- arquitetura em camadas com separação entre `Api`, `Application`, `Domain` e `Infrastructure`
- padrão CQS/CQRS leve com `Commands` e `Queries`
- autenticação JWT Bearer
- refresh token via cookie HTTP-only
- SignalR
- Serilog
- OpenTelemetry
- Scalar/OpenAPI para documentação da API
- Dapper
- Dapper.Contrib
- FluentMigrator
- Npgsql
- MailKit
- StackExchange.Redis
- AWS SDK S3 compatível com MinIO
- Argon2 para hash de senha
- Newtonsoft.Json

### Banco de dados e infraestrutura

- PostgreSQL 18
- Valkey
- MinIO
- Docker
- Docker Compose
- MailPit
- Aspire Dashboard

## Estrutura geral do sistema

O sistema está dividido em três frentes:

- `eztrip`: orquestração local com Docker Compose
- `eztrip-frontend`: aplicação web do usuário final
- `eztrip-backend`: API principal e regras de negócio

## Fluxo de alto nível

1. O usuário acessa o frontend em `http://localhost`.
2. O frontend envia requisições HTTP para `/api`, encaminhadas pelo Nginx ao backend.
3. O backend processa comandos e consultas usando MediatR.
4. Os dados são persistidos no PostgreSQL.
5. Cache e mensageria de tempo real usam Valkey.
6. E-mails de confirmação e recuperação são enviados para o MailPit.
7. Arquivos de mídia usam MinIO.
8. Logs, traces e métricas são exportados para o Aspire Dashboard.

## Arquitetura do backend

- **Api**: controllers, middlewares, autenticação, autorização e hub de tempo real
- **Application**: commands, queries, handlers e behaviors
- **Domain**: entidades, eventos de domínio, regras de negócio e value objects
- **Infrastructure**: repositórios, acesso ao banco, cache, e-mail, storage e migrações

## Tempo real

O backend expõe um hub SignalR em `/hubs/user`. O frontend se conecta a esse hub para receber:

- novas notificações
- entrada de participante na viagem
- remoção de participante
- criação de solicitação de acesso
- aceite de solicitação de acesso
- rejeição de solicitação de acesso

## Análise funcional por módulo

### Autenticação e conta

Fontes encontradas:

- `AuthController`
- páginas `/register`, `/login`, `/verify-email`, `/forgot-password`, `/reset-password`
- testes de integração de autenticação

Capacidades identificadas:

- cadastro com nome, e-mail e senha
- validação de senha forte
- confirmação de e-mail por token
- reenvio de e-mail de verificação
- login apenas após verificação do e-mail
- refresh token por cookie
- logout
- fluxo de esqueci minha senha
- redefinição de senha por token

### Idioma do usuário

Fontes encontradas:

- `UsersController`
- `LanguageSettingsDrawer`

Capacidades identificadas:

- alteração do idioma do usuário entre `ptBR` e `enUS`

### Gestão de viagens

Fontes encontradas:

- `TripsController`
- páginas `/trips`, `/trips/create`, `/trips/$tripId`

Capacidades identificadas:

- listar viagens do usuário autenticado
- pesquisar viagens por nome, destino e datas
- criar viagem com nome, destino, período e permissões iniciais
- visualizar resumo da viagem

### Participantes e permissões

Fontes encontradas:

- `ParticipantsController`
- `ParticipantPermissionHandler`
- página `/trips/$tripId/participants`

Capacidades identificadas:

- visualizar participantes da viagem
- consultar permissões do participante autenticado
- diferenciar administrador e membro comum
- remover participante somente se o solicitante for administrador
- controlar permissões por participante para módulos específicos

### Convites e solicitações de acesso

Fontes encontradas:

- `TripInvitationsController`
- `InvitationLinksController`
- `AccessRequestsController`
- página pública `/invitation/$token`

Capacidades identificadas:

- gerar link temporário de convite
- visualizar prévia pública da viagem por link
- solicitar acesso à viagem por link
- aprovar solicitação de acesso
- rejeitar solicitação de acesso

### Notificações

Fontes encontradas:

- `NotificationsController`
- `NotificationEventsHandler`
- `NotificationsDrawer`

Capacidades identificadas:

- listar notificações por viagem
- filtrar por não lidas
- marcar uma notificação como lida
- marcar todas como lidas
- receber notificações em tempo real

Eventos confirmados no backend:

- solicitação de acesso à viagem
- entrada de participante
- remoção de participante

### Gastos

Fontes encontradas:

- `ExpensesController`
- páginas `/trips/$tripId/expenses`, `/expenses/add`, `/expenses/$expenseId/edit`

Capacidades identificadas:

- listar gastos da viagem
- criar gasto
- editar gasto
- excluir gasto
- atribuir participantes ao gasto
- calcular participação por pessoa
- agrupar visualmente por moeda e categoria
- filtrar entre todos os gastos e gastos do usuário

Categorias identificadas na interface:

- Food
- Transport
- Accommodation
- Shopping
- Entertainment
- Other

Moedas identificadas na interface:

- BRL
- USD
- EUR

### Lista de bagagem

Fontes encontradas:

- `PackingListsController`
- página `/trips/$tripId/packing-list`
- testes de frontend do módulo

Capacidades identificadas:

- visualizar a lista de bagagem individual do participante dentro da viagem
- visualizar progresso de bagagem
- aplicar template inicial de categorias
- criar categoria manual
- remover categoria
- adicionar item
- editar título do item
- marcar item como pronto
- remover item
- persistência por operações de patch e controle de versão

Observação importante:

- a lista de bagagem não é compartilhada entre todos os participantes da viagem
- a lista é individual por participante/viagem
