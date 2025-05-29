FROM debian:latest

RUN apt-get update && \
    apt-get install -y postfix sasl2-bin && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -y procps

RUN apt-get update && apt-get install -y rsyslog

RUN apt-get update && apt-get install -y telnet

# Criar os diretórios necessários para o Postfix
RUN mkdir -p /var/spool/postfix/public /var/spool/postfix/private && \
    mkdir -p /var/mail && \
    chown -R postfix:postfix /var/spool/postfix && \
    chown -R postfix:postfix /var/mail && \
    chmod -R 755 /var/spool/postfix && \
    chmod -R 770 /var/mail

# Copiar configurações do Postfix
COPY main.cf /etc/postfix/main.cf
COPY master.cf /etc/postfix/master.cf
COPY smtpd.conf /etc/postfix/sasl/smtpd.conf
COPY users /etc/postfix/users

# Copiar certificados
COPY ../certs/mail.infra.local.pem /etc/ssl/mail/mail.infra.local.pem
COPY ../certs/mail.infra.local-key.pem /etc/ssl/mail/mail.infra.local-key.pem

# Garantir que arquivos sensíveis tenham as permissões corretas
RUN chmod 644 /etc/postfix/sasl/smtpd.conf && \
    chmod 644 /etc/postfix/users && \
    chown root:root /etc/postfix/sasl/smtpd.conf /etc/postfix/users

RUN postmap /etc/postfix/users

# Definir permissões do Postfix na inicialização
RUN postfix set-permissions && postfix check

# Comando para iniciar o Postfix em primeiro plano
CMD ["postfix", "start-fg"]