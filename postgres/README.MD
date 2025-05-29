## 🐘 PostgreSQL - Banco de Dados para Desenvolvimento Local

Esta stack configura um container PostgreSQL pronto para uso com Docker Swarm, voltado para ambientes de desenvolvimento. Integrado à rede `keiko_network`, ele serve de backend para serviços como Odoo, Keycloak, Grafana, entre outros.

---

### 🎯 Objetivo

Prover um banco de dados relacional robusto, acessível por outros containers na rede `keiko_network`, com persistência e segurança.

---

### 🛠️ Configuração (docker-compose.yml)

```yaml
services:
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=1234
      - POSTGRES_DB=dev
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - keiko_network
    deploy:
      restart_policy:
        condition: any
      placement:
        constraints:
          - node.role == manager

volumes:
  postgres_data:

networks:
  keiko_network:
    external: true
```

---

### 🧩 Funcionalidades

* Banco de dados PostgreSQL 15
* Volume persistente de dados
* Pronto para uso com aplicações como Odoo, Keycloak e outras
* Conexão segura via rede Swarm

---

### 🌐 Acesso

| Endereço   | Descrição                  |
| ---------- | -------------------------- |
| `postgres` | Host interno via Docker    |
| `5432`     | Porta padrão do PostgreSQL |

Você pode acessar via terminal com:

```bash
psql -h postgres -U odoo -d dev
```

Ou usando ferramentas gráficas como DBeaver e TablePlus.

---

### 📁 Estrutura

```
postgres/
├── docker-compose.yml     # Stack do PostgreSQL
└── README.md              # Este documento
```

---

### 📦 Deploy

```bash
docker stack deploy -c docker-compose.yml postgres
```

---

### ✅ Requisitos

* Docker Swarm inicializado
* Rede externa `keiko_network` criada previamente:

```bash
docker network create --driver=overlay --attachable keiko_network
```
