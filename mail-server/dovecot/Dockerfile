FROM debian:latest

RUN apt-get update && \
    apt-get install -y dovecot-core dovecot-imapd dovecot-lmtpd && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -y net-tools

# Criar usuário postfix para comunicação entre os serviços
RUN groupadd -g 5000 postfix && useradd -u 5000 -g postfix postfix

# Criar os diretórios necessários para armazenamento dos emails
RUN mkdir -p /var/mail && \
    chown -R dovecot:dovecot /var/mail && \
    chmod -R 770 /var/mail

COPY dovecot.conf /etc/dovecot/dovecot.conf
COPY passdb /etc/dovecot/passdb
COPY userdb /etc/dovecot/userdb

# Copiar certificados
COPY ../certs/mail.infra.local.pem /etc/ssl/mail/mail.infra.local.pem
COPY ../certs/mail.infra.local-key.pem /etc/ssl/mail/mail.infra.local-key.pem

RUN chmod 600 /etc/dovecot/passdb /etc/dovecot/userdb

CMD ["dovecot", "-F"]