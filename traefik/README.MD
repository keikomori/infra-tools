# Traefik com HTTPS Local usando mkcert

Este guia explica como configurar o Traefik em um ambiente local com suporte a HTTPS via certificados gerados pelo `mkcert`, usando Docker Swarm e rede compartilhada.

## Estrutura de diretórios

```bash
infra-tools/
├── certs/
│   ├── traefik.infra.local.pem
│   ├── traefik.infra.local-key.pem
│   ├── portainer.infra.local.pem
│   ├── portainer.infra.local-key.pem
│   └── grafana.portainer.infra.local.pem
│   └── grafana.portainer.infra.local-key.pem
├── traefik/
│   ├── docker-compose.yml
│   ├── traefik.toml
│   ├── traefik_dynamic.toml
│   └── acme.json
```

## 1. Gerar certificados com mkcert

```bash
mkcert traefik.infra.local
mkcert portainer.infra.local
mkcert grafana.portainer.infra.local

mv *.pem ../certs/
```

## 2. Atualizar o traefik\_dynamic.toml

Adicione:

```toml
[tls]
  [[tls.certificates]]
    certFile = "/certs/traefik.infra.local.pem"
    keyFile = "/certs/traefik.infra.local-key.pem"

  [[tls.certificates]]
    certFile = "/certs/portainer.infra.local.pem"
    keyFile = "/certs/portainer.infra.local-key.pem"

  [[tls.certificates]]
    certFile = "/certs/grafana.portainer.infra.local.pem"
    keyFile = "/certs/grafana.portainer.infra.local-key.pem"

[http.routers.api]
  rule = "Host(`traefik.infra.local`)"
  entrypoints = ["websecure"]
  service = "api@internal"
  [http.routers.api.tls]
    domains = ["traefik.infra.local"]
```

## 3. Atualizar volumes no docker-compose.yml

```yaml
    volumes:
      - ./traefik.toml:/traefik.toml
      - ./traefik_dynamic.toml:/traefik_dynamic.toml
      - ./acme.json:/acme.json
      - ../certs/traefik.infra.local.pem:/certs/traefik.infra.local.pem
      - ../certs/traefik.infra.local-key.pem:/certs/traefik.infra.local-key.pem
      - ../certs/portainer.infra.local.pem:/certs/portainer.infra.local.pem
      - ../certs/portainer.infra.local-key.pem:/certs/portainer.infra.local-key.pem
      - ../certs/grafana.portainer.infra.local.pem:/certs/grafana.portainer.infra.local.pem
      - ../certs/grafana.portainer.infra.local-key.pem:/certs/grafana.portainer.infra.local-key.pem
```

## 4. Atualizar /etc/hosts

```bash
echo "127.0.0.1 traefik.infra.local portainer.infra.local grafana.portainer.infra.local" | sudo tee -a /etc/hosts
```

## 5. Deploy com Docker Swarm

```bash
cd traefik
docker stack deploy -c docker-compose.yml traefik
```

## 6. Acessar o Traefik

Abra [https://traefik.infra.local](https://traefik.infra.local) no navegador. O certificado deve estar confiável se o `mkcert` estiver corretamente instalado.
