# Unbound DNS64 Docker Container Build

This repository provides instructions on how to build multi-architecture Docker containers for unbound (a DNS64 implementation).

## Building the Docker Container

The following steps outline how to build the Tayga Docker container images for different architectures (arm, arm64, amd64) using Docker Buildx and save them as tar files.

1.  Install Docker and Buildx plugins:
    ```bash
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
2.  List existing buildx builders:
    ```bash
    docker buildx ls
    ```
3.  Install QEMU emulators for multi-architecture builds (requires `--privileged`):
    ```bash
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```
4.  Build the container image for ARM architecture (v7):
    ```bash
    docker buildx build --no-cache --platform arm --output=type=docker -t unbound-arm .
    ```
    (Note: The '.' assumes your Dockerfile is in the current directory)
5.  Build the container image for ARM64 (aarch64):
    ```bash
    docker buildx build --no-cache --platform arm64 --output=type=docker -t unbound-arm64 .
    ```
6.  Build the container image for AMD64:
    ```bash
    docker buildx build --no-cache --platform amd64 --output=type=docker -t unbound-amd64 .
    ```
7.  Save the built images to tar files. These files can then be uploaded to your target device (like Mikrotik RouterOS).
    ```bash
    docker save unbound-arm > /tmp/unbound-arm.tar
    docker save unbound-arm64 > /tmp/unbound-arm64.tar
    docker save unbound-amd64 > /tmp/unbound-amd64.tar
    ```
    Choose the appropriate tar file based on your target device's architecture.

8. Settings on MikroTik RouerOS
    ```RouterOS
        /interface veth
        add address=172.17.0.4/24,2001:db8:0:ffff::4/64 gateway=172.17.0.1 gateway6=2001:db8:0:ffff::1 name=veth1
        
        /interface bridge add name=containers
        /interface bridge port add bridge=containers interface=veth1
        /ip address add address=172.17.0.1/24 interface=containers network=172.17.0.0
        
        /container
        add envlists=unbound file=-disk1/unbound-arm64.tar interface=veth1 logging=yes name=unbound root-dir=-disk1/unbound workdir=/
        
        /container envs
        add key=UNBOUND_CACHE_MAX_TTL list=unbound value=86400
        add key=UNBOUND_CACHE_MIN_TTL list=unbound value=3600
        add key=UNBOUND_DNS64_PREFIX list=unbound value=2001:db8:1:ffff::/96
        add key=UNBOUND_DNS64_SYNTHALL list=unbound value=yes
        add key=UNBOUND_DO_IP6 list=unbound value=yes
        add key=UNBOUND_FORWARD_ADDR1 list=unbound value=8.8.8.8
        add key=UNBOUND_FORWARD_ADDR2 list=unbound value=1.1.1.1
        add key=UNBOUND_MODULE_CONFIG list=unbound value="dns64 validator iterator"
        
        /ipv6 address
        add address=2001:db8:0:ffff::1/64 advertise=no interface=containers
    ```

---

# Сборка Docker Контейнера Tayga NAT64

Этот репозиторий содержит инструкции по сборке Docker контейнеров для Tayga (реализация NAT64) для различных архитектур.

## Сборка Docker Контейнера

Следующие шаги описывают, как собрать образы Docker контейнера Tayga для различных архитектур (arm, arm64, amd64) с использованием Docker Buildx и сохранить их в tar-файлы.

1.  Установите Docker и плагины Buildx:
    ```bash
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
2.  Просмотрите существующие buildx билдеры:
    ```bash
    docker buildx ls
    ```
3.  Установите эмуляторы QEMU для сборки под разные архитектуры (требуется `--privileged`):
    ```bash
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```
4.  Соберите образ контейнера для архитектуры ARM (v7):
    ```bash
    docker buildx build --no-cache --platform arm --output=type=docker -t unbound-arm .
    ```
    (Примечание: '.' предполагает, что ваш Dockerfile находится в текущей директории)
5.  Соберите образ контейнера для ARM64 (aarch64):
    ```bash
    docker buildx build --no-cache --platform arm64 --output=type=docker -t unbound-arm64 .
    ```
6.  Соберите образ контейнера для AMD64:
    ```bash
    docker buildx build --no-cache --platform amd64 --output=type=docker -t unbound-amd64 .
    ```
7.  Сохраните собранные образы в tar-файлы. Эти файлы затем могут быть загружены на ваше целевое устройство (например, Mikrotik RouterOS).
    ```bash
    docker save unbound-arm > /tmp/unbound-arm.tar
    docker save unbound-arm64 > /tmp/unbound-arm64.tar
    docker save unbound-amd64 > /tmp/unbound-amd64.tar
    ```
    Выберите подходящий tar-файл в зависимости от архитектуры вашего целевого устройства.

8. Настройка в MikroTik RouerOS
    ```RouterOS
        /interface veth
        add address=172.17.0.4/24,2001:db8:0:ffff::4/64 gateway=172.17.0.1 gateway6=2001:db8:0:ffff::1 name=veth1
        
        /interface bridge add name=containers
        /interface bridge port add bridge=containers interface=veth1
        /ip address add address=172.17.0.1/24 interface=containers network=172.17.0.0
        
        /container
        add envlists=unbound file=-disk1/unbound-arm64.tar interface=veth1 logging=yes name=unbound root-dir=-disk1/unbound workdir=/
        
        /container envs
        add key=UNBOUND_CACHE_MAX_TTL list=unbound value=86400
        add key=UNBOUND_CACHE_MIN_TTL list=unbound value=3600
        add key=UNBOUND_DNS64_PREFIX list=unbound value=2001:db8:1:ffff::/96
        add key=UNBOUND_DNS64_SYNTHALL list=unbound value=yes
        add key=UNBOUND_DO_IP6 list=unbound value=yes
        add key=UNBOUND_FORWARD_ADDR1 list=unbound value=8.8.8.8
        add key=UNBOUND_FORWARD_ADDR2 list=unbound value=1.1.1.1
        add key=UNBOUND_MODULE_CONFIG list=unbound value="dns64 validator iterator"
        
        /ipv6 address
        add address=2001:db8:0:ffff::1/64 advertise=no interface=containers
        
    ```

        
