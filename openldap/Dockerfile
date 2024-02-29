FROM debian:12-slim AS base

ARG OPENLDAP_VERSION_BRANCH=OPENLDAP_REL_ENG_2_6

FROM base AS builder

WORKDIR /builder

RUN apt-get update && \
    apt-get install -y git build-essential autotools-dev libtool libevent-dev libargon2-dev libssl-dev libsasl2-dev groff-base libperl-dev unixodbc-dev

RUN git clone https://git.openldap.org/openldap/openldap.git --single-branch --branch ${OPENLDAP_VERSION_BRANCH} --depth 1

WORKDIR /builder/openldap

RUN ./configure --prefix=/opt/openldap \
    --enable-backends --disable-wt \
    --enable-modules --enable-argon2 \
    --disable-shared && \
    make depend -j$(nproc) && \
    make -j$(nproc) && \
    make install

FROM base AS dist

RUN apt-get update && \
    apt-get install -y libltdl7 libsasl2-2 perl unixodbc && \
    apt-get clean

COPY --from=builder /opt/openldap /opt/openldap

WORKDIR /opt/openldap

COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

ENV PATH="/opt/openldap/bin:/opt/openldap/sbin:$PATH"

EXPOSE 389 636
VOLUME /docker-entrypoint-initdb.d
VOLUME /opt/openldap/var/openldap-data

ENTRYPOINT ["/opt/openldap/entrypoint.sh"]
CMD ["/opt/openldap/libexec/slapd", "-d", "256"]
