# EzTrip: Como Rodar com Docker Compose

## Objetivo

Este documento explica como executar o EzTrip localmente usando Docker Compose, quais serviços são iniciados e como acessá-los.

## Requisitos

É necessário ter instalado:

- Docker
- Docker Compose

Observações importantes:

- não são exigidas credenciais especiais para subir o ambiente
- as imagens referenciadas no `docker-compose.prod.yml` são públicas
- o arquivo de compose já contém as variáveis de ambiente necessárias para a execução local

## Passo a passo

### 1. Clonar o repositório

```bash
git clone <URL_DO_REPOSITORIO>
cd eztrip
```

Se o clone gerar a pasta raiz com os três diretórios (`eztrip`, `eztrip-backend` e `eztrip-frontend`), entre na pasta `eztrip`, pois é nela que está o `docker-compose.prod.yml`.

### 2. Baixar as imagens

```bash
docker compose -f docker-compose.prod.yml pull
```

### 3. Subir o ambiente

```bash
docker compose -f docker-compose.prod.yml up -d
```

### 4. Verificar os containers

```bash
docker compose -f docker-compose.prod.yml ps
```

O arquivo analisado não define `healthcheck` explícito para os serviços. Portanto, a validação prática deve ser feita assim:

- confirme se todos os containers estão com status `Up`
- abra as URLs do frontend, da API e do MailPit
- aguarde alguns segundos adicionais na primeira inicialização, pois o backend executa migrações ao iniciar

### 5. Encerrar o ambiente

```bash
docker compose -f docker-compose.prod.yml down
```

Para remover também os volumes persistidos:

```bash
docker compose -f docker-compose.prod.yml down -v
```

## Serviços do Docker Compose

Os serviços abaixo foram identificados diretamente no arquivo `eztrip/docker-compose.prod.yml`.

### Frontend

- **Serviço**: `frontend`
- **Propósito**: interface web principal do sistema
- **Porta**: `80`
- **Acesso**: `http://localhost`
- **Observação**: o Nginx também encaminha chamadas `/api/` para o backend

### Backend API

- **Serviço**: `api`
- **Propósito**: API principal com autenticação, viagens, convites, gastos, bagagem e notificações
- **Porta**: `5000`
- **Acesso direto**: `http://localhost:5000`
- **Documentação da API**:
  - `http://localhost:5000/swagger`
  - `http://localhost:5000/docs`
  - ambos redirecionam para a interface Scalar
- **Observação**: pelo frontend, as chamadas chegam via `http://localhost/api/...`

### Banco de dados

- **Serviço**: `postgres`
- **Propósito**: persistência principal do sistema
- **Porta**: `5432`
- **Banco**: `eztrip`
- **Acesso técnico**: `localhost:5432`
- **Usuário configurado no compose**: `postgres`
- **Senha configurada no compose**: `postgres`

### Cache

- **Serviço**: `valkey`
- **Propósito**: cache e suporte à infraestrutura de tempo real
- **Porta**: `6379`
- **Acesso técnico**: `localhost:6379`
- **Senha configurada no compose**: `password-eztrip-labs`

### Armazenamento de objetos

- **Serviço**: `minio`
- **Propósito**: armazenamento compatível com S3 para arquivos do sistema
- **Portas**:
  - `9000`: API do storage
  - `9001`: painel web do MinIO
- **Acessos**:
  - API: `http://localhost:9000`
  - Console: `http://localhost:9001`
- **Credenciais configuradas no compose**:
  - usuário: `user-eztrip-labs`
  - senha: `password-eztrip-labs`

### Servidor de e-mail de teste

- **Serviço**: `mailpit`
- **Propósito**: capturar e-mails enviados pelo sistema para validação manual
- **Portas**:
  - `1025`: SMTP
  - `8025`: interface web
- **Acessos**:
  - SMTP técnico: `localhost:1025`
  - UI web: `http://localhost:8025`
- **Uso no projeto**:
  - confirmação de e-mail
  - reenvio de confirmação
  - recuperação de senha

### Observabilidade

- **Serviço**: `aspire-dashboard`
- **Propósito**: visualização de logs, traces e métricas
- **Portas**:
  - `18888`: dashboard
  - `18889`: OTLP gRPC
  - `18890`: OTLP HTTP
- **Acesso principal**: `http://localhost:18888`

## Como acessar o Aspire Dashboard

O dashboard exige autenticação por link/token gerado na inicialização. Para acessar:

1. Descubra o nome exato do container:

```bash
docker compose -f docker-compose.prod.yml ps
```

2. Leia os logs do container do dashboard:

```bash
docker logs <NOME_DO_CONTAINER>
```

3. Procure nos logs a URL gerada com token de autenticação.

4. Abra essa URL no navegador.

## Endereços principais para uso durante os testes

- Frontend: `http://localhost`
- API direta: `http://localhost:5000`
- Documentação da API: `http://localhost:5000/swagger`
- MailPit: `http://localhost:8025`
- MinIO Console: `http://localhost:9001`
- Aspire Dashboard: `http://localhost:18888` com URL autenticada obtida via logs
