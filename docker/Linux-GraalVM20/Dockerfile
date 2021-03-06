# (This is Oracle Linux)
FROM oracle/graalvm-ce:20.1.0-java11

ENV M2_HOME="/opt/maven"
ENV PATH="${M2_HOME}/bin:${PATH}"

# Install wget
RUN yum -y -q install wget

# Install repo for FFmpeg
RUN wget https://yum.oracle.com/public-yum-ol7.repo -O /etc/yum.repos.d/public-yum-ol7.repo

RUN yum -y -q install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
        && yum  -y -q localinstall --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm \
                https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-7.noarch.rpm

# Install various dependencies
RUN yum -y -q update && yum -y -q install \
        curl \
		ffmpeg \
		openjpeg2-tools

RUN ln -s /usr/bin/opj2_decompress /usr/bin/opj_decompress

# Install Maven
RUN cd /opt \
        && wget -q https://www-us.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz \
        && tar xzf apache-maven-3.6.3-bin.tar.gz \
        && ln -s apache-maven-3.6.3 maven \
        && export PATH=${M2_HOME}/bin:${PATH}

# Install TurboJpegProcessor dependencies
RUN mkdir -p /opt/libjpeg-turbo/lib
COPY docker/Linux-JDK11/image_files/libjpeg-turbo/lib64 /opt/libjpeg-turbo/lib

# Install KakaduNativeProcessor dependencies
COPY dist/deps/Linux-x86-64/lib/* /usr/lib/

# A non-root user is needed for some FilesystemSourceTest tests to work.
ARG user=cantaloupe
ARG home=/home/$user
RUN adduser --home $home $user
RUN chown -R $user $home
USER $user
WORKDIR $home

# Add mirrors for maven central as the one referenced in ImageIO-EXT's POM is
# offline. TODO: come back in the future and check whether this is still necessary
RUN mkdir $home/.m2
COPY docker/Linux-JDK11/image_files/.m2/settings.xml $home/.m2/settings.xml

# Install application dependencies
COPY ./pom.xml pom.xml
RUN mvn --quiet dependency:resolve

# Copy the code
COPY --chown=cantaloupe docker/Linux-JDK11/image_files/test.properties test.properties
COPY --chown=cantaloupe ./src src

ENTRYPOINT mvn test -Pfreedeps
