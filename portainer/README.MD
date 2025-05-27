# Portainer com HTTPS Local usando Traefik e mkcert

Este guia documenta a configuração do Portainer em um ambiente local utilizando Docker Swarm e acesso seguro via Traefik com certificados HTTPS gerados por `mkcert`.

## Estrutura do Projeto

```bash
infra-tools/
├── certs/
│   ├── portainer.infra.local.pem
│   └── portainer.infra.local-key.pem
├── portainer/
│   └── docker-compose.yml
```

## 1. Gerar certificados com mkcert

No diretório `infra-tools/traefik/certs/`:

```bash
cd infra-tools/traefik
mkcert portainer.infra.local
mv portainer.infra.local.pem certs/
mv portainer.infra.local-key.pem certs/
```

## 2. Adicionar certificados ao Traefik

Atualize o `traefik_dynamic.toml` com:

```toml
[[tls.certificates]]
  certFile = "/certs/portainer.infra.local.pem"
  keyFile = "/certs/portainer.infra.local-key.pem"
```

E monte os volumes no `traefik/docker-compose.yml`:

```yaml
- ../certs/portainer.infra.local.pem:/certs/portainer.infra.local.pem
- ../certs/portainer.infra.local-key.pem:/certs/portainer.infra.local-key.pem
```

## 3. Configurar o Portainer

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:2.30.1
    deploy:
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      - keiko_network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.infra.local`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
      - "traefik.docker.network=keiko_network"

volumes:
  portainer_data:

networks:
  keiko_network:
    external: true
```

## 4. Atualizar /etc/hosts

```bash
echo "127.0.0.1 portainer.infra.local" | sudo tee -a /etc/hosts
```

## 5. Deploy

```bash
docker stack deploy -c portainer/docker-compose.yml portainer
```

## 6. Acessar

Abra [https://portainer.infra.local](https://portainer.infra.local) no navegador para acessar o Portainer com HTTPS válido localmente.
