##
## Development environment to build the seL4cp SDK on a workstation.
##

# Extend the base image. Make sure that you have already built it.
FROM makatea-project/sel4cp-sdk-builder

# These args should come from the command line, but these should be
# relatively sane defaults.
ARG USER_NAME=user
ARG USER_UID=1000
ARG USER_GID=1000

# Add a user with username/uid/gid matching those of the developer.
# This way files created by the container in shared mountpoints have
# correct ownerships. Also add a password-less sudo, that's
# occasionally handy to add ad-hoc packages.
RUN set -x                                                              \
    && apt-get -y update                                                \
    && apt-get -y install --no-install-recommends sudo                  \
    && apt-get clean                                                    \
    && rm -rf /var/cache/apt                                            \
    && addgroup --gid $USER_GID $USER_NAME                              \
    && adduser --uid $USER_UID --gecos $USER_NAME                       \
               --ingroup $USER_NAME --disabled-password $USER_NAME      \
    && echo "$USER_NAME ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
WORKDIR /home/$USER_NAME
ENV CARGO_HOME=/home/$USER_NAME/.cache/cargo \
    TEXMFVAR=/home/$USER_NAME/.cache/texlive/texmf-vars
USER $USER_NAME
