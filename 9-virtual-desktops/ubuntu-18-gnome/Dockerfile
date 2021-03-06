#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
# 
#   http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM ubuntu:18.04
ENV DEBIAN_FRONTEND noninteractive

ENV LANG=en_GB.UTF-8
ENV TZ=Europe/London

RUN \
    # Enable partner repository (needed for codecs)
    sed -i 's/# deb http:\/\/archive.canonical.com\/ubuntu bionic partner/deb http:\/\/archive.canonical.com\/ubuntu bionic partner/' /etc/apt/sources.list && \
    # Remove "This system has been minimized" warning.
    rm -f /etc/update-motd.d/60-unminimize && \
    # Update base packages.
    apt-get update && \
    apt-get -fy -o Dpkg::Options::="--force-confnew" -o APT::Immediate-Configure=false dist-upgrade && \
    # Add the main packages
    apt-get install -y --no-install-recommends \
    base-files add-apt-key aptdaemon apt-transport-https \
    language-pack-en gnome ubuntu-minimal ubuntu-standard ubuntu-desktop \
    ubuntu-system-service ubuntu-restricted-extras gsettings-ubuntu-schemas \
    gnome-shell-extension-ubuntu-dock gnome-shell-extension-appindicator \
    gnome-control-center-faces gnome-software-plugin-snap \
    # Add additional packages to be similar to Ubuntu iso installation.
    synaptic thunderbird-locale-en thunderbird-locale-en-us \
    transmission-gtk firefox-locale-en tracker-miner-fs avahi-utils \
    remmina remmina-common remmina-plugin-rdp remmina-plugin-vnc \
    remmina-plugin-nx remmina-plugin-spice remmina-plugin-xdmcp \
    mesa-utils paprefs pavucontrol libcanberra-pulse \
    pulseaudio-module-bluetooth gstreamer1.0-pulseaudio \
    pulseaudio-module-zeroconf cups system-config-printer-gnome \
    && \
    # Tidy up
    apt-get -y autoremove && \
    apt-get clean && \
    # Generate locales
    echo LANG=$LANG > /etc/default/locale && \
    update-locale LANG=$LANG && \
    # Set up the timezone
    echo $TZ > /etc/timezone && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure tzdata && \
    # Configure console
    echo console-setup console-setup/charmap select UTF-8 | debconf-set-selections && \
    # Fix synaptic Empty Dir::Cache::pkgcache setting not handled correctly
    # https://bugs.launchpad.net/ubuntu/+source/synaptic/+bug/1243615
    # which causes synaptic to barf with:
    # E: Could not open file - open (2: No such file or directory)
    # E: _cache->open() failed, please report.
    sed -i 's/Dir::Cache::pkgcache ""; //' /etc/apt/apt.conf.d/docker-clean && \
    # Remove some unnecessary files from /etc/X11/Xsession.d The files in
    # this directory are called by /etc/X11/Xsession and the ones below run
    # X clients (xhost and xrdb), this causes Xephyr to terminate prematurely
    # as we have set Xephyr to reset and terminate after last client exits.
    rm /etc/X11/Xsession.d/30x11-common_xresources && \
    rm /etc/X11/Xsession.d/35x11-common_xhost-local && \
    rm /etc/X11/Xsession.d/60x11-common_localhost && \
    # Remove unnecessary file from /etc/update-motd.d to suppress warning
    rm /etc/update-motd.d/98-fsck-at-reboot && \
    # Fix issues with slow shutdown
    sed -i 's/#DefaultTimeoutStopSec=90s/DefaultTimeoutStopSec=5s/' /etc/systemd/system.conf && \
    # Fix Polkit issues caused by container login being considered
    # to be an "inactive" session.
    chmod 755 /etc/polkit-1/localauthority && \
    # Date & Time
    echo "[Date & Time]\nIdentity=unix-user:*\nAction=org.cinnamon.settingsdaemon.datetimemechanism.configure\nResultAny=auth_admin_keep\nResultInactive=auth_admin_keep\nResultActive=auth_admin_keep\n" > /etc/polkit-1/localauthority/50-local.d/10-datetimemechanism.pkla  && \
    # Xapian Index
    echo "[Xapian Index]\nIdentity=unix-user:*\nAction=org.debian.aptxapianindex.update\nResultAny=yes\nResultInactive=yes\nResultActive=yes\n" > /etc/polkit-1/localauthority/50-local.d/10-xapian.pkla && \
    # Gnome System Log
    echo "[Gnome System Log]\nIdentity=unix-user:*\nAction=org.debian.pkexec.gnome-system-log.run\nResultAny=auth_admin_keep\nResultInactive=auth_admin_keep\nResultActive=auth_admin_keep\n" > /etc/polkit-1/localauthority/50-local.d/10-system-log.pkla && \
    # System Color Manager
    echo "[System Color Manager]\nIdentity=unix-user:*\nAction=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile;org.freedesktop.color-manager.device-inhibit;org.freedesktop.color-manager.sensor-lock\nResultAny=yes\nResultInactive=yes\nResultActive=yes\n" > /etc/polkit-1/localauthority/50-local.d/10-color.pkla && \
    # Shutdown & Restart
    # Note that auth_admin_keep may be better than yes here, but there seems to be
    # an issue with the authentication dialog appearing.
    echo "[Shutdown & Restart]\nIdentity=unix-user:*\nAction=org.freedesktop.login1.power-off;org.freedesktop.login1.power-off-multiple-sessions;org.freedesktop.login1.reboot;org.freedesktop.login1.reboot-multiple-sessions\nResultAny=yes\nResultInactive=yes\nResultActive=yes\n" > /etc/polkit-1/localauthority/50-local.d/10-shutdown.pkla

#-------------------------------------------------------------------------------
# Example usage
# 
# Build the image
# docker build -t ubuntu-gnome:18 .

