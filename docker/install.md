# Installation #

## Installation on Ubuntu ##

There are 2 ways to install Docker (See [Install Docker on
Linux](https://docs.docker.com/desktop/install/linux-install/)):
1. Install [Docker Server/Engine](https://docs.docker.com/engine/).

   ```bash
   # Make sure no docker is installed
   # 1. Remove old versions
   for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
   # 2. Uninstall Docker
   sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   # 3. Remove images, containers, and volumes
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
   
   # Install a new fresh Docker
   # 1. Set env variables
   ARCH=$(dpkg --print-architecture)
   CODENAME=$(. /etc/os-release && echo "$VERSION_CODENAME")
   APT_URL="https://download.docker.com/linux/ubuntu"
   APT_LIST="/etc/apt/sources.list.d/docker.list"
   KEYRING_DIR="/etc/apt/keyrings"
   GPG_PATH="${KEYRING_DIR}/docker.asc"
   GPG_URL="${APT_URL}/gpg"
   APT_ENTRY="deb [arch=${ARCH} signed-by=${GPG_PATH}] ${APT_URL} ${CODENAME} stable"
   # 2. Update APT package index in case of new fresh install
   sudo apt-get update
   # 3. Install prerequisites
   sudo apt-get install -y ca-certificates curl
   # 4. Add Docker's official GPG key
   sudo install -m 0755 -d "${KEYRING_DIR}"
   sudo curl -fsSL "${GPG_URL}" -o "${GPG_PATH}"
   sudo chmod a+r "${GPG_PATH}"
   # 5. Set Docker APT repo source
   #    Need VPN in China to access the docker server
   echo "${APT_ENTRY}" | sudo tee "${APT_LIST}" > /dev/null
   sudo apt-get update
   # 6. Install the latest Docker community edition
   sudo apt-get install -y docker-ce
   sudo apt-get install -y docker-ce-cli
   sudo apt-get install -y containerd.io
   sudo apt-get install -y docker-buildx-plugin
   sudo apt-get install -y docker-compose-plugin
   # 7. Test if docker is installed properly.
   sudo docker run hello-world
   
   # Manage Docker as a non-root user
   # See https://docs.docker.com/engine/install/linux-postinstall/
   # 1. Add current user to the docker group
   sudo groupadd docker
   sudo usermod -aG docker $USER
   newgrp docker
   sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
   sudo chmod g+rwx "$HOME/.docker" -R
   # 2. (Optional on Ubuntu) Start Docker service on boot
   sudo systemctl enable docker.service
   sudo systemctl enable containerd.service
 
   # (Optional) Run the Docker daemon as a non-root user (Rootless mode)
   # See https://docs.docker.com/engine/security/rootless/
   # 1. Install prerequisites
   sudo apt-get install -y dbus-user-session
   sudo apt-get install -y uidmap
   sudo apt-get install -y systemd-container
   sudo apt-get install -y docker-ce-rootless-extras
   # 2. Disable system-wide docker daemon
   sudo systemctl disable --now docker.service docker.socket
   sudo rm /var/run/docker.sock
   # 3. Install rootless docker daemon
   dockerd-rootless-setuptool.sh install
   # 4. Start rootless docker daemon
   systemctl --user start docker
   # 5. Enable the systemd service and launch the daemon on system startup
   systemctl --user enable docker
   sudo loginctl enable-linger $(whoami)
   # 6. Switch docker context to rootless
   docker context use rootless
   ```

1. Install [Docker Desktop](https://docs.docker.com/desktop/).
   + Docker Desktop is GA, but it is not recommended to have both
     Docker Desktop and Docker Engine on the same machine.
   + Unlike Docker Engine, Docker Desktop on Linux runs a VM instead
     of directly on the machine.  More details can be found at [FAQs
     for Linux](https://docs.docker.com/desktop/faqs/linuxfaqs/).
