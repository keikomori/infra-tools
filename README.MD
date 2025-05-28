# Infraestrutura de Desenvolvimento com Docker Swarm e Traefik

Este projeto configura um ambiente de desenvolvimento completo com Docker Swarm, centralizando serviços essenciais como Traefik, PostgreSQL, Portainer, Grafana e Odoo, com suporte a HTTPS local utilizando `mkcert`.

## Estrutura Geral do Projeto

```bash
infra-tools/
├── certs/                           # Certificados SSL gerados com mkcert
├── traefik/                         # Proxy reverso com HTTPS local
├── postgres/                        # Banco de dados PostgreSQL para dev
├── portainer/                       # Interface de gerenciamento de containers
├── monitoring/                      # Grafana e Prometheus
├── odoo/                            # Plataforma Odoo com suporte a debug remoto
│   ├── docker/                      # Dockerfiles por versão
│   │   └── 18.0/Dockerfile          # Dockerfile com debugpy
│   ├── versions/                    # Código oficial clonado por versão
│   ├── addons/                      # Módulos personalizados
│   └── instances/
│       └── dev/                     # Instância de desenvolvimento
│           ├── config/odoo.conf
│           ├── logs/
│           └── docker-compose.yml
```

## Funcionalidades

### 🔐 Traefik com HTTPS Local

* Gerenciamento de roteamento com SSL local usando `mkcert`
* Acesso via domínios personalizados como `traefik.infra.local`, `portainer.infra.local`, etc
* Configuração dinâmica via `traefik_dynamic.toml`

### 🗃️ PostgreSQL para Desenvolvimento

* Rodando em container isolado
* Integração direta com Odoo
* Acesso local via `localhost:5432` e pela rede Swarm via `postgres`

### 📊 Portainer

* Interface web para gerenciamento de stacks, serviços e containers
* Acesso com HTTPS local em `https://portainer.infra.local`

### 📈 Grafana

* Dashboard de visualização com HTTPS local em `https://grafana.portainer.infra.local`
* Pronto para se integrar com Prometheus e cAdvisor

### 🧠 Odoo com Debug Remoto

* Estrutura baseada em instâncias por pasta
* Build customizado com `debugpy` e `pandas`
* Configuração via `odoo.conf` e Traefik para domínio `dev.infra.local`
* Porta `5678` publicada para conexão com PyCharm
* Breakpoints funcionam no core e em módulos personalizados

## Rede Compartilhada

Todos os serviços usam a rede externa `keiko_network`, o que permite comunicação entre containers como:

* Odoo acessando o banco PostgreSQL via `postgres`
* Grafana acessando Prometheus (em futuras integrações)
* Portainer gerenciando containers de todas as stacks

## Certificados com mkcert

```bash
mkcert traefik.infra.local
mkcert portainer.infra.local
mkcert grafana.portainer.infra.local
mkcert dev.infra.local
```

Os certificados `.pem` são montados diretamente no Traefik para HTTPS local confiável.

## Hosts Locais

Adicione ao `/etc/hosts`:

```bash
127.0.0.1 traefik.infra.local portainer.infra.local grafana.portainer.infra.local dev.infra.local
```

## Deploy das Stacks

```bash
cd traefik && docker stack deploy -c docker-compose.yml traefik
cd postgres && docker stack deploy -c docker-compose.yml postgres
cd portainer && docker stack deploy -c docker-compose.yml portainer
cd monitoring && docker stack deploy -c docker-compose.yml monitoring
cd odoo/instances/dev && docker stack deploy -c docker-compose.yml odoo_dev
```

---

Com isso, você terá um ambiente local robusto e realista para desenvolvimento, testes e validações de serviços que simulam a produção.
