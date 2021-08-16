ARG PYTHON_BASE_IMAGE="ruanroloff/python:3.8.11"
FROM ${PYTHON_BASE_IMAGE}

ARG AIRFLOW_VERSION="2.0.2"
ARG AIRFLOW_EXTRAS="async,amazon,cncf.kubernetes,docker,http,ldap,google,microsoft.azure,redis,sftp,ssh"
ARG ADDITIONAL_AIRFLOW_EXTRAS=""
ARG ADDITIONAL_PYTHON_DEPS=""
ARG AIRFLOW_DEPS=""
ARG PYTHON_DEPS=""


ARG AIRFLOW_HOME=/opt/airflow
ARG AIRFLOW_UID="50000"
ARG AIRFLOW_GID="50000"

#ARG AIRFLOW_USER_HOME=/usr/local/airflow
ARG AIRFLOW_USER_HOME=/home/airflow
ARG AIRFLOW_USER_HOME_DIR=/home/airflow

ARG AIRFLOW_PIP_VERSION=21.1.2

SHELL ["/bin/bash", "-o", "pipefail", "-e", "-u", "-x", "-c"]

#ARG AIRFLOW_UID
#ARG AIRFLOW_GID
COPY docker-context-files /docker-context-files
COPY scripts/docker/*.sh /scripts/docker/
COPY scripts/docker/install_mysql.sh scripts/docker/install_pip_version.sh /scripts/docker/


LABEL org.apache.airflow.distro="debian" \
  org.apache.airflow.module="airflow" \
  org.apache.airflow.component="airflow" \
  org.apache.airflow.image="airflow" \
  org.apache.airflow.uid="${AIRFLOW_UID}" \
  org.apache.airflow.gid="${AIRFLOW_GID}"


# Install curl and gnupg2 - needed for many other installation steps
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
           curl \
           gnupg2 \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ARG RUNTIME_APT_DEPS="\
       apt-transport-https \
       apt-utils \
       ca-certificates \
       curl \
       dumb-init \
       freetds-bin \
       gnupg \
       gosu \
       krb5-user \
       ldap-utils \
       libffi6 \
       libldap-2.4-2 \
       libsasl2-2 \
       libsasl2-modules \
       libssl1.1 \
       locales  \
       lsb-release \
       netcat \
       openssh-client \
       rsync \
       sasl2-bin \
       sqlite3 \
       sudo \
       unixodbc"

#RUN 
ARG DEBIAN_FRONTEND=noninteractive

ARG RUNTIME_APT_DEPS_LDAP="\
    build-essential python3-dev \
    libldap2-dev libsasl2-dev slapd ldap-utils tox \
    lcov valgrind"


# USERS
RUN addgroup --gid "${AIRFLOW_GID}" "airflow" && \
    adduser --quiet "airflow" --uid "${AIRFLOW_UID}" \
        --gid "${AIRFLOW_GID}" \
        --home "${AIRFLOW_USER_HOME_DIR}" && \
    mkdir -pv "${AIRFLOW_HOME}"; \
    mkdir -pv "${AIRFLOW_HOME}/dags"; \
    mkdir -pv "${AIRFLOW_HOME}/logs"; \
    chown -R "airflow:root" "${AIRFLOW_USER_HOME_DIR}" "${AIRFLOW_HOME}"; \
    find "${AIRFLOW_HOME}" -executable -print0 | xargs --null chmod g+x && \
        find "${AIRFLOW_HOME}" -print0 | xargs --null chmod g+rw

RUN set -ex \
    && buildDeps=' \
        git \
    ' \
    && apt-get update -yqq \
    && apt-get upgrade -yqq \
    && apt-get install -yqq --no-install-recommends \
        $buildDeps \
	$RUNTIME_APT_DEPS \
	$RUNTIME_APT_DEPS_LDAP \
        freetds-bin \
        apt-utils \
        curl \
        rsync \
        netcat \
        locales \
    && sed -i 's/^# en_US.UTF-8 UTF-8$/en_US.UTF-8 UTF-8/g' /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 \
#    && useradd -ms /bin/bash -d ${AIRFLOW_USER_HOME} airflow \
    && pip install -U pip setuptools wheel \
    && pip install pytz \
    && pip install pyOpenSSL \
    && pip install ndg-httpsclient \
    && pip install pyasn1 \
    && pip install apache-airflow[${AIRFLOW_EXTRAS}]==${AIRFLOW_VERSION} \
#    && pip install apache-airflow[${AIRFLOW_EXTRAS}]==${AIRFLOW_VERSION} --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.8.txt"\
    && pip install 'redis==3.2' \
#    && if [ -n "${PYTHON_DEPS}" ]; then pip install ${PYTHON_DEPS}; fi \
    && apt-get purge --auto-remove -yqq $buildDeps \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base

### INSTAL MYSQL
RUN apt-get update \
    && apt install default-libmysqlclient-dev -y \
    && apt install dirmngr -y \
    && rm -rf /var/lib/apt
RUN apt-get update && apt-get install -y mysql-client && rm -rf /var/lib/apt \	
	&& pip install mysqlclient==2.0.3
RUN pip install apache-airflow-providers-mysql==1.1.0

ENV AIRFLOW_HOME=/opt/airflow
ENV AIRFLOW_VERSION="2.0.2"
ENV AIRFLOW_USER_HOME=/home/airflow

COPY --chown=airflow:root scripts/in_container/prod/entrypoint_prod.sh /entrypoint
COPY --chown=airflow:root scripts/in_container/prod/clean-logs.sh /clean-logs

RUN chmod a+x /entrypoint /clean-logs && \
    chmod g=u /etc/passwd

WORKDIR ${AIRFLOW_HOME}

#EXPOSE 8080
EXPOSE 8080 5555 8793

RUN usermod -g 0 airflow -G ${AIRFLOW_GID}

USER ${AIRFLOW_UID}



LABEL org.apache.airflow.distro="debian" \
  org.apache.airflow.distro.version="buster" \
  org.apache.airflow.module="airflow" \
  org.apache.airflow.component="airflow" \
  org.apache.airflow.image="airflow" \
  org.apache.airflow.version="${AIRFLOW_VERSION}" \
  org.apache.airflow.uid="${AIRFLOW_UID}" \
  org.apache.airflow.gid="${AIRFLOW_GID}" \
  org.apache.airflow.main-image.build-id="${BUILD_ID}" \
  org.apache.airflow.main-image.commit-sha="${COMMIT_SHA}" \
  org.opencontainers.image.source="${AIRFLOW_IMAGE_REPOSITORY}" \
  org.opencontainers.image.created=${AIRFLOW_IMAGE_DATE_CREATED} \
  org.opencontainers.image.authors="dev@airflow.apache.org" \
  org.opencontainers.image.url="https://airflow.apache.org" \
  org.opencontainers.image.documentation="https://airflow.apache.org/docs/apache-airflow/stable/production-deployment.html" \
  org.opencontainers.image.source="https://github.com/apache/airflow" \
  org.opencontainers.image.version="${AIRFLOW_VERSION}" \
  org.opencontainers.image.revision="${COMMIT_SHA}" \
  org.opencontainers.image.vendor="Apache Software Foundation" \
  org.opencontainers.image.licenses="Apache-2.0" \
  org.opencontainers.image.ref.name="airflow" \
  org.opencontainers.image.title="Production Airflow Image" \
  org.opencontainers.image.description="Installed Apache Airflow"


ENTRYPOINT ["/usr/bin/dumb-init", "--", "/entrypoint"]
CMD []

