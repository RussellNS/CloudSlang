###########################################################################################
#                                 CloudSlang - Full Dev                                   #
###########################################################################################
# The intention of this container is to offer a full CloudSlang development suite.  It    #
# includes many of the components that can be helpful for developing CloudSlang content   #
# or for connecting/troubleshooting including:                                            #
#   * CloudSlang CLI (and content)                                                        #
#   * CloudSlang WebApp (for REST API)                                                    #
#   * Java (headless)                                                                     #
#   * SSH (Server)                                                                        #
#   * SSH (Client)                                                                        #
#   * Git                                                                                 #
#   * Python                                                                              #
#   * Pip                                                                                 #
#   * Editors                                                                             #
#       - Nano                                                                            #
#       - Vim                                                                             #
#       - Emacs                                                                           #
#   * IDE                                                                                 #
#       - Atom (with CloudSlang Atom Package)                                             #
###########################################################################################
#                                     Additional Info                                     #
###########################################################################################
# This container goes against some Docker best practices.  For instance:                  #
#   *  Primarily, this container is designed to run as a VM.                              #
#         o  More specifically, it's a Docker "anti-pattern" that includes the running of #
#            multiple services at the same time (including the SSH daemon, "sshd", to     #
#            ensure the container stays running and accessible via SSH, even when it's    #
#            not doing anything).                                                         #
#   *  This container updates the underlying OS packages to the latest version, which can #
#      break things.                                                                      #
###########################################################################################
#                                          TO DO                                          #
###########################################################################################
# 1) Incorporate latest 'cs-actions' on GitHub.                                           #
#      * https://github.com/CloudSlang/cs-actions.                                        #
#          o This is where MF actively develops new CS content                            #
#          o The content here is days or even hours old                                   #
#          o Development is *very* live here                                              #
#          o Including this could break things                                            #
#          o Will require logic to:                                                       #
#              - exclude active development versions (beta, SNAPSHOT, RC##, etc.)         #
#              - determine/clone the latest stable version                                #
#          o After that, it's just 'mvn clean install'                                    #
#          o Is this even worth it?                                                       #
#          o Expose as a build ARG maybbe, user can choose?                               #
###########################################################################################

# SET GLOBAL VARIABLE(S)
# NOTE: These global variables will be re-used as local variables in each stage of the
#       Docker multistage build process.  This way you only need to change a value in one
#       place.
#
# Use 'centos_version' to set the version of base OS from which to pull.  For reference,
# this container has been able to run successfully with:
#   *  "7.9.2009" - NOTE: The Atom Editor will not install successfully with CentOS 7.x.
#                         You will have to comment the atom install lines manually.
#   *  "8.3.2011"
#
# Use 'jdk_version' to set the version of Java JDK - valid values include:
#   *  "1.7.0" - CentOS 7
#   *  "1.8.0" - CentOS 7 & 8
#   *  "11"    - CentOS 7 & 8
#
# Use 'cs_cli_version' to set the version of CS to build.  For reference, this
# container has been able to run successfully with:
#   *  "1.0.10"
#   *  "1.0.23"
#   *  "2.0.205"
#
# The CS WebApp and the CS CLI are two completely different, stand-alone applications.
# The CS WebApp is a Spring Boot Web app that fully embeds a version of CS CLI inside
# it (as a Jar file).  Use 'cs_web_version' to set the version of CS CLI for the
# CS WebApp to embed during the CS WebApp build.  The reason to separate the versions
# in this Dockerfile for the CS CLI and the CS WebApp is because during the development
# of the CS CLI, breaking changes are introduced that cause the CS WebApp not to build.
# This allows for both apps to build using separate versions.  For reference, this
# container has been able to run successfully with:
#   *  "1.0.10"
#   *  "1.0.23"
#   *  "2.0.205"
#
# NOTE: In order for a version to be elligable, it must be published to the Maven
#       repo for CloudSlang.  Not all versions are published.  Only use versions
#       that are published.  Check here to see:
#         * https://repo1.maven.org/maven2/io/cloudslang/lang/cloudslang-all/

ARG centos_version="latest"
ARG jdk_version="11"
ARG cs_cli_version="2.0.205"
ARG cs_web_version="2.0.205"

# NOTE: The ARG values above are default values.  You can override these values
#       with your own values using "docker run" or in a "docker-compose.yml" file.
#       If problems are encountered, then try reverting back to these values.

FROM centos:${centos_version} AS builder

# SET LOCAL VARIABLE(S) FROM GLOBAL FOR MULTISTAGE BUILD
ARG jdk_version
ARG cs_cli_version
ARG cs_web_version

#COPY ./mf_content /media/mf_content

# DEBUG - The 'RUN' line below causes the Docker build process to pause
# DEBUG - indefinitely.  This can be used to expand on Docker build logic or
# DEBUG - to identify and troubleshoot Docker build problems.  Perform these
# DEBUG - steps to use:
# DEBUG -   1) Copy/paste the 'RUN' line to right before any build errors.
# DEBUG -   2) Use 'docker run' to start the build process.
# DEBUG -   3) When the build pauses, press CRTL+z (not CTRL+c).
# DEBUG -   4) Enter 'docker container ls' to view container name that's
# DEBUG -      building.
# DEBUG -   5) Use 'docker exec -it <container_name> sh' (or 'bash') to enter
# DEBUG -      unfinished build.
# DEBUG -   6) Manually run your commands inside the container to see what
# DEBUG -      works or to troubleshoot errors.
# DEBUG - Comment or delete when not needed.
# DEBUG - NOTE: If CTRL+z does not work, open another SSH session, and
# DEBUG -       continue the steps.
#RUN echo "##### DEBUG - Docker build pause #####" && echo "Use the following commands:" && echo "    CTRL+z" && echo "    docker container ls # get container name" && echo "    docker exec -it <container_name> [sh|bash]" && echo "    Run your commands during Docker build" && tail -f /dev/null

RUN echo "##### Creating Java (v${jdk_version}) Build Environment #####" \
  && yum install -y \
    java-${jdk_version}-openjdk-devel \
    git \
    which \
    unzip

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
                     && /usr/bin/cp -rf "/media/mf_content/thisContentPack/Content/Library/io/*" "/opt/cslang-cli/content/io/" \
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
                               #&& /opt/cslang-cli/maven/apache-maven-3.3.9/bin/mvn install:install-file -Dmaven.repo.local=/opt/cslang-cli/maven/repo -Dfile=${cpJar} -DpomFile=${cpPom} \
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

###########################################################################################
# NOTE: The following RUN command has been created with the following criteria            #
# in mind:                                                                                #
#   *  Per Docker best practice, this RUN command creates as few additional               #
#      layers as possible.                                                                #
#   *  It's documented such that future changes to any part can be more easily            #
#      evaluated.                                                                         #
###########################################################################################
RUN echo "##### Starting Base OS/Environment Install #####"                               \
    #                                                                                     \
    # BASE OS: Install Misc Packages                                                      \
    && echo "##### Install Misc Packages #####"                                           \
    && yum install -y --setopt=tsflags=nodocs                                             \
        dnf                                                                               \
        net-tools                                                                         \
        nmap                                                                              \
        tree                                                                              \
        which                                                                             \
        unzip                                                                             \
    #                                                                                     \
    # CS OPTIONAL: Install EPEL Repo                                                      \
    && echo "##### Install EPEL Repo #####"                                               \
    && dnf install -y --setopt=tsflags=nodocs                                             \
        epel-release                                                                      \
    #                                                                                     \
    # CS OPTIONAL: Install SSH packages                                                   \
    && echo "##### Install SSH (Client & Server) #####"                                   \
    && dnf install -y --setopt=tsflags=nodocs                                             \
        openssh-server                                                                    \
        openssh-clients                                                                   \
        passwd                                                                            \
    #                                                                                     \
    # CS OPTIONAL: Configure SSH (with recommendations for SSH from Docker)               \
    #   *  https://docs.docker.com/engine/examples/running_ssh_service/                   \
    # NOTE: This sets the password for the 'root' account to 'cslang'.                    \
    && echo "##### Configure SSH Server - From Docker #####"                              \
    && mkdir /var/run/sshd                                                                \
    && echo "root:cslang" | chpasswd                                                      \
    && sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd \
    #                                                                                     \
    # CS OPTIONAL: Configure SSH (with Cloudslang & MF SA recommendations)                \
    && echo "##### Configure SSH Server - From CloudSlang and SA #####"                   \
    && /usr/bin/ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''                      \
    #&& /usr/bin/ssh-keygen -t rsa -f /etc/ssh/ssh_host_key -N ''                         \
    #&& /usr/bin/ssh-keygen -t rsa -f /etc/ssh/ssh_host_dsa_key -N ''                     \
    #&& /usr/bin/ssh-keygen -t rsa -f /etc/ssh/ssh_host_ecdsa_key -N ''                   \
    #&& /usr/bin/ssh-keygen -t rsa -f /etc/ssh/ssh_host_ed25519_key -N ''                 \
    && echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config                             \
    && echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config                             \
    && sed -i 's/without-password/yes/g' /etc/ssh/sshd_config                             \
    #                                                                                     \
    # CS OPTIONAL: Install Editors                                                        \
    && echo "##### Install Editors #####"                                                 \
    && echo "----- Install Nano, VIM, & Emacs -----"                                      \
    && dnf install -y --setopt=tsflags=nodocs                                             \
        nano                                                                              \
        vim                                                                               \
        emacs                                                                             \
    #                                                                                     \
    # CS OPTIONAL: Install Atom                                                           \
    #              NOTE: This stopped working in recent versions of Atom.                 \
    #                    Haven't fixed yet.                                               \
    && echo "----- Install Atom -----"                                                    \
    && curl -SLo /media/atom.rpm https://atom.io/download/rpm                             \
    && dnf localinstall -y --nogpgcheck /media/atom.rpm                                   \
    && rm -f /media/atom.rpm                                                              \
    #                                                                                     \
    # CS OPTIONAL: Configure Atom                                                         \
    && apm install language-cloudslang                                                    \
    #                                                                                     \
    # CS OPTIONAL: Upgrade CentOS & Clean Yum/DNF Cache                                   \
    #                                                                                     \
    # BASE OS: Upgrade CentOS Container                                                   \
    #          THIS IS NOT DOCKER BEST PRACTICE, Because:                                 \
    #            * THIS CAN BREAK THINGS                                                  \
    #              Changing the versions of components inside a Docker image WILL         \
    #              introduce changes in software inside the image, even if the change     \
    #              is an upgrade.  These changes have the possibility of introducing      \
    #              breaking changes to your image/container.  The Docker best             \
    #              practice is to hard code versions, and test upgrades by creating       \
    #              new Dockerfiles with upgraded (but still hard coded) versions.  If     \
    #              the new image tests & works well, then that becomes the 'upgraded'     \
    #              docker image/container.                                                \
    #            * THIS INCREASES IMAGE SIZE                                              \
    #              This is a little less intuitive, but Docker images are made up of      \
    #              read-only layers.  Upgrading components in later layers and            \
    #              deleting previously installed versions of these components only        \
    #              deletes the pointer to the file in a later layer.  It does not         \
    #              actually delete the file in a previous layer.  In this way, your       \
    #              final docker image file size will only grow... dramatically.           \
    && yum upgrade -y --setopt=tsflags=nodocs                                             \
    && yum clean all                                                                      \
    && rm -rf /var/cache/yum                                                              \
    && rm -rf /var/cache/dnf

###########################################################################################
#                                    Application Setup                                    #
###########################################################################################

RUN echo "##### Prep Container for Docker COPY #####" \
  && echo "----- Add CloudSlang User & Group ('cslang') -----" \
  && groupadd -r cslang \
  && groupmod -g 2000 cslang \
  && useradd -r -g cslang cslang \
  && usermod -u 2000 cslang

COPY --from=builder /opt/cslang-cli /opt/cslang-cli
COPY --from=builder /opt/cslang-rest /opt/cslang-rest

###########################################################################################
# NOTE: The following RUN command has been created with the following criteria            #
# in mind:                                                                                #
#   *  Per Docker best practice, this RUN command creates as few additional               #
#      layers as possible.                                                                #
#   *  It's documented such that future changes to any part can be more easily            #
#      evaluated.                                                                         #
#   *  It uses '&&' and '||' (posix AND_IF and OR_IF) grammar in order to perform         #
#      commands as part of a conditional.  Example:                                       #
#        o  command one && command two || command three && command four                   #
#             -  If 'command one' succeeds, then 'command two' will run, and              #
#                'command three' will be ignored.  Assuming 'command two' completes       #
#                successfully, then 'command four' will pick up, and everything past      #
#                (command five, six, whatever) will run normally.                         #
#             -  If 'command one' fails, then 'command two' will be ignored, and          #
#                'command three' will run.  Assuming 'command three' completes            #
#                successfully, then 'command four' will pick up, and everything past      #
#                (command five, six, whatever) will run normally.                         #
#             -  If 'command one' fails, then 'command three' also fails, 'command four'  #
#                will not continue, and the Dockerfile build will not complete.           #
#             -  It works.  Trust me.                                                     #
#        o  This construct is required in order to use this Dockerfile for both           #
#           CentOS v7.x and CentOS v8.x, as there are package differences for both        #
#           Python and Pip.                                                               #
###########################################################################################
RUN echo "##### Starting CloudSlang Setup #####"                                          \
    #                                                                                     \
    # CS REQUIRED: TAR - Install CloudSlang Dependencies                                  \
    && echo "##### Install Java (v${jdk_version}) #####"                                  \
    && yum install -y --setopt=tsflags=nodocs                                             \
        java-${jdk_version}-openjdk-headless                                              \
    #                                                                                     \
    # CS OPTIONAL: Create Link for Universal 'JAVA_HOME' Environment Variable             \
    # This should work for any Java package of any version.                               \
    # Remember to set 'ENV JAVA_HOME=/usr/lib/jvm/java/jre' somewhere in your Dockerfile. \
    # NOTE: This assumes you're not installing multiple versions of Java in 1 container.  \
    && echo "##### Create Universal JAVA_HOME Symbolic Link #####"                        \
    && ln -s $(readlink -f $(which java) | sed 's|/bin/java||') /usr/lib/jvm/java         \
    #                                                                                     \
    # CS OPTIONAL: Make CloudSlang runnable from anywhere                                 \
    && echo "##### Make CloudSlang ('cslang') Runnable from Anywhere #####"               \
    && ln -s /opt/cslang-cli/bin/cslang /usr/sbin/cslang                                  \
    #                                                                                     \
    # CS OPTIONAL: Install Git                                                            \
    && echo "##### Install Git #####"                                                     \
    && yum install -y --setopt=tsflags=nodocs                                             \
        git                                                                               \
    #                                                                                     \
    # CS OPTIONAL: Install Python                                                         \
    && echo "##### Install Python #####"                                                  \
    && yum install -y --setopt=tsflags=nodocs                                             \
        python36                                                                          \
    #                                                                                     \
    # CS OPTIONAL: Install Pip                                                            \
    # CentOS 7.x with Python2 uses 'python-pip'                                           \
    # CentOS 8.x with python3 uses 'platform-python-pip'                                  \
    && echo "##### Install Pip #####"                                                     \
    && yum install -y --setopt=tsflags=nodocs                                             \
        python3-pip                                                                       \
    || yum install -y --setopt=tsflags=nodocs                                             \
        platform-python-pip                                                               \
    #                                                                                     \
    # CS OPTIONAL: Upgrade PIP                                                            \
    && echo "##### Upgrade Pip #####"                                                     \
    && pip3 install --upgrade pip                                                         \
    #                                                                                     \
    # CS OPTIONAL: Install Python API for Jenkins (which is Java centric)                 \
    && echo "##### Install Python API for Jenkins #####"                                  \
    && pip3 install -r /opt/cslang-cli/python-lib/requirements.txt -t /opt/cslang-cli/python-lib/ \
    #                                                                                     \
    # CS OPTIONAL: Clean Yum Cache                                                        \
    && yum clean all                                                                      \
    && rm -rf /var/cache/yum


ENV JAVA_HOME=/usr/lib/jvm/java
ENV APP_HOME=/opt/cslang-rest

# HEALTHCHECK will repeatedly run a command inside a container to determine if a container
# is still functioning.  The command is anything, but must exit in 0 (healthy) or non-zero.
# There are 4 arguments that can also be passed to control when a health check starts, how
# often it is run, and how long the command should be allowed to run before a timeout.
# Usage:
#   *  HEALTHCHECK CMD curl -f http://localhost/ || exit 1
#   *  HEALTHCHECK --interval=1m --start-period=30s CMD curl -f http://localhost/ || exit 1
#   *  HEALTHCKECK NONE
#      NOTE: HEALTHCHECK NONE will disable healthchecks inherited from the base images.
#
# Arguments:
#   *  --interval=DURATION (default: 30s)
#        This is the frequency that a health check is run.  The default is run every 30
#        seconds.
#
#   *  --timeout=DURATION (default: 30s)
#        This is how long the command is allowed to run before the health check fails.
#
#   *  --start-period=DURATION (default: 0s)
#        This is the time to allow the container to start running before running the first
#        health check.
#
#   *  --retries=N (default: 3)
#        This is the number of retries to re-run a health check before failing.
HEALTHCHECK --interval=10m --timeout=30s --start-period=30s --retries=3 CMD /usr/sbin/cslang date

WORKDIR /opt/cslang-cli/bin/

EXPOSE 8443/tcp
EXPOSE 22/tcp

# Run SSH daemon
#   *  "-D" - do not detach
#   *  "-d" - do not detach with debug logging
#   *  "-e" - log to stderr
ENTRYPOINT /usr/sbin/sshd -e ; java -jar -Dapp.home=/opt/cslang-rest /opt/cslang-rest/cloudslang-webapp-2.0.0.jar
