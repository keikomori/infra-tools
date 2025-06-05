## 🔄 Apache NiFi - Plataforma de Integração de Dados

Este serviço configura o **Apache NiFi** como ferramenta de fluxo de dados (ETL) em ambiente Docker Swarm, com suporte a HTTPS local via Traefik e certificados gerados com `mkcert`.

---

### 🎯 Objetivo

Prover uma plataforma visual para orquestrar, transformar e monitorar fluxos de dados entre sistemas distintos, com suporte a mapeamentos, transformação, e integração com bancos, APIs e arquivos.

---

### 🛠️ Configuração (docker-compose.yml)

```yaml
services:
  nifi:
    image: apache/nifi:latest
    hostname: nifi
    ports:
      - "8080:8080"
    environment:
      - NIFI_WEB_HTTP_PORT=8443
      - NIFI_WEB_HTTP_HOST=0.0.0.0
    volumes:
      - nifi_data:/opt/nifi/nifi-current/data
    networks:
      - keiko_network
    deploy:
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nifi.rule=Host(`nifi.infra.local`)"
      - "traefik.http.routers.nifi.entrypoints=websecure"
      - "traefik.http.routers.nifi.tls=true"
      - "traefik.http.services.nifi.loadbalancer.server.port=8443"
      - "traefik.docker.network=keiko_network"

volumes:
  nifi_data:

networks:
  keiko_network:
    external: true
```

---

### 🧩 Funcionalidades

* Interface visual em `https://nifi.infra.local`
* Persistência de dados dos fluxos no volume `nifi_data`
* Roteamento SSL via Traefik
* Suporte a autenticação e templates
* Pronto para integração com Kafka, RabbitMQ, PostgreSQL, APIs REST, etc.

---

### 🔐 HTTPS Local

Certificados gerados via [`mkcert`](https://github.com/FiloSottile/mkcert):

```bash
mkcert nifi.infra.local
```

Depois, coloque os arquivos `.pem` na pasta `certs/` usada pelo Traefik.

---

### 🗂️ Estrutura

```
nifi/
├── docker-compose.yml     # Stack do NiFi
└── README.md              # Este documento
```

---

### 🌐 Acesso

| Endereço                                             | Descrição                     |
| ---------------------------------------------------- | ----------------------------- |
| [https://nifi.infra.local](https://nifi.infra.local) | Interface Web do NiFi         |
| `nifi` (na rede Swarm)                               | Host interno para integrações |

---

### 📦 Deploy

```bash
docker stack deploy -c nifi/docker-compose.yml nifi
```

---

### ✅ Requisitos

* Rede Docker Swarm ativa
* Certificado SSL local válido para `nifi.infra.local`
* Entrada no `/etc/hosts`:

```bash
127.0.0.1 nifi.infra.local
```