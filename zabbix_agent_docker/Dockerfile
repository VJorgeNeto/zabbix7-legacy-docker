# ==============================================================================
# ESTÁGIO 1: Builder
# ==============================================================================
FROM almalinux:8 as builder

ARG ZABBIX_VERSION=7.4.5
ARG ZABBIX_URL=https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-${ZABBIX_VERSION}.tar.gz

# Instalação de dependências de build
RUN dnf install -y epel-release && \
    dnf config-manager --set-enabled powertools && \
    dnf install -y \
    gcc make automake autoconf \
    pcre2-devel \
    openssl-devel \
    libcurl-devel \
    libxml2-devel \
    net-snmp-devel \
    openldap-devel \
    libssh-devel \
    wget tar gzip

WORKDIR /usr/src/zabbix

RUN wget -q ${ZABBIX_URL} -O zabbix.tar.gz && \
    tar -xzf zabbix.tar.gz --strip-components=1

# --- MUDANÇA AQUI: --enable-agent ---
RUN ./configure \
    --enable-agent \
    --with-net-snmp \
    --with-libcurl \
    --with-libxml2 \
    --with-openssl \
    --with-libpcre2 \
    --with-ssh \
    --silent && \
    make -j$(nproc) install

# ==============================================================================
# ESTÁGIO 2: Runtime
# ==============================================================================
FROM almalinux:8-minimal

LABEL maintainer="Hitfy - SRE Team"

# Usuário Zabbix
RUN microdnf install -y shadow-utils && \
    groupadd --system zabbix && \
    useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin zabbix

# Instalar dependências de runtime
RUN microdnf install -y epel-release && \
    microdnf install -y \
    pcre2 \
    openssl-libs \
    libcurl \
    libxml2 \
    net-snmp-libs \
    libssh \
    # Ferramentas úteis para UserParameter
    iproute \
    procps-ng \
    && microdnf clean all

# --- MUDANÇA AQUI: Copiar o binário do Agente ---
COPY --from=builder /usr/local/sbin/zabbix_agentd /usr/sbin/
COPY --from=builder /usr/local/etc/zabbix_agentd.conf /etc/zabbix/

# Criar pastas
WORKDIR /var/lib/zabbix
RUN mkdir -p /var/lib/zabbix/enc /var/lib/zabbix/modules /etc/zabbix/zabbix_agentd.d && \
    chown -R zabbix:zabbix /var/lib/zabbix /etc/zabbix

EXPOSE 10050
VOLUME ["/etc/zabbix/zabbix_agentd.d"]

USER zabbix

CMD ["/usr/sbin/zabbix_agentd", "-c", "/etc/zabbix/zabbix_agentd.conf", "-f"]