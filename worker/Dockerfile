FROM base/devel
MAINTAINER minextu

RUN pacman -Sy --noconfirm python2-pip python2-twisted python2-future git wget && \
    find /var/cache/pacman/pkg -mindepth 1 -delete

# install buildbot worker
RUN pip2 install buildbot-worker

# remove all systemd services
USER root
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done) && \
    rm -f /lib/systemd/system/multi-user.target.wants/*       && \
    rm -f /etc/systemd/system/*.wants/*                       && \
    rm -f /lib/systemd/system/local-fs.target.wants/*         && \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*     && \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*  && \
    rm -f /lib/systemd/system/basic.target.wants/*            && \
    rm -f /lib/systemd/system/anaconda.target.wants/*

# add buildbot systemd service
ADD buildbot-worker.service /etc/systemd/system/
RUN systemctl enable buildbot-worker.service

# add buildbot user and give passwordless sudo access (needed for archzfs build scripts)
RUN groupadd -r buildbot && \
    useradd -m -r -g buildbot buildbot && \
    mkdir /worker && \
    chown buildbot:buildbot /worker && \
    echo "buildbot ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# aur prep
RUN useradd aur && \
    echo "aur ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers  && \
    sed -i '/en_US\.UTF-8 UTF-8/s/^#//' /etc/locale.gen && \
    echo "LANG=en_US.UTF-8" > /etc/locale.conf          && \
    locale-gen

# install archzfs dependencies
RUN pacman -Sy --noconfirm pkgbuild-introspection sshfs && \
    find /var/cache/pacman/pkg -mindepth 1 -delete

# clean-chroot-manager
USER aur
RUN cd /tmp && \
    wget https://aur.archlinux.org/cgit/aur.git/snapshot/clean-chroot-manager.tar.gz && \
    tar -xvf clean-chroot-manager.tar.gz && \
    cd clean-chroot-manager              && \
    makepkg -si --noconfirm              && \
    rm -r /tmp/*

# run ccm64 once to generate config file
USER buildbot
RUN sudo ccm64                      && \
    sudo mkdir -p /scratch/chroot64

USER root
# fix for https://github.com/systemd/systemd/issues/6347
RUN sed -i 's/--register=no/--register=no --machine \$(uuidgen)/' /usr/bin/arch-nspawn

ADD start.sh /
VOLUME [ "/sys/fs/cgroup" ]
CMD ["sh", "/start.sh"]