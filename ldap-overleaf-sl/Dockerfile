# Base image with ShareLaTeX
FROM sharelatex/sharelatex:4.2.0

LABEL maintainer="Simon Haller-Seeber"
LABEL version="0.2"

# Set working directory
WORKDIR /overleaf/services/web

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g npm@8.19.3 && \
    npm install ldapts@3.2.4


# Install additional packages
RUN apt-get update && \
    apt-get -y install \
        libxml-libxslt-perl \
        cpanminus \
        libbtparse2 \
        python-pygments && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install latest TeX Live 2024
RUN wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz && \
    tar -xzf install-tl-unx.tar.gz && \
    cd install-tl-* && \
    echo "selected_scheme scheme-full" > texlive.profile && \
    ./install-tl --profile=texlive.profile && \
    cd .. && \
    rm -rf install-tl-* install-tl-unx.tar.gz && \
    export PATH="/usr/local/texlive/2024/bin/x86_64-linux:$PATH" && \
    tlmgr update --self --all

# Set PATH for TeX Live binaries
ENV PATH="/usr/local/texlive/2024/bin/x86_64-linux:${PATH}"

# Copy custom ShareLaTeX configuration files
COPY sharelatex/AuthenticationManager.js /overleaf/services/web/app/src/Features/Authentication/
COPY sharelatex/AuthenticationController.js /overleaf/services/web/app/src/Features/Authentication/
COPY sharelatex/ContactController.js /overleaf/services/web/app/src/Features/Contacts/
COPY sharelatex/router.js /overleaf/services/web/app/src/router.js
COPY sharelatex/settings.pug /overleaf/services/web/app/views/user/
COPY sharelatex/login.pug /overleaf/services/web/app/views/user/
COPY sharelatex/navbar.pug /overleaf/services/web/app/views/layout/
COPY sharelatex/navbar-marketing.pug /overleaf/services/web/app/views/layout/
COPY sharelatex/admin-index.pug /overleaf/services/web/app/views/admin/index.pug
COPY sharelatex/admin-sysadmin.pug /tmp/admin-sysadmin.pug

# Apply customizations using sed
RUN sed -iE "s/email@example.com/${login_text:-user}/g" /overleaf/services/web/app/views/user/login.pug && \
    sed -iE "s%-synctex=1\",%-synctex=1\", \"-shell-escape\",%g" /overleaf/services/clsi/app/js/LatexRunner.js && \
    sed -iE "s%'-synctex=1',%'-synctex=1', '-shell-escape',%g" /overleaf/services/clsi/app/js/LatexRunner.js && \
    if [ "${admin_is_sysadmin}" = "true" ]; then \
        cp /tmp/admin-sysadmin.pug /overleaf/services/web/app/views/admin/index.pug; \
    else \
        rm /tmp/admin-sysadmin.pug; \
    fi
