# Proxy #

## For Docker Daemon and CLI Itself Only ##

If the machine connects to the internet via a proxy server, we need to
tell the Docker daemon where the proxy server is, via the following 3
ways, so that the Docker daemon can also connect to the internet.  See
[Daemon proxy
configuration](https://docs.docker.com/engine/daemon/proxy) and
[Environment
variables](https://docs.docker.com/reference/cli/docker/#environment-variables)
for more details.
* Daemon configuration set in
  [`/etc/docker/daemon.json`](https://docs.docker.com/reference/cli/dockerd/#daemon-configuration-file)
  for regular setup or
  [`${HOME}/.config/docker/daemon.json`](https://docs.docker.com/engine/security/rootless/tips/)
  for rootless mode.  See also [Configuraiton
  file](https://docs.docker.com/engine/daemon/#configuration-file).

  ```json
  {
    "proxies": {
      "http-proxy": "http://10.60.204.164:3128",
      "https-proxy": "http://10.60.204.164:4128",
      "no-proxy": "10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com"
    }
  }
  ```

  And then restart the Docker service via

  ```
  sudo systemctl restart docker
  ```

* CLI flags for the `dockerd` command, such as `--http-proxy` and
  `--https-proxy`.
* Environment variables set in `/etc/environment`

  ```
  http_proxy='http://10.60.204.164:3128'
  HTTP_PROXY='http://10.60.204.164:3128'
  https_proxy='http://10.60.204.164:4128'
  HTTPS_PROXY='http://10.60.204.164:4128'
  no_proxy='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
  NO_PROXY='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
  ```


## For Commands and Apps During Docker Build and Run ##

Proxy settings configured for the Docker daemon have no effect on
commands and apps requiring internet connections during `docker build`
and `docker run`.  The following ways of proxy configuration are
provided for them.  See [Use a proxy server with the Docker
CLI](https://docs.docker.com/engine/cli/proxy) and [Automatic proxy
configuration for
containers](https://docs.docker.com/reference/cli/docker/#automatic-proxy-configuration-for-containers)
for more details:
* Client configuration set in `${HOME}/.docker/config.json`.

  ```json
  {
   "proxies": {
     "default": {
       "httpProxy": "http://10.60.204.164:3128",
       "httpsProxy": "http://10.60.204.164:4128",
       "noProxy": "10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com"
     }
   }
  }
  ```

  And the settings take effect after saving the file.

* CLI flags for the `docker` command
  + `--build-arg` for `docker build` via [predefined
    ARGs](https://docs.docker.com/reference/dockerfile/#predefined-args)
    for proxy configuration, such as

    ```bash
    docker build . \
      --build-arg http_proxy='http://10.60.204.164:3128'
      --build-arg HTTP_PROXY='http://10.60.204.164:3128'
      --build-arg https_proxy='http://10.60.204.164:4128'
      --build-arg HTTPS_PROXY='http://10.60.204.164:4128'
      --build-arg no_proxy='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
      --build-arg NO_PROXY='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
    ```
  + `--env` for `docker run`, such as

    ```bash
    docker run \
      --env http_proxy='http://10.60.204.164:3128'
      --env HTTP_PROXY='http://10.60.204.164:3128'
      --env https_proxy='http://10.60.204.164:4128'
      --env HTTPS_PROXY='http://10.60.204.164:4128'
      --env no_proxy='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
      --env NO_PROXY='10.60.204.164:5000,mirrors.ucloud.cn,developer.download.nvidia.com'
    ```
