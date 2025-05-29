# 📬 Mail Server Local com Docker Swarm, Postfix, Dovecot e SSL

Esta stack implementa um servidor de e-mail local de desenvolvimento utilizando **Postfix** (SMTP) e **Dovecot** (IMAP) com suporte a SSL via certificados `mkcert`, totalmente orquestrado com **Docker Swarm** e integrado ao **Traefik** como proxy reverso seguro.

---

## 📁 Estrutura do Projeto

```bash
mail-server/
├── postfix/               # Dockerfile e configurações do Postfix (SMTP)
├── dovecot/               # Dockerfile e configurações do Dovecot (IMAP)
├── docker-compose.yml     # Stack Docker Swarm do mail server
└── README.md              # Este arquivo
```

---

## 📦 Serviços

### 📤 Postfix (SMTP)

* Porta externa: `2525`
* Interna (SMTP): `25`
* Envia emails de containers ou aplicações internas
* Integrado ao Dovecot via LMTP (Local Mail Transfer Protocol)

### 📥 Dovecot (IMAP)

* Porta externa: `2143`
* Interna (IMAP): `143`
* Recebe mensagens salvas por Postfix
* Gerencia autenticação e acesso à caixa de entrada (Maildir)

---

## 🔐 SSL com mkcert

Certificados locais gerados com `mkcert`:

* `mail.infra.local.pem`
* `mail.infra.local-key.pem`

Esses arquivos devem estar no diretório `certs/` compartilhado com o Traefik.

---

## 🧠 Autenticação

* Usuários e senhas estão definidos em:

  * `dovecot/passdb` e `userdb`
  * `postfix/users`

* Postfix usa SASL via Dovecot para autenticar envio

* Suporte a Maildir: `mail_location = maildir:/var/mail/%u`

---

## 🗃️ Volumes

| Nome            | Caminho              | Uso                      |
| --------------- | -------------------- | ------------------------ |
| `mail_data`     | `/var/mail`          | Armazenamento das caixas |
| `postfix_spool` | `/var/spool/postfix` | Spool do Postfix         |

---

## 🌐 Integração com Traefik

Para expor o serviço com domínio local e HTTPS:

Adicione ao `traefik_dynamic.toml` (se necessário):

```toml
[[tls.certificates]]
  certFile = "/certs/mail.infra.local.pem"
  keyFile  = "/certs/mail.infra.local-key.pem"
```

E no `/etc/hosts`:

```bash
127.0.0.1 mail.infra.local
```

---

## 🐳 Como Usar

### 1. ⚙️ Construa as imagens

```bash
cd mail-server/postfix
docker build -t mail-postfix:latest .

cd ../dovecot
docker build -t mail-dovecot:latest .
```

> Alternativamente, envie para um registry se for usar em múltiplos nós do Swarm.

---

### 2. 🚀 Suba a stack

```bash
cd mail-server
docker stack deploy -c docker-compose.yml mail
```

---

## 📬 Teste com Telnet (SMTP)

```bash
telnet localhost 2525
EHLO mail.infra.local
AUTH PLAIN [base64_usuario_senha]
MAIL FROM:<teste@mail.infra.local>
RCPT TO:<destino@mail.infra.local>
DATA
Subject: Teste
Mensagem
.
QUIT
```

---

## 📥 Teste com Cliente IMAP (Thunderbird, Outlook)

* Servidor IMAP: `mail.infra.local`
* Porta: `2143`
* SSL: pode ser ativado com certificado gerado via mkcert
* Usuário/senha conforme `userdb` e `passdb`

---

## ✅ Próximos passos

* [ ] Adicionar suporte a SMTP TLS (STARTTLS)
* [ ] Gerenciar usuários via base externa (LDAP, Keycloak etc)
* [ ] Adicionar Roundcube como webmail local (opcional)
