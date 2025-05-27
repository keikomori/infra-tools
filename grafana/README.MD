# Grafana com HTTPS Local usando Traefik e mkcert

Este guia documenta a configuração do Grafana em um ambiente local utilizando Docker Swarm e acesso seguro via Traefik com certificados HTTPS gerados por `mkcert`.

## Estrutura do Projeto

```bash
infra-tools/
├── certs/
│   ├── grafana.portainer.infra.local.pem
│   └── grafana.portainer.infra.local-key.pem
├── monitoring/
│   └── docker-compose.yml
```

## 1. Gerar certificados com mkcert

No diretório `infra-tools/certs/`:

```bash
cd infra-tools/traefik
mkcert grafana.portainer.infra.local
mv grafana.portainer.infra.local.pem certs/
mv grafana.portainer.infra.local-key.pem certs/
```

## 2. Adicionar certificados ao Traefik

Atualize o `traefik_dynamic.toml` com:

```toml
[[tls.certificates]]
  certFile = "/certs/grafana.portainer.infra.local.pem"
  keyFile = "/certs/grafana.portainer.infra.local-key.pem"
```

E monte os volumes no `traefik/docker-compose.yml`:

```yaml
- ../certs/grafana.portainer.infra.local.pem:/certs/grafana.portainer.infra.local.pem
- ../certs/grafana.portainer.infra.local-key.pem:/certs/grafana.portainer.infra.local-key.pem
```

## 3. Configurar o Grafana

```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: any
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - keiko_network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.grafana.rule=Host(`grafana.portainer.infra.local`)"
      - "traefik.http.routers.grafana.entrypoints=websecure"
      - "traefik.http.routers.grafana.tls=true"
      - "traefik.http.services.grafana.loadbalancer.server.port=3000"
      - "traefik.docker.network=keiko_network"

volumes:
  grafana_data:

networks:
  keiko_network:
    external: true
```

## 4. Atualizar /etc/hosts

```bash
echo "127.0.0.1 grafana.portainer.infra.local" | sudo tee -a /etc/hosts
```

## 5. Deploy

```bash
docker stack deploy -c monitoring/docker-compose.yml monitoring
```

## 6. Acessar

Abra [https://grafana.portainer.infra.local](https://grafana.portainer.infra.local) no navegador para acessar o dashboard do Grafana com HTTPS válido localmente.
