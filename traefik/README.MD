🔐 HTTPS Local com Traefik e mkcert
1. Instale o mkcert
Se ainda não tem o mkcert:

```bash
sudo apt install libnss3-tools
brew install mkcert
mkcert -install
```
O mkcert -install adiciona um certificado raiz confiável ao sistema.

2. Gere o certificado local
Dentro da pasta traefik/ do seu projeto:

```bash
cd traefik
mkcert traefik.localhost
mkdir -p certs
mv traefik.localhost.pem certs/
mv traefik.localhost-key.pem certs/
```

3. Atualize o traefik.toml

```toml
[entryPoints]
  [entryPoints.web]
    address = ":80"
    [entryPoints.web.http]
      [entryPoints.web.http.redirections]
        [entryPoints.web.http.redirections.entryPoint]
          to = "websecure"
          scheme = "https"

  [entryPoints.websecure]
    address = ":443"

[log]
  level = "WARN"

[accessLog]

[metrics.prometheus]

[api]
  dashboard = true

[providers.docker]
  watch = true
  network = "keiko_network"

[providers.file]
  filename = "/traefik_dynamic.toml"
```

4. Atualize o traefik_dynamic.toml

```toml
[tls]
  [[tls.certificates]]
    certFile = "/certs/traefik.localhost.pem"
    keyFile = "/certs/traefik.localhost-key.pem"

[http.routers.api]
  rule = "Host(`traefik.localhost`)"
  entrypoints = ["websecure"]
  service = "api@internal"
  [http.routers.api.tls]
    domains = ["traefik.localhost"]
```

5. Atualize o /etc/hosts
Adicione:

```bash
127.0.0.1 traefik.localhost
```

6. Redeploy com Docker Swarm

```bash
docker stack rm traefik
docker stack deploy -c traefik/docker-compose.yml traefik
```

✅ Pronto!
Acesse: https://traefik.localhost
Você verá o painel Traefik com um certificado válido e confiável no navegador.

