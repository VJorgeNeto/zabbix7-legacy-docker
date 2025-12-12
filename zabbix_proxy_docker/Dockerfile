# ==============================================================================
# ESTÁGIO 1: Builder
# ==============================================================================
FROM almalinux:8 as builder

ARG ZABBIX_VERSION=7.4.5
ARG ZABBIX_URL=https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-${ZABBIX_VERSION}.tar.gz

# Instalando ferramentas e habilitando repositórios necessários
RUN dnf install -y epel-release && \
    dnf config-manager --set-enabled powertools && \
    dnf install -y \
    gcc make automake autoconf \
    sqlite-devel \
    libxml2-devel \
    libcurl-devel \
    libevent-devel \
    pcre2-devel \
    openssl-devel \
    net-snmp-devel \
    openldap-devel \
    libssh-devel \
    wget tar gzip

WORKDIR /usr/src/zabbix

RUN wget -q ${ZABBIX_URL} -O zabbix.tar.gz && \
    tar -xzf zabbix.tar.gz --strip-components=1

RUN ./configure \
    --enable-proxy \
    --with-sqlite3 \
    --with-net-snmp \
    --with-libcurl \
    --with-libxml2 \
    --with-openssl \
    --with-libevent \
    --with-libpcre2 \
    --with-ssh \
    --silent && \
    make -j$(nproc) install

# ==============================================================================
# ESTÁGIO 2: Runtime
# ==============================================================================
FROM almalinux:8-minimal

LABEL maintainer="Hitfy - SRE Team"

# Criar usuário Zabbix
RUN microdnf install -y shadow-utils && \
    groupadd --system zabbix && \
    useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin zabbix

# CORREÇÃO AQUI:
# 1. Instalamos o epel-release PRIMEIRO
# 2. Depois instalamos o fping e as outras libs
RUN microdnf install -y epel-release && \
    microdnf install -y \
    sqlite-libs \
    libxml2 \
    libcurl \
    libevent \
    pcre2 \
    openssl-libs \
    net-snmp-libs \
    libssh \
    fping \
    python3 \
    python3-pip \
    && microdnf clean all

# Copiar os binários compilados
COPY --from=builder /usr/local/sbin/zabbix_proxy /usr/sbin/
COPY --from=builder /usr/local/etc/zabbix_proxy.conf /etc/zabbix/

WORKDIR /var/lib/zabbix
RUN mkdir -p /var/lib/zabbix/enc /var/lib/zabbix/modules /usr/lib/zabbix/externalscripts && \
    chown -R zabbix:zabbix /var/lib/zabbix /etc/zabbix /usr/lib/zabbix

EXPOSE 10051
VOLUME ["/var/lib/zabbix", "/usr/lib/zabbix/externalscripts"]

USER zabbix

CMD ["/usr/sbin/zabbix_proxy", "-c", "/etc/zabbix/zabbix_proxy.conf", "-f"]