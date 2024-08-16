# Installation #

## Installation on Ubuntu ##

There are 2 ways to install Docker (See [Install Docker on
Linux](https://docs.docker.com/desktop/install/linux-install/)):
1. Install [Docker Server/Engine](https://docs.docker.com/engine/).

   ```bash
   ARCH=$(dpkg --print-architecture)
   CODENAME=$(. /etc/os-release && echo "$VERSION_CODENAME")
   
   APT_URL="https://download.docker.com/linux/ubuntu"
   APT_LIST="/etc/apt/sources.list.d/docker.list"
   KEYRING_DIR="/etc/apt/keyrings"
   GPG_PATH="${KEYRING_DIR}/docker.asc"
   GPG_URL="${APT_URL}/gpg"
   
   APT_ENTRY="deb [arch=${ARCH} signed-by=${GPG_PATH}] ${APT_URL} ${CODENAME} stable"
   
   # Update APT package index in case of new fresh install
   sudo apt-get update
   
   # Install necessary dependencies
   sudo apt-get install -y ca-certificates curl
   
   # Add Docker's official GPG key
   sudo install -m 0755 -d "${KEYRING_DIR}"
   curl -fsSL "${GPG_URL}" | sudo gpg --dearmor -o "${GPG_PATH}"
   sudo chmod a+r "${GPG_PATH}"
   
   # Set Docker APT repo source
   echo "${APT_ENTRY}" | sudo tee "${APT_LIST}" > /dev/null
   sudo apt-get update
   
   # Install the latest Docker community edition
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   
   # Test if docker is installed properly.
   sudo docker run hello-world
   ```

1. Install [Docker Desktop](https://docs.docker.com/desktop/).
   + Docker Desktop is GA, but it is not recommended to have both
     Docker Desktop and Docker Engine on the same machine.
   + Unlike Docker Engine, Docker Desktop on Linux runs a VM instead
     of directly on the machine.  More details can be found at [FAQs
     for Linux](https://docs.docker.com/desktop/faqs/linuxfaqs/).
