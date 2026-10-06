FROM nginx:stable-bookworm

ARG SOURCE
ARG COMMIT_HASH
ARG COMMIT_ID
ARG BUILD_TIME

LABEL source=${SOURCE}
LABEL commit_hash=${COMMIT_HASH}
LABEL commit_id=${COMMIT_ID}
LABEL build_time=${BUILD_TIME}

# Container user configuration
ARG container_user=mosip
ARG container_user_group=mosip
ARG container_user_uid=1001
ARG container_user_gid=1001

# Install packages, Java, Maven and create user
RUN rm -rf /var/lib/apt/lists/* && \
    apt-get clean && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        gnupg2 \
        unzip \
        wget \
        zip \
        openjdk-11-jdk && \
    groupadd -g ${container_user_gid} ${container_user_group} && \
    wget https://archive.apache.org/dist/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz && \
    tar -xzf apache-maven-3.8.8-bin.tar.gz && \
    mv apache-maven-3.8.8 /usr/local/maven && \
    rm apache-maven-3.8.8-bin.tar.gz && \
    useradd \
        -u ${container_user_uid} \
        -g ${container_user_group} \
        -s /bin/sh \
        -m ${container_user} && \
    mkdir -p /var/run/nginx /var/tmp/nginx && \
    chown -R ${container_user}:${container_user} \
        /usr/share/nginx \
        /var/run/nginx \
        /var/tmp/nginx && \
    rm -rf /var/lib/apt/lists/*

# Architecture-independent JAVA_HOME
RUN JAVA_PATH="$(dirname "$(dirname "$(readlink -f "$(which java)")")")" && \
    ln -s "${JAVA_PATH}" /opt/java-home

COPY ./nginx.conf /etc/nginx/nginx.conf

ENV MAVEN_HOME=/usr/local/maven
ENV JAVA_HOME=/opt/java-home
ENV PATH="${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${PATH}"

WORKDIR /home/${container_user}

ENV work_dir=/home/${container_user}

ENV base_path=/usr/share/nginx/html/artifactory

ENV biosdk_client_zip_path=${base_path}/libs-release-local/biosdk/mock/0.9
ENV demosdk_zip_path=${base_path}/libs-release-local/demosdk/default/
ENV biosdk_lib_zip_path=${base_path}/libs-release-local/biosdk/
ENV jpegsdk_lib_zip_path=${base_path}/libs-release-local/jpeg-sdk/
ENV cache_path=${base_path}/libs-release-local/cache
ENV ida_sh_path=${base_path}/libs-release-local/deployment/docker/id-authentication
ENV hsm_client_path=${base_path}/libs-release-local/hsm
ENV mosip_plugins_zip_path=${base_path}/libs-release-local/mosip-plugins
ENV sdk_path=${base_path}/libs-release-local/mock-sdk/1.1.5/
ENV jpeg_sdk_path=${base_path}/libs-release-local/jpeg-mock-sdk/1.1.5/
ENV image_compressor_path=${base_path}/libs-release-local/compressor/
ENV kernel_jar_path=${base_path}/libs-release-local/io/mosip/kernel
ENV test_jar_path=${base_path}/libs-release-local/io/mosip/testing/
ENV idobject_jar_path=${base_path}/libs-release-local/io/mosip/kernel/kernel-ref-idobjectvalidator
ENV regproc_jar_path=${base_path}/libs-release-local/io/mosip/kernel/kernel-auth-adapter
ENV icu4j_jar_path=${base_path}/libs-release-local/icu4j
ENV i18n_zip_path=${base_path}/libs-release-local/i18n
ENV theme_zip_path=${base_path}/libs-release-local/theme
ENV image_zip_path=${base_path}/libs-release-local/image
ENV idp_auth_wrapper_lib_zip_path=${base_path}/libs-release-local/idp/idp-auth-wrapper
ENV certify_plugin_zip_path=${base_path}/libs-release-local/certify
ENV esignet_wrapper_lib_zip_path=${base_path}/libs-release-local/esignet
ENV clamav_path=${base_path}/libs-release-local/clamav
ENV regclient_jar_path=${base_path}/libs-release-local
ENV master_template_path=${base_path}/libs-release-local/master-templates
ENV child_auth_filter_jar_path=${base_path}/libs-release-local/io/mosip/authentication/authentication-ref-impl
ENV scripts_path=/home/mosip/scripts

ENV version=1.2.1-SNAPSHOT
ENV idp_auth_wrapper_version=0.0.1-SNAPSHOT

# Create required directories
RUN mkdir -p \
    ${biosdk_client_zip_path}/biosdk-client \
    ${demosdk_zip_path}/demosdk \
    ${biosdk_lib_zip_path}/biosdk-lib \
    ${cache_path} \
    ${ida_sh_path} \
    ${hsm_client_path} \
    ${mosip_plugins_zip_path} \
    ${kernel_jar_path} \
    ${test_jar_path} \
    ${idobject_jar_path} \
    ${regproc_jar_path} \
    ${sdk_path} \
    ${image_compressor_path}/image-compressor \
    ${icu4j_jar_path} \
    ${clamav_path} \
    ${i18n_zip_path} \
    ${theme_zip_path} \
    ${image_zip_path} \
    ${child_auth_filter_jar_path} \
    ${base_path}/libs-release-local/reg-client \
    ${scripts_path} \
    ${master_template_path} \
    ${jpeg_sdk_path} \
    ${jpegsdk_lib_zip_path}/jpeg-sdk-lib/ \
    ${idp_auth_wrapper_lib_zip_path} \
    ${esignet_wrapper_lib_zip_path}/esignet-wrapper \
    ${certify_plugin_zip_path}/certify-plugin

# SDK files
COPY /src/sdk/biosdk/install.sh \
    ${biosdk_client_zip_path}/biosdk-client/

COPY /src/sdk/demosdk/install.sh \
    ${demosdk_zip_path}/demosdk/

COPY /src/deployment/docker/id-authentication/* \
    ${ida_sh_path}/

# HSM client
COPY /src/hsm/client.zip \
    ${hsm_client_path}/

# MOSIP plugins
COPY /src/mosip-plugins/sign-in-with-esignet/sign-in-with-esignet.zip \
    ${mosip_plugins_zip_path}/

# Testing
COPY /src/testing/regproc-reprocessor-ceylon-cache-repo.zip \
    ${test_jar_path}/

COPY /src/testing/glowroot.zip \
    ${test_jar_path}/

# i18n resources
COPY /src/i18n/regclient-i18n-bundle/* \
    ${work_dir}/resources/

COPY /src/i18n/admin-entity-spec-bundle/* \
    ${work_dir}/admin-entity-spec-bundle/

COPY /src/i18n/admin-i18n-bundle/* \
    ${work_dir}/admin-i18n-bundle/

COPY /src/i18n/pmp-entity-spec-bundle/* \
    ${work_dir}/pmp-entity-spec-bundle/

COPY /src/i18n/pmp-i18n-bundle/* \
    ${work_dir}/pmp-i18n-bundle/

COPY /src/i18n/pre-registration-i18n-bundle/* \
    ${work_dir}/pre-registration-i18n-bundle/

COPY /src/master-templates/* \
    ${work_dir}/master-templates/

COPY /src/i18n/idp-i18n-bundle/* \
    ${work_dir}/idp-i18n-bundle/

COPY /src/i18n/oidc-demo-i18n-bundle/* \
    ${work_dir}/oidc-demo-i18n-bundle/

COPY /src/i18n/mock-relying-party-i18n-bundle/* \
    ${work_dir}/mock-relying-party-i18n-bundle/

COPY /src/i18n/esignet-i18n-bundle/* \
    ${work_dir}/esignet-i18n-bundle/

COPY /src/theme/esignet-theme/* \
    ${work_dir}/esignet-theme/

COPY /src/image/esignet-image/* \
    ${work_dir}/esignet-image/

COPY /src/image/esignet-signup-image/* \
    ${work_dir}/esignet-signup-image/

COPY /src/i18n/esignet-signup-i18n-bundle/* \
    ${work_dir}/esignet-signup-i18n-bundle/

COPY /src/theme/esignet-signup-theme/* \
    ${work_dir}/esignet-signup-theme/

# Windows JRE dependency
COPY /src/jre/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip \
    ${regclient_jar_path}/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip

# ICU4J
COPY /src/icu4j/* \
    ${icu4j_jar_path}/

# Auth libraries
COPY /src/auth/* \
    ${regproc_jar_path}/

# Configuration
COPY ./configure.sh \
    ${work_dir}/

COPY pom.xml \
    ${work_dir}/

# Maven build
RUN cd ${work_dir} && \
    mvn dependency:resolve && \
    mvn clean install

# Configure artifacts
RUN bash ${work_dir}/configure.sh

# Change nginx port from 80 to 8080
RUN sed -i \
    -e '/listen/!b' \
    -e '/80;/!b' \
    -e 's/80;/8080;/' \
    /etc/nginx/conf.d/default.conf

# Permissions
RUN chown -R ${container_user}:${container_user} \
    /home/${container_user} \
    /usr/share/nginx \
    /var/run/nginx \
    /var/tmp/nginx

USER ${container_user_uid}:${container_user_gid}

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
