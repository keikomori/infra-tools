# PostgreSQL para Desenvolvimento com Docker Swarm

Este guia documenta a configuração do PostgreSQL em um ambiente de desenvolvimento, usando Docker Swarm e rede `keiko_network`, sem Traefik ou SSL, para facilitar integração com serviços como Odoo.

## Estrutura do Projeto

```bash
infra-tools/
├── postgres/
│   └── docker-compose.yml
```

## 1. Configuração do PostgreSQL

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:17
    deploy:
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=devpass
      - POSTGRES_DB=devdb
    ports:
      - target: 5432
        published: 5432
        mode: host
    networks:
      - keiko_network
    logging:
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  postgres_data:

networks:
  keiko_network:
    external: true
```

## 2. Deploy

```bash
cd postgres
docker stack deploy -c docker-compose.yml postgres
```

## 3. Acessar o banco

* **Host:** localhost
* **Porta:** 5432
* **Usuário:** odoo
* **Senha:** 1234
* **Database:** devdb

Você pode se conectar usando DBeaver, Adminer ou via outros containers na rede `keiko_network` com `postgres:5432`.
