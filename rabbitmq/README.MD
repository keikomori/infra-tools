## 🐰 RabbitMQ - Fila de Mensagens para Ambientes Distribuídos

O RabbitMQ é um broker de mensagens robusto e amplamente utilizado em arquiteturas assíncronas baseadas em filas (AMQP). Nesta stack, o serviço é configurado com SSL local, pronto para uso com Docker Swarm e Traefik.

---

### 🎯 Objetivo

Fornecer um serviço de mensageria confiável e acessível via HTTPS local com interface de gerenciamento, ideal para sistemas distribuídos, microserviços e integração com workers e task queues.

---

### 🧩 Principais Funcionalidades

* Interface de administração web (RabbitMQ Management)
* Comunicação via AMQP (porta 5672) e HTTP (porta 15672)
* Compatível com SSL local via Traefik
* Deploy em ambiente Docker Swarm

---

### 🛠️ Configuração (docker-compose.yml)

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - keiko_network
    deploy:
      restart_policy:
        condition: any
      placement:
        constraints:
          - node.role == manager
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.rabbitmq.rule=Host(`rabbitmq.infra.local`)"
      - "traefik.http.routers.rabbitmq.entrypoints=websecure"
      - "traefik.http.routers.rabbitmq.tls=true"
      - "traefik.http.services.rabbitmq.loadbalancer.server.port=15672"
      - "traefik.docker.network=keiko_network"
```

---

### 📁 Estrutura de Arquivos

```
rabbitmq/
├── docker-compose.yml     # Stack com RabbitMQ + Traefik
└── README.md              # Este documento
```

---

### 🔐 Certificados

Certificados SSL para acesso local com HTTPS, gerados com `mkcert`:

```bash
mkcert rabbitmq.infra.local
```

Arquivos `.pem` devem estar em `certs/` e serem montados no serviço do Traefik.

---

### 🌐 Acesso

| URL                                                          | Descrição                      |
| ------------------------------------------------------------ | ------------------------------ |
| [https://rabbitmq.infra.local](https://rabbitmq.infra.local) | Interface de administração web |
| Usuário padrão                                               | `admin`                        |
| Senha padrão                                                 | `admin`                        |

---

### 📦 Deploy

```bash
docker stack deploy -c docker-compose.yml rabbitmq
```

---

### 📎 Observações

* A interface web roda na porta `15672`, exposta pelo Traefik via HTTPS.
* A porta `5672` para comunicação AMQP não precisa ser publicada externamente no Swarm, pois será usada via rede interna `keiko_network`.

---

### ✅ Requisitos

* Rede externa `keiko_network` criada.
* Certificados `.pem` válidos no diretório `certs/`.
* Stack `traefik` funcionando corretamente com TLS.
