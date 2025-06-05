# Infraestrutura Local de Desenvolvimento com Docker Swarm & Traefik

Este repositório fornece uma infraestrutura local completa e segura para ambientes de desenvolvimento, utilizando **Docker Swarm**, **Traefik** como proxy reverso com HTTPS local, e serviços essenciais como:

- 🔐 Autenticação com **Keycloak** (Auth)
- 📨 Servidor de E-mail com **Postfix + Dovecot**
- 🐘 Banco de Dados **PostgreSQL**
- 🐇 Mensageria com **RabbitMQ**
- 🔍 Monitoramento com **Grafana + Prometheus**
- ⚙️ Gerenciamento com **Portainer**
- ⚡ Orquestração com **Traefik**
- 📡 Orquestração de dados com **Apache NiFi**

Todos os serviços compartilham a rede `keiko_network` e podem ser acessados com domínios `.infra.local`, com certificados gerados por `mkcert`.

---

## 📁 Estrutura de Diretórios

```bash
infra-tools/
├── auth/           → Keycloak configurado como IDP local
├── certs/          → Certificados locais (.pem) gerados com mkcert
├── mail-server/    → Servidor SMTP/IMAP local (Postfix + Dovecot + SSL)
├── monitoring/     → Grafana + Prometheus configurados com SSL
├── nifi/           → Apache NiFi com acesso seguro via Traefik
├── portainer/      → Gerenciamento de containers e stacks
├── postgres/       → Banco de dados PostgreSQL (modo swarm)
├── rabbitmq/       → RabbitMQ com UI via Traefik e SSL
├── traefik/        → Configuração do Traefik e certificados HTTPS
└── README.md       → Este arquivo
````

---

## 🔐 Certificados com `mkcert`

Os certificados foram gerados localmente com [mkcert](https://github.com/FiloSottile/mkcert):

```bash
mkcert traefik.infra.local
mkcert portainer.infra.local
mkcert grafana.infra.local
mkcert mail.infra.local
mkcert nifi.infra.local
mkcert rabbitmq.infra.local
````

Os arquivos `.pem` estão em `./certs/` e são montados nos containers via volume.

---

## 🧠 Serviços Disponíveis

| Serviço     | URL                                                            | Porta interna | Stack dir      |
| ----------- | -------------------------------------------------------------- | ------------- | -------------- |
| Traefik     | [https://traefik.infra.local](https://traefik.infra.local)     | 8080          | `traefik/`     |
| Portainer   | [https://portainer.infra.local](https://portainer.infra.local) | 9000          | `portainer/`   |
| Grafana     | [https://grafana.infra.local](https://grafana.infra.local)     | 3000          | `monitoring/`  |
| PostgreSQL  | (sem UI, acesso via apps)                                      | 5432          | `postgres/`    |
| RabbitMQ    | [https://rabbitmq.infra.local](https://rabbitmq.infra.local)   | 15672         | `rabbitmq/`    |
| Keycloak    | [https://auth.infra.local](https://auth.infra.local)           | 8080          | `auth/`        |
| NiFi        | [https://nifi.infra.local](https://nifi.infra.local)           | 8443          | `nifi/`        |
| Mail Server | SMTP (2525/465), IMAP (2143/993)                               | vários        | `mail-server/` |

---

## ⚙️ Como executar

1. Inicialize a rede compartilhada:

```bash
docker network create --driver=overlay --attachable keiko_network
```

2. Faça o deploy de cada stack (em modo swarm):

```bash
cd traefik && docker stack deploy -c docker-compose.yml traefik
cd postgres && docker stack deploy -c docker-compose.yml postgres
cd portainer && docker stack deploy -c docker-compose.yml portainer
cd monitoring && docker stack deploy -c docker-compose.yml monitoring
cd rabbitmq && docker stack deploy -c docker-compose.yml rabbitmq
cd auth && docker stack deploy -c docker-compose.yml auth
cd mail-server && docker stack deploy -c docker-compose.yml mail
cd nifi && docker stack deploy -c docker-compose.yml nifi
```

---

## 🛠️ Requisitos

* Docker + Docker Swarm
* mkcert instalado e confiável no sistema
* Domínios adicionados em `/etc/hosts`:

```bash
127.0.0.1 traefik.infra.local portainer.infra.local grafana.infra.local mail.infra.local rabbitmq.infra.local auth.infra.local nifi.infra.local
```

---

## 📊 Monitoramento

* **Grafana** e **Prometheus** monitoram containers e serviços.
* Pré-configurações podem ser adicionadas via `prometheus.yml` e dashboards JSON.

---

## ✉️ Servidor de E-mail Local

* Postfix e Dovecot com suporte a SSL/TLS.
* Usuários podem ser configurados via arquivos `userdb`, `passdb`, etc.
* Ideal para testes de envio/recebimento de e-mails em ambiente de desenvolvimento.

---

## 🧪 Teste e Validação

Você pode testar o acesso seguro a qualquer serviço via navegador, ou com `curl`:

```bash
curl -k https://grafana.infra.local
curl -k https://portainer.infra.local
```

---

## 📌 Observações

* Todos os certificados são **locais**. Não usar em produção.
* Serviços utilizam **Traefik com entrypoint `websecure`**.
* Estrutura modular permite a inclusão de outros serviços futuros.

---

## 📂 Créditos e Licença

Desenvolvido para uso interno e ambientes de desenvolvimento.
Licença MIT.

---
