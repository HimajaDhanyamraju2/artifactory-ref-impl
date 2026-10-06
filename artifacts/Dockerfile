# ============================================================
# STAGE 1 - Build Artifactory contents using Java 11
# ============================================================

FROM eclipse-temurin:11.0.32_9-jdk-jammy AS builder

ARG container_user=mosip
ARG container_user_group=mosip
ARG container_user_uid=1001
ARG container_user_gid=1001

ARG MAVEN_VERSION=3.8.8
ARG MAVEN_SHA512=332088670d14fa9ff346e6858ca0acca304666596fec86eea89253bd496d3c90deae2be5091be199f48e09d46cec817c6419d5161fb4ee37871503f472765d00

# Only build-time utilities.
# Java 11 is already provided by Eclipse Temurin.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        gnupg2 \
        unzip \
        wget \
        zip && \
    rm -rf /var/lib/apt/lists/*

# Install Maven 3.8.8 and verify checksum
RUN wget -q \
        https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
        -O /tmp/apache-maven.tar.gz && \
    echo "${MAVEN_SHA512}  /tmp/apache-maven.tar.gz" | sha512sum -c - && \
    mkdir -p /usr/local/maven && \
    tar -xzf /tmp/apache-maven.tar.gz \
        -C /usr/local/maven \
        --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz

ENV MAVEN_HOME=/usr/local/maven
ENV PATH="${MAVEN_HOME}/bin:${PATH}"

# Create MOSIP user
RUN groupadd -g ${container_user_gid} ${container_user_group} && \
    useradd \
        -u ${container_user_uid} \
        -g ${container_user_group} \
        -s /bin/bash \
        -m ${container_user}

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

# Create all required Artifactory paths
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

# ============================================================
# Copy artifact source files
# ============================================================

COPY src/sdk/biosdk/install.sh \
    ${biosdk_client_zip_path}/biosdk-client/

COPY src/sdk/demosdk/install.sh \
    ${demosdk_zip_path}/demosdk/

COPY src/deployment/docker/id-authentication/* \
    ${ida_sh_path}/

# HSM
# client-21.zip intentionally removed.
COPY src/hsm/client.zip \
    ${hsm_client_path}/

# MOSIP plugins
COPY src/mosip-plugins/sign-in-with-esignet/sign-in-with-esignet.zip \
    ${mosip_plugins_zip_path}/

# Testing
COPY src/testing/regproc-reprocessor-ceylon-cache-repo.zip \
    ${test_jar_path}/

COPY src/testing/glowroot.zip \
    ${test_jar_path}/

# i18n resources
COPY src/i18n/regclient-i18n-bundle/* \
    ${work_dir}/resources/

COPY src/i18n/admin-entity-spec-bundle/* \
    ${work_dir}/admin-entity-spec-bundle/

COPY src/i18n/admin-i18n-bundle/* \
    ${work_dir}/admin-i18n-bundle/

COPY src/i18n/pmp-entity-spec-bundle/* \
    ${work_dir}/pmp-entity-spec-bundle/

COPY src/i18n/pmp-i18n-bundle/* \
    ${work_dir}/pmp-i18n-bundle/

COPY src/i18n/pre-registration-i18n-bundle/* \
    ${work_dir}/pre-registration-i18n-bundle/

# Master templates
COPY src/master-templates/* \
    ${work_dir}/master-templates/

# IDP
COPY src/i18n/idp-i18n-bundle/* \
    ${work_dir}/idp-i18n-bundle/

# OIDC Demo
COPY src/i18n/oidc-demo-i18n-bundle/* \
    ${work_dir}/oidc-demo-i18n-bundle/

# Mock relying party
COPY src/i18n/mock-relying-party-i18n-bundle/* \
    ${work_dir}/mock-relying-party-i18n-bundle/

# eSignet
COPY src/i18n/esignet-i18n-bundle/* \
    ${work_dir}/esignet-i18n-bundle/

COPY src/theme/esignet-theme/* \
    ${work_dir}/esignet-theme/

COPY src/image/esignet-image/* \
    ${work_dir}/esignet-image/

# eSignet signup
COPY src/image/esignet-signup-image/* \
    ${work_dir}/esignet-signup-image/

COPY src/i18n/esignet-signup-i18n-bundle/* \
    ${work_dir}/esignet-signup-i18n-bundle/

COPY src/theme/esignet-signup-theme/* \
    ${work_dir}/esignet-signup-theme/

# Windows registration client JRE
COPY src/jre/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip \
    ${regclient_jar_path}/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip

# ICU4J
COPY src/icu4j/* \
    ${icu4j_jar_path}/

# Authentication libraries
COPY src/auth/* \
    ${regproc_jar_path}/

# Build configuration
COPY configure.sh \
    ${work_dir}/configure.sh

COPY pom.xml \
    ${work_dir}/pom.xml

# Verify tool versions before build
RUN java -version && \
    mvn -version

# Resolve and create Artifactory content
RUN cd ${work_dir} && \
    mvn -B dependency:resolve && \
    mvn -B clean install

# Generate/configure artifacts
RUN cd ${work_dir} && \
    bash ./configure.sh


# ============================================================
# STAGE 2 - Final Nginx runtime
# ============================================================

FROM nginx:stable-bookworm@sha256:552e7481ca93ffccd046aa658dbbed22caefbc09c66fa7cd247cbb90b8a5c609

ARG SOURCE
ARG COMMIT_HASH
ARG COMMIT_ID
ARG BUILD_TIME

LABEL source=${SOURCE}
LABEL commit_hash=${COMMIT_HASH}
LABEL commit_id=${COMMIT_ID}
LABEL build_time=${BUILD_TIME}

ARG container_user=mosip
ARG container_user_group=mosip
ARG container_user_uid=1001
ARG container_user_gid=1001

# Preserve environment variables used by existing deployments
ENV work_dir=/home/mosip
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

# Create non-root MOSIP user
RUN groupadd -g ${container_user_gid} ${container_user_group} && \
    useradd \
        -u ${container_user_uid} \
        -g ${container_user_group} \
        -s /bin/sh \
        -m ${container_user} && \
    mkdir -p \
        /var/run/nginx \
        /var/tmp/nginx \
        /usr/share/nginx/html/artifactory && \
    chown -R ${container_user}:${container_user} \
        /usr/share/nginx \
        /var/run/nginx \
        /var/tmp/nginx \
        /home/${container_user}

# Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy only completed artifacts from Java/Maven builder
COPY --from=builder \
    /usr/share/nginx/html/artifactory \
    /usr/share/nginx/html/artifactory

# Preserve generated files from /home/mosip as in the old image
COPY --from=builder \
    /home/mosip \
    /home/mosip

# Existing image expected Nginx to listen on 8080
RUN sed -i \
        -e '/listen/!b' \
        -e '/80;/!b' \
        -e 's/80;/8080;/' \
        /etc/nginx/conf.d/default.conf && \
    chown -R ${container_user}:${container_user} \
        /usr/share/nginx \
        /home/${container_user} \
        /var/run/nginx \
        /var/tmp/nginx

WORKDIR /home/${container_user}

USER ${container_user_uid}:${container_user_gid}

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
