## 🛠️ Portainer - Gerenciador de Containers para Docker Swarm

O Portainer fornece uma interface gráfica leve para o gerenciamento de containers, volumes, stacks e redes no Docker Swarm.

---

### 🎯 Objetivo

Facilitar o gerenciamento da infraestrutura Docker de forma visual, sem depender exclusivamente da linha de comando.

---

### 🧩 Principais Funcionalidades

* Gerenciamento de stacks e serviços no Swarm.
* Visualização de logs, volumes, redes e containers.
* Suporte a múltiplos ambientes (Swarm remoto, agentes, etc).
* Acesso com SSL local em `https://portainer.infra.local`.

---

### 📦 Configuração

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
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
      - "traefik.http.routers.portainer.rule=Host(`portainer.infra.local`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
      - "traefik.docker.network=keiko_network"
```

---

### 🗂️ Estrutura de Arquivos

```
portainer/
├── docker-compose.yml     # Stack com serviço e labels Traefik
└── README.md              # Este documento
```

---

### 📁 Volumes

* `portainer_data`: volume persistente contendo credenciais, configurações, ambientes e tokens da interface.

---

### 🔐 Certificados

Certificados `.pem` gerados com `mkcert`:

```bash
mkcert portainer.infra.local
```

Devem estar disponíveis na pasta `certs/` e montados no Traefik.

---

### 🌐 Acesso

| URL                                                            | Descrição                      |
| -------------------------------------------------------------- | ------------------------------ |
| [https://portainer.infra.local](https://portainer.infra.local) | Interface de gerenciamento web |

---

### 📦 Deploy

```bash
docker stack deploy -c docker-compose.yml portainer
```

---

### ✅ Requisitos

* O Traefik deve estar em funcionamento na rede `keiko_network`.
* É necessário ter um node com role `manager` no Swarm.
