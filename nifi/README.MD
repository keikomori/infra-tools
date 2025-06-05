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
    deploy:
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
    ports:
      - "8443:8443"
    networks:
      - keiko_network
    volumes:
      - nifi_data:/opt/nifi/nifi-current
      - ../certs/nifi.infra.local.pem:/certs/nifi.infra.local.pem
      - ../certs/nifi.infra.local-key.pem:/certs/nifi.infra.local-key.pem
    environment:
      - NIFI_WEB_HTTPS_PORT=8443
      - NIFI_WEB_HTTPS_HOST=0.0.0.0
      - NIFI_WEB_PROXY_CONTEXT_PATH=/nifi
      - SINGLE_USER_CREDENTIALS_USERNAME=keiko
      - SINGLE_USER_CREDENTIALS_PASSWORD=123456123456
      - NIFI_SECURITY_KEYSTORE=/opt/nifi/certs/nifi.infra.local.pem
      - NIFI_SECURITY_KEYSTOREPASSWD=changeit
      - NIFI_SECURITY_KEYPASSWD=changeit
      - NIFI_SECURITY_TRUSTSTORE=/opt/nifi/certs/nifi.infra.local.pem
      - NIFI_SECURITY_TRUSTSTOREPASSWD=changeit
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nifi.rule=Host(`nifi.infra.local`) && PathPrefix(`/nifi`)"
      - "traefik.http.routers.nifi.entrypoints=websecure"
      - "traefik.http.routers.nifi.tls=true"
      - "traefik.http.services.nifi.loadbalancer.server.port=8443"
      - "traefik.http.services.nifi.loadbalancer.server.scheme=https"
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