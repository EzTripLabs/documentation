# Sistema de E-mails EzTrip

## Arquitetura

```
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────────┐
│ Command Handler  │────>│   IEmailQueue         │────>│  Hangfire Worker  │
│ (Application)   │     │   (HangfireEmailQueue) │     │  (SendEmailJob)   │
└─────────────────┘     └──────────────────────┘     └────────┬─────────┘
                                                              │
                                              ┌───────────────┴───────────────┐
                                              │                               │
                                         ┌────▼────┐                   ┌─────▼─────┐
                                         │  Retry  │                   │ Dead Letter│
                                         │ (x5 max)│                   │   Queue    │
                                         └────┬────┘                   └─────┬─────┘
                                              │                              │
                                         ┌────▼────┐                        │
                                         │  Smtp   │                        │
                                         │  Pool   │                        │
                                         └────┬────┘                        │
                                              │                              │
                                         ┌────▼────┐                        │
                                         │ MailKit  │                       │
                                         └─────────┘                       │
                                                              ┌─────────────▼──────┐
                                                              │ email_tracking     │
                                                              │ email_dead_letter  │
                                                              └────────────────────┘
```

### Componentes

| Camada | Projeto | Responsabilidade |
|--------|---------|-----------------|
| **Contracts** | `Shared.Kernel.Email` | Interfaces, modelos, configuração, i18n |
| **Domain** | `Trips.Domain` | Fábricas de mensagens (`AuthEmailMessageFactory`) |
| **Infrastructure** | `Trips.Infrastructure` | Implementação (MailKit, Hangfire, MJML, Tracking) |
| **API** | `EzTrip.Api` | DI, Hangfire Dashboard, Program.cs |

### Schemas PostgreSQL

| Schema | Tabelas | Finalidade |
|--------|---------|------------|
| `eztrip_email` | `email_tracking`, `email_dead_letter` | Rastreamento de envios |
| `hangfire` | (automático) | Jobs background do Hangfire |

---

## Como adicionar um novo tipo de e-mail

### Passo a passo

1. **Criar template MJML**

   Arquivo: `src/Modules/Trips/Trips.Infrastructure/Email/Templates/{Categoria}/{Nome}.mjml`

   ```mjml
   <mjml>
     <mj-head>
       <mj-title>{{email.meu_novo.subject}}</mj-title>
     </mj-head>
     <mj-body>
       ...
     </mj-body>
   </mjml>
   ```

2. **Adicionar chaves de tradução**

   `src/Shared.Kernel.Email/Resources/pt-BR.json`:
   ```json
   {
     "email.meu_novo.subject": "[EzTrip] Meu novo e-mail",
     "email.meu_novo.title": "Título do e-mail"
   }
   ```

   `src/Shared.Kernel.Email/Resources/en-US.json`:
   ```json
   {
     "email.meu_novo.subject": "[EzTrip] My new email",
     "email.meu_novo.title": "Email title"
   }
   ```

3. **Adicionar chave de tradução tipada (opcional)**

   `src/Shared.Kernel.Email/Localization/IEmailTranslationKeys.cs`:
   ```csharp
   public const string MeuNovoSubject = "email.meu_novo.subject";
   ```

4. **Adicionar no enum `EmailType`**

   `src/Shared.Kernel.Email/Models/EmailType.cs`:
   ```csharp
   public enum EmailType { ..., MeuNovoEmail }
   ```

5. **Criar método na factory**

   `src/Modules/Trips/Trips.Domain/Services/User/Auth/AuthEmailMessageFactory.cs`:
   ```csharp
   public async Task<EmailMessage> CreateMeuNovoEmailMessageAsync(
       string email, string language = "ptBR")
   {
       var subject = localizer.Translate(EmailTranslationKeys.MeuNovoSubject, language);
       var html = await templateRenderer.RenderAsync("MeuNovo", language, new
       {
           year = DateTime.UtcNow.Year
       });

       return new EmailMessage(EmailType.MeuNovoEmail, email, subject)
       {
           BodyHtml = html,
           BodyPlain = "...",
           Language = language
       };
   }
   ```

6. **Compilar o MJML**

   ```bash
   cd src/Modules/Trips/Trips.Infrastructure/Email/Templates
   npm run build
   ```

7. **No Command Handler, usar `IEmailQueue`**

   ```csharp
   var message = await factory.CreateMeuNovoEmailMessageAsync(email, user.Language);
   await emailQueue.EnqueueAsync(message, cancellationToken: ct);
   ```

8. **Testar com Mailpit em dev**

---

## Configuração por Ambiente

### Local (desenvolvimento) - Mailpit

O `docker-compose.yml` já inclui Mailpit.

**SMTP:** `localhost:1025` (sem autenticação)
**UI Web:** `http://localhost:8025`

`appsettings.Development.json`:
```json
{
  "SmtpSettings": {
    "Host": "localhost",
    "Port": 1025,
    "UseSsl": false,
    "UseStartTls": false,
    "Username": "",
    "Password": "",
    "FromEmail": "no-reply@eztrip.local",
    "FromName": "[EzTrip Dev]",
    "MaxConnectionPoolSize": 2,
    "ThrottlePerSecond": 50
  },
  "EmailRetrySettings": {
    "MaxRetryAttempts": 1,
    "RetryDelayBaseSeconds": 5
  },
  "EmailTemplateSettings": {
    "TemplatesBasePath": "Email/Templates/Compiled"
  }
}
```

### Produção - Gmail (estágio atual)

**Requisitos:**
- Conta Google Workspace ou Gmail com 2FA ativado
- App Password (gerado em https://myaccount.google.com/apppasswords)
- Limites: 500 emails/dia (Gmail comum) | 2000/dia (Google Workspace)

`appsettings.Production.json`:
```json
{
  "SmtpSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587,
    "UseSsl": false,
    "UseStartTls": true,
    "Username": "no-reply@eztrip.com.br",
    "Password": "SEU_APP_PASSWORD_16_CARACTERES",
    "FromEmail": "no-reply@eztrip.com.br",
    "FromName": "EzTrip",
    "UseOAuth2": false,
    "MaxConnectionPoolSize": 5,
    "ThrottlePerSecond": 10
  }
}
```

**No Docker Compose (produção):**
```yaml
SmtpSettings__Host: "smtp.gmail.com"
SmtpSettings__Port: 587
SmtpSettings__UseStartTls: true
SmtpSettings__Username: "no-reply@eztrip.com.br"
SmtpSettings__Password: "${SMTP_PASSWORD}"
```

> A senha SMTP deve ser passada via variável de ambiente (`SMTP_PASSWORD`), nunca versionada.

### Futuro - AWS SES

**Quando migrar:**

`appsettings.Production.json`:
```json
{
  "SmtpSettings": {
    "Host": "email-smtp.us-east-1.amazonaws.com",
    "Port": 587,
    "UseStartTls": true,
    "Username": "SES_SMTP_USERNAME",
    "Password": "SES_SMTP_PASSWORD",
    "FromEmail": "no-reply@eztrip.com.br",
    "FromName": "EzTrip"
  }
}
```

**Configuração necessária na AWS:**
1. Verificar domínio no SES
2. Solicitar aumento de limite de envio
3. Configurar DKIM no SES (gerenciamento automático)
4. Migrar DNS SPF para incluir `include:amazonses.com`

---

## Hangfire Dashboard

Após iniciar a aplicação, acessar:

```
http://localhost:5000/hangfire
```

O dashboard permite:
- Monitorar fila de e-mails
- Ver jobs com falha e retentativas
- Reenviar jobs manualmente
- Ver métricas de throughput

**Em produção, proteger com autenticação.**

---

## i18n - Internacionalização

### Como funciona

1. `User.Language` armazena o idioma do usuário (`"ptBR"` ou `"enUS"`)
2. Esse valor é passado para a factory ao criar a mensagem
3. O `ITemplateRenderer` resolve as chaves de tradução (`{{email.verification.title}}`)
4. O `IEmailLocalizer` carrega do JSON correspondente (`pt-BR.json` ou `en-US.json`)

### Adicionar novo idioma

1. Criar `src/Shared.Kernel.Email/Resources/{codigo}.json`
2. Adicionar mapeamento em `JsonEmailLocalizer.NormalizeLanguage()`

---

## DKIM

### Gerar chaves

```bash
openssl genrsa -out dkim-private.pem 2048
openssl rsa -in dkim-private.pem -pubout -out dkim-public.pem
```

### Publicar DNS

```
default._domainkey.eztrip.com.br  TXT  "v=DKIM1; h=sha256; k=rsa; p=CONTEUDO_DA_CHAVE_PUBLICA"
```

### No código

O DKIM signing é automático quando o arquivo `dkim-private.pem` existe no diretório da aplicação.

Para configurar via ambiente Docker:
```yaml
DKIM_PRIVATE_KEY_PATH: "/run/secrets/dkim-private.pem"
```

---

## Boas práticas ao desenvolver

1. **Sempre use `IEmailQueue`, não `IEmailSender` diretamente** - garante persistência e retry
2. **Passe `user.Language` para a factory** - respeita a preferência do usuário
3. **MJML templates primeiro, compile depois** - nunca edite o HTML compilado diretamente
4. **Teste com Mailpit** - veja os e-mails em `http://localhost:8025`
5. **Verifique o tracking** - a tabela `eztrip_email.email_tracking` mostra o status de cada envio
6. **Dead Letter** - e-mails com falha permanente vão para `eztrip_email.email_dead_letter`

---

## Troubleshooting

| Problema | Causa provável | Solução |
|----------|---------------|---------|
| E-mail não enviado | Hangfire não configurado | Verificar `AddEzripEmail()` em Program.cs |
| Erro SMTP "Authentication failed" | Senha incorreta ou App Password não gerado | Gerar novo App Password no Google |
| Template não encontrado | MJML não compilado | Rodar `npm run build` na pasta Templates |
| Erro "Schema does not exist" | Migration não rodou | Verificar `app.RunMigrations()` em Program.cs |
| E-mail cai em spam | DKIM/DMARC não configurado | Configurar DNS records |
| "RetryCount" não funciona | Hangfire `AutomaticRetry` configurado como 0 | Deixar `[AutomaticRetry]` sem parâmetros |
