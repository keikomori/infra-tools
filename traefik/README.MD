## 🔀 Traefik - Proxy Reverso com HTTPS Local

Este serviço fornece o proxy reverso central da infraestrutura. Ele é responsável por:

* Encaminhar tráfego HTTPS para os serviços internos com base no domínio.
* Gerenciar certificados locais gerados com [mkcert](https://github.com/FiloSottile/mkcert).
* Servir a UI de administração do Traefik via dashboard.

---

### 🧩 Principais Funcionalidades

* Entrada única para todos os serviços via HTTPS local.
* Integração com Docker Swarm e labels para roteamento automático.
* Suporte a certificados gerados com mkcert.
* UI disponível em `https://traefik.infra.local`.

---

### 🗂️ Estrutura de Arquivos

```
traefik/
├── docker-compose.yml       # Stack Traefik com suporte a HTTPS
├── acme.json                # Certificados gerenciados por Traefik (deixe com chmod 600)
├── traefik.toml             # Configuração principal do Traefik
└── traefik_dynamic.toml     # Regras dinâmicas, middlewares, TLS, etc.
```

---

### ⚙️ Como funciona

* O serviço `traefik` escuta em duas portas:

  * `80` (HTTP) → redireciona para HTTPS
  * `443` (HTTPS) → entrada principal com certificados locais

* A configuração `traefik.toml` aponta para `traefik_dynamic.toml`, onde estão definidos:

  * Middlewares como redirecionamento, autenticação, etc
  * Certificados TLS estáticos
  * Roteamento baseado em `Host()` (ex: `grafana.infra.local`)

---

### 🔐 Certificados Locais

Os certificados são gerados com mkcert e montados dentro do container:

```bash
mkcert traefik.infra.local
```

```toml
[tls.certificates]
  certFile = "/etc/traefik/certs/traefik.infra.local.pem"
  keyFile = "/etc/traefik/certs/traefik.infra.local-key.pem"
```

---

### 🔍 Acessos

| URL                                                        | Descrição                      |
| ---------------------------------------------------------- | ------------------------------ |
| [https://traefik.infra.local](https://traefik.infra.local) | Painel do Traefik (porta 8080) |

---

### 📦 Deploy

```bash
docker stack deploy -c traefik/docker-compose.yml traefik
```

> Lembre-se de que todos os outros serviços devem estar na mesma rede `keiko_network` para serem roteados corretamente.

---

### 🔐 Segurança

* A dashboard pode ser protegida com middleware de autenticação.
* Para ambientes sensíveis, é recomendável habilitar autenticação básica ou OAuth2.

