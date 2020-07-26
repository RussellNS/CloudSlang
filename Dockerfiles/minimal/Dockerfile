###########################################################################################
#                               CloudSlang (CS) - Minimal                                 #
###########################################################################################
# The intention of this container is to offer a minimal install of CS with updated        #
# CS content.  This 'minimal' install of CS includes:                                     #
#   * CS CLI (and content)                                                                #
#   * CS WebApp (for REST API)                                                            #
#   * Java (headless)                                                                     #
#   * SSH (Client)                                                                        #
###########################################################################################

# SET GLOBAL VARIABLE(S)
# NOTE: These global variables will be re-used as local variables in each stage of the
#       Docker multistage build process.  This way you only need to change a value in one
#       place.
#
# Use 'centos_version' to set the version of base OS from which to pull.  For reference,
# this container has been able to run successfully with:
#   *  "7.8.2003"
#   *  "8.2.2004"
#
# Use 'jdk_version' to set the version of Java JDK - valid values include:
#   *  "1.7.0" - CentOS 7
#   *  "1.8.0" - CentOS 7 & 8
#   NOTE:  Setting this value to "11" will cause the CS WebApp to build, but fail
#          to run (with error "Illegal reflective access" or
#          "nested exception is java.lang.NoClassDefFoundError").
#
# Use 'cs_cli_version' to set the version of CS to build.  For reference, this
# container has been able to run successfully with:
#   *  "1.0.10"
#   *  "1.0.23"
#   *  "1.0.161"
#
# The CS WebApp and the CS CLI are two completely different, stand-alone applications.
# The CS WebApp is a Spring Boot Web app that fully embeds a version of CS CLI inside
# it (as a Jar file).  Use 'cs_web_version' to set the version of CS CLI for the
# CS WebApp to embed during the CS WebApp build.  For reference, this container has
# been able to run successfully with:
#   *  "1.0.10"
#   *  "1.0.17"
#   *  "1.0.32"
#   *  "1.0.34" - Highest working version - Breaking change introduced in
#                 CS CLI v1.0.35
# NOTE: In order for a version to be elligable, it must be published to the Maven
#       repo for CloudSlang.  Not all versions are published.  Only use versions
#       that are published.  Check here to see:
#         * https://repo1.maven.org/maven2/io/cloudslang/lang/cloudslang-all/

ARG centos_version="7.4.1708"
ARG jdk_version="1.8.0"
ARG cs_cli_version="1.0.23"
ARG cs_web_version="1.0.10"

# NOTE: The ARG values above are default values.  You can override these values
#       with your own values.  If problems are encountered, then try reverting
#       back to these values.

FROM centos:${centos_version} AS builder

# SET LOCAL VARIABLE(S) FROM GLOBAL FOR MULTISTAGE BUILD
ARG jdk_version
ARG cs_cli_version
ARG cs_web_version

#COPY ./mf_content /media/mf_content

RUN echo "##### Creating Java (v${jdk_version}) Build Environment #####" \
  && yum install -y java-${jdk_version}-openjdk-devel git which unzip

RUN echo "##### Build CloudSlang CLI v${cs_cli_version} #####" \
  && echo "----- Download CloudSlang CLI v${cs_cli_version} -----" \
  && curl -SLo /media/cslang-cli.tar.gz "https://github.com/CloudSlang/cloud-slang/releases/download/cloudslang-${cs_cli_version}/cslang-cli.tar.gz" \
  && echo "----- Extract CloudSlang CLI v${cs_cli_version} -----" \
  && tar xf /media/cslang-cli.tar.gz -C /opt/ \
  && echo "----- Force 'Execute' Perms on Maven Scripts (1/2) -----" \
  && cd /opt/cslang-cli/maven/apache-maven-3.3.9/bin/ \
  && chmod a+x mvn mvnDebug mvnyjp

RUN echo "##### Updating CloudSlang CLI Property File(s) #####" \
  && echo "----- Change CloudSlang CLI Remote Repo - Maven 'http' is disabled, use 'https' -----" \
  && sed -i "s|http:|https:|" /opt/cslang-cli/configuration/cslang.properties \
  # The setting below shouldn't need to be done if the setting
  # ${cloudslang.maven.repo.local} is variablized correctly (which it
  # is...  I think).  Regardless, I'm not a Maven guy, it's not working,
  # and the setting below fixes (hard codes) it. \
  && echo "----- Change CloudSlang CLI Local Repo - set local repo in main Maven settings.xml -----" \
  && sed -i '/<!-- localRepository/i \\n\ \ <localRepository>/opt/cslang-cli/maven/repo</localRepository>' /opt/cslang-cli/maven/apache-maven-3.3.9/conf/settings.xml

RUN echo "##### Include Latest CloudSlang Content #####" \
  && echo "----- Download Latest CloudSlang Content -----" \
  && git clone --no-hardlinks https://github.com/CloudSlang/cs-content.git /media/cslang-content/ \
  && echo "----- Copy CloudSlang Content -----" \
  && /usr/bin/cp -rf /media/cslang-content/content/* /opt/cslang-cli/content/ \
  # Without the next line, the CS-WebApp breaks!  It has to do with a single
  # 'README.md' file that's in amazon/aws/lambda, but it could be any CS content
  # that's not a *.sl file.  In short, delete all files in 'content' that are
  # not *.sl files.
  && echo "----- Delete All Files That Are Not CloudSlang Content Files -----" \
  && find /opt/cslang-cli/content/ -type f ! -name '*.sl' -delete

RUN echo "##### Build CloudSlang WebApp v${cs_web_version} (the REST API) #####" \
  && echo "----- Download CloudSlang WebApp -----" \
  && git clone --no-hardlinks https://github.com/CloudSlang/cloudslang-webapp.git /media/cslang-webapp/ \
  && echo "----- Updating CloudSlang WebApp Version -----" \
  && sed -i "s|<cloudslang.version>1.0.10</cloudslang.version>|<cloudslang.version>${cs_web_version}</cloudslang.version>|" /media/cslang-webapp/pom.xml \
  && echo "----- Build CloudSlang WebApp ZIP -----" \
  && cd /media/cslang-webapp/ \
  && export JAVA_HOME=$(readlink -f $(which java) | sed 's|/bin/java||') \
  && /opt/cslang-cli/maven/apache-maven-3.3.9/bin/mvn clean install -B -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn \
  && echo "----- Extract CloudSlang WebApp ZIP -----" \
  && unzip -q /media/cslang-webapp/target/cloudslang-webapp-*.zip -d /opt/cslang-rest/ \
  && echo "----- Force 'Execute' Perms on Maven Scripts (2/2) -----" \
  && cd /opt/cslang-rest/maven/apache-maven-3.3.9/bin/ \
  && chmod a+x mvn mvnDebug mvnyjp

RUN echo "##### Updating CloudSlang WebApp Property File(s) #####" \
  && echo "----- Change CloudSlang WebApp Version - match CS CLI version -----" \
  && sed -i "s|slang.version=1.0.10|slang.version=${cs_web_version}|" /opt/cslang-rest/configuration/cslang.properties \
  && echo "----- Change CloudSlang WebApp Content Path - map to CS CLI content path -----" \
  && sed -i "s|content.path=.*|content.path=/opt/cslang-cli/content/|" /opt/cslang-rest/configuration/cslang.properties \
  && echo "----- Change CloudSlang WebApp Remote Repo - Maven 'http' is disabled, use 'https' only -----" \
  && sed -i "s|http:|https:|" /opt/cslang-rest/configuration/cslang.properties \
  && echo "----- Change CloudSlang WebApp Local Repo to CloudSlang CLI Local Repo (1/2) -----" \
  && sed -i "s|cloudslang.maven.repo.local=.*|cloudslang.maven.repo.local=/opt/cslang-cli/maven/repo|" /opt/cslang-rest/configuration/cslang.properties \
  # The setting below shouldn't need to be done if the setting
  # ${cloudslang.maven.repo.local} is variablized correctly (which it
  # is...  I think).  Regardless, I'm not a Maven guy, it's not working,
  # and the setting below fixes (hard codes) it. \
  && echo "----- Change CloudSlang WebApp Local Repo to CloudSlang CLI Local Repo (2/2) -----" \
  && sed -i '/<!-- localRepository/i \\n\ \ <localRepository>/opt/cslang-cli/maven/repo</localRepository>' /opt/cslang-rest/maven/apache-maven-3.3.9/conf/settings.xml

RUN echo "##### IMPORT CLOUDSLANG CONTENT PACK(S) - STARTING #####" \
  && if [ -d "/media/mf_content" ] \
   ; then \
       if [ $(ls /media/mf_content/*.jar  | wc -l) -ge 1 ] \
       ; then \
           mfContentPacks=$(ls /media/mf_content/*.jar) \
           && for cp in ${mfContentPacks} \
            ; do \
                mfCPfile=$(echo ${cp##*/}) \
                mfCPname=$(echo ${mfCPfile%.*}) \
                && echo "##### Import CS Content Pack - ${mfCPname} #####" \
                && echo "----- Extract CS Content Pack - ${mfCPname} -----" \
                && unzip -q ${cp} -d /media/mf_content/thisContentPack \
                && if [ -d "/media/mf_content/thisContentPack/Content/Library/io" ] \
                 ; then \
                     echo "----- Copy CS Flows & Operations - ${mfCPname} -----" \
                     && /usr/bin/cp -rf "/media/mf_content/thisContentPack/Content/Library/io" "/opt/cslang-cli/content/" \
                 ; fi \
                && if [ -d "/media/mf_content/thisContentPack/Content/Configuration/System Properties/io" ] \
                 ; then \
                     echo "----- Copy CS System Properties - ${mfCPname} -----" \
                     && /usr/bin/cp -rf "/media/mf_content/thisContentPack/Content/Configuration/System Properties/io" "/opt/cslang-cli/configuration/properties/" \
                 ; fi \
                && if [ -d "/media/mf_content/thisContentPack/Lib/io/cloudslang/content" ] \
                 ; then \
                     echo "----- Import CS Action(s) - ${mfCPname} -----" \
                     && mfCPjarsPath=$(find /media/mf_content/thisContentPack/Lib/io/cloudslang/content/ -type f -name "*.jar") \
                     && for cpJar in ${mfCPjarsPath} \
                      ; do \
                          cpJarFile=$(echo ${cpJar##*/}) \
                          cpJarName=$(echo ${cpJarFile%.*}) \
                          cpJarPath=$(echo ${cpJar%/*}) \
                          && if [ -f "${cpJarPath}/${cpJarName}.pom" ] \
                           ; then \
                               cpPom=${cpJarPath}/${cpJarName}.pom \
                               && echo "----- Import CS Action - ${mfCPname} -> ${cpJarName} -----" \
                               && /opt/cslang-cli/maven/apache-maven-3.3.9/bin/mvn install:install-file -Dmaven.repo.local=/opt/cslang-cli/maven/repo -Dfile=${cpJar} -DpomFile=${cpPom} -B -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn \
                           ; fi \
                      ; done \
                 ; fi \
                && echo "----- Import Clean-Up - ${mfCPname} -----" \
                && rm -rf /media/mf_content/thisContentPack \
            ; done \
       ; fi \
      && echo "##### IMPORT CLOUDSLANG CONTENT PACK(S) - ENDING #####" \
   ; else \
       echo "##### NO CLOUDSLANG CONTENT PACK(S) TO IMPORT - ENDING #####" \
   ; fi

RUN echo "##### Completing CloudSlang CLI and WebApp Build Process #####" \
  && echo "##### Starting CloudSlang CLI and WebApp Image/Container Creation #####"

###########################################################################################
#                               Base OS / Application Setup                               #
###########################################################################################
FROM centos:${centos_version}

# SET LOCAL VARIABLE(S) FROM GLOBAL FOR MULTISTAGE BUILD
ARG jdk_version

COPY --from=builder /opt/cslang-cli /opt/cslang-cli
COPY --from=builder /opt/cslang-rest /opt/cslang-rest

RUN echo "##### Create CloudSlang CLI/WebApp Container #####" \
  && echo "----- Install Required Files -----" \
  && yum install -y --setopt=tsflags=nodocs \
    java-${jdk_version}-openjdk-headless \
    which \
    openssh-clients \
  && echo "----- Create Universal JAVA_HOME Symbolic Link -----" \
  && ln -s $(readlink -f $(which java) | sed 's|/bin/java||') /usr/lib/jvm/java \
  && echo "----- Make CloudSlang CLI ('cslang') Runnable from Anywhere -----" \
  && ln -s /opt/cslang-cli/bin/cslang /usr/sbin/cslang \
  && echo "----- Add CloudSlang User & Group ('cslang') -----" \
  && groupadd -r cslang \
  && groupmod -g 2000 cslang \
  && useradd -r -g cslang cslang \
  && usermod -u 2000 cslang \
  && echo "----- Update Owner of CloudSlang CLI Directory -----" \
  && chown -R cslang:cslang /opt/cslang-cli \
  && echo "----- Update Owner of CloudSlang WebApp Directory -----" \
  && chown -R cslang:cslang /opt/cslang-rest

ENV JAVA_HOME=/usr/lib/jvm/java
ENV APP_HOME=/opt/cslang-rest

WORKDIR /opt/cslang-cli/bin/

EXPOSE 8443/tcp

ENTRYPOINT ["java", "-jar", "-Dapp.home=/opt/cslang-rest", "/opt/cslang-rest/cloudslang-webapp-0.0.1.jar"]
