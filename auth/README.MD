# 🔐 Keycloak como IDP Local com Docker Swarm e Traefik
Este serviço disponibiliza uma instância do [Keycloak](https://www.keycloak.org/) como **Identity Provider (IDP)** para autenticação local, com HTTPS via **Traefik**, ideal para desenvolvimento e testes de SSO (Single Sign-On), OAuth2, OpenID Connect e autenticação centralizada.

---

## 📁 Estrutura do Projeto

```
keycloak/
├── docker-compose.yml       # Stack Docker Swarm do Keycloak + Postgres
└── README.md                # Este arquivo
```

---

## 🧰 Serviços Disponíveis

### 🔑 Keycloak

* Interface Web em: [https://auth.infra.local](https://auth.infra.local)
* Admin Console com usuário `admin` e senha `admin`
* Utiliza PostgreSQL como banco de dados
* Executado em modo `start-dev` para facilitar o uso em desenvolvimento

### 🗃️ Banco de Dados

* PostgreSQL 15
* Volume persistente: `keycloak_db_data`

---

## ⚙️ Configurações do Keycloak

| Variável de ambiente      | Valor       |
| ------------------------- | ----------- |
| `KC_DB`                   | postgres    |
| `KC_DB_URL_HOST`          | keycloak-db |
| `KC_DB_URL_DATABASE`      | keycloak    |
| `KC_DB_USERNAME`          | keycloak    |
| `KC_DB_PASSWORD`          | secret      |
| `KEYCLOAK_ADMIN`          | admin       |
| `KEYCLOAK_ADMIN_PASSWORD` | admin       |

---

## 🔐 Integração com Traefik

O serviço está pronto para ser roteado pelo Traefik com HTTPS:

* Domínio: `https://auth.infra.local`
* Porta interna: `8080`
* Roteamento configurado via labels

---

## 📜 Pré-requisitos

### 🧾 Entrada no `/etc/hosts`

Adicione a seguinte linha:

```
127.0.0.1 auth.infra.local
```

### 🔐 Geração de Certificado SSL com `mkcert`

```bash
mkcert auth.infra.local
```

* Coloque os arquivos `.pem` gerados no diretório de certificados do Traefik.

---

## 🚀 Deploy da Stack

Para subir o serviço no Docker Swarm:

```bash
docker stack deploy -c docker-compose.yml keycloak
```

---

## 🧪 Acessando

1. Abra `https://auth.infra.local` no navegador.
2. Faça login com:

   * **Usuário:** `admin`
   * **Senha:** `admin`

---

## 🛠️ Dicas de Configuração Avançada

* Utilize `start` ao invés de `start-dev` para produção.
* Configure SMTP para redefinição de senhas.
* Habilite login com provedores externos (Google, GitHub, etc).

---

Com isso, você tem um **IDP local robusto**, com roteamento HTTPS confiável para integrar suas aplicações internas com autenticação unificada via Keycloak.
