# 🧩 Monitoramento com Docker Swarm, Traefik, Grafana, Prometheus e cAdvisor

Esta stack provê uma solução de **monitoramento local** utilizando Docker Swarm com HTTPS via `Traefik` + `mkcert`. Você poderá visualizar métricas de containers Docker e serviços em tempo real.

---

## 📦 Tecnologias Utilizadas

* ✅ **Grafana** – Dashboards de visualização
* ✅ **Prometheus** – Coletor de métricas
* ✅ **cAdvisor** – Monitoramento de containers Docker
* ✅ **Traefik** – Proxy reverso com suporte a TLS local (via `mkcert`)

---

## 📁 Estrutura do Projeto

```bash
monitoring/
├── docker-compose.yml         # Stack de monitoramento com Traefik
├── prometheus.yml             # Configura os alvos de scraping do Prometheus
└── README.md                  # Este documento
```

---

## 🚀 Serviços

### 📊 Grafana

* Interface: [https://grafana.infra.local](https://grafana.infra.local)
* Porta interna: `3000`
* Persistência: volume `grafana_data`
* Integrado com Prometheus como data source

### 📡 Prometheus

* Interface: [http://localhost:9090](http://localhost:9090)
* Porta interna: `9090`
* Scrape configurado para:

  * Ele mesmo (`localhost:9090`)
  * cAdvisor (`cadvisor:8080`)
* Armazena métricas localmente (pode ser estendido com Alertmanager)

### 📦 cAdvisor

* Interface: [http://localhost:8081](http://localhost:8081) (opcional)
* Expõe métricas dos containers Docker
* Lê dados diretamente do host via `/var/run/docker.sock`

---

## 🔐 HTTPS Local com Traefik e mkcert

Certificados locais são usados para HTTPS realista em ambiente de desenvolvimento.

### 📌 Gerando Certificados

```bash
mkcert grafana.infra.local
```

Coloque os arquivos `.pem` gerados no diretório de certificados do Traefik (`../traefik/certs/`).

---

## 🔧 Configuração do Prometheus

```yaml
# monitoring/prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

---

## 🖥️ /etc/hosts

Adicione as seguintes entradas no seu arquivo `/etc/hosts`:

```bash
127.0.0.1 grafana.infra.local
```

---

## 🐳 Deploy da Stack

### 1. Inicie o Docker Swarm (caso ainda não esteja ativo):

```bash
docker swarm init
```

### 2. Suba a stack:

```bash
docker stack deploy -c monitoring/docker-compose.yml monitoring
```

---

## 🌐 Acessos

| Serviço    | URL                                                        | Porta |
| ---------- | ---------------------------------------------------------- | ----- |
| Grafana    | [https://grafana.infra.local](https://grafana.infra.local) | 3000  |
| Prometheus | [http://localhost:9090](http://localhost:9090)             | 9090  |
| cAdvisor   | [http://localhost:8081](http://localhost:8081)             | 8080  |

---

## 📂 Volumes

| Nome              | Uso                              |
| ----------------- | -------------------------------- |
| `grafana_data`    | Dados e configurações do Grafana |
| `prometheus_data` | Banco de métricas do Prometheus  |

---

## ✅ Próximos Passos

* [ ] Adicionar dashboards automáticos no Grafana (via provisioning)
* [ ] Configurar autenticação via Keycloak ou OAuth no Grafana
* [ ] Adicionar Alertmanager para alertas de eventos críticos

---

💡 **Dica**: Você pode integrar o Grafana com outras fontes de dados como Loki, Elasticsearch ou PostgreSQL dependendo da necessidade da sua infraestrutura.

---