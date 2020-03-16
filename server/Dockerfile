# Upstream Dockerfile https://github.com/rocker-org/rocker-versioned/blob/master/tidyverse/3.4.4/Dockerfile
FROM rocker/tidyverse

RUN apt-get update && apt-get install --no-install-recommends -y \
    nano \
    libcurl4-openssl-dev \
    openssh-client \
    less \
    htop \
    procps \
    net-tools \
    rsync \
    gdebi-core \
    libxt-dev \
    man-db \
    git-lfs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

ENV USERNAME=rstudio

# Setup various variables
ENV TZ="Europe/Helsinki" \
    HOME="/home/${USERNAME}" \
    TINI_VERSION=v0.18.0 \
    APP_UID=999 \
    APP_GID=999 \
    PKG_SHINY_VERSION=1.5.7.907

# Setup Tini, as S6 does not work when run as non-root users
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /sbin/tini
RUN chmod +x /sbin/tini

# Setup Shiny
RUN export ADD=shiny && bash /etc/cont-init.d/add && \
    install2.r -e shiny rmarkdown shinythemes shinydashboard && \
    cp -R /usr/local/lib/R/site-library/shiny/examples/* /srv/shiny-server/ && \
    mkdir -p /var/log/shiny-server && \
    chown rstudio.rstudio /var/log/shiny-server && \
    chmod go+w -R /var/log/shiny-server /usr/local/lib/R /srv /var/lib/shiny-server

# add requirements
COPY requirements.txt /tmp/

RUN cat /tmp/requirements.txt | xargs install2.r -e && rm /tmp/requirements.txt

COPY rserver.conf rsession.conf /etc/rstudio/
COPY start.sh /usr/local/bin/start.sh
COPY shiny-server.conf /etc/shiny-server/shiny-server.conf
COPY .Renviron $HOME/.Renviron

# add git config and keys

COPY flopo-at-github.pem $HOME/.ssh/id_rsa

COPY gitconfig $HOME/.gitconfig

RUN ssh-keyscan github.com > $HOME/.ssh/known_hosts

#OpenShift additions
RUN if [ "$USERNAME" != "rstudio" ] ; then useradd -m $USERNAME ; fi

RUN chmod a+x /usr/local/bin/start.sh

RUN chmod go+w -R $HOME && \
    usermod -u 988 rstudio-server && \
    groupmod -g 988 rstudio-server && \
    usermod -u "$APP_UID" "$USERNAME" && \
    groupmod -g "$APP_GID" "$USERNAME" && \
    chmod -R go+w /tmp/downloaded_packages /etc/rstudio/rsession.conf

RUN chgrp root -R /usr/local/lib/R/site-library

COPY fix.sh /usr/local/bin/fix.sh
RUN chmod ug+rw /etc/passwd
RUN chmod ug+rw /etc/group

RUN chgrp root /etc/shadow

RUN chmod ug+rw /etc/shadow

RUN chgrp root /usr/local/bin/fix.sh \
    && chmod 774 /usr/local/bin/fix.sh

RUN chgrp -R 0 $HOME && chmod -R g=u $HOME

USER $APP_UID:$APP_GID
WORKDIR $HOME
EXPOSE 8787 3838
#MAINTAINER
ENTRYPOINT ["/sbin/tini", "-g", "--"]
CMD ["/usr/local/bin/start.sh"]
