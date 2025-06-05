# Rol de Ansible: Docker

[![CI](https://github.com/afreisinger/ansible-role-docker/actions/workflows/ci.yml/badge.svg)](https://github.com/afreisinger/ansible-role-docker/actions/workflows/ci.yml)

Un rol de Ansible que instala [Docker](https://www.docker.com) en Linux.

## Requisitos

Ninguno.

## Variables del Rol

A continuación, se enumeran las variables disponibles junto con sus valores predeterminados (ver `defaults/main.yml`):

```yaml
# La edición puede ser: 'ce' (Community Edition) o 'ee' (Enterprise Edition).
docker_edition: 'ce'
docker_packages:
    - "docker-{{ docker_edition }}"
    - "docker-{{ docker_edition }}-cli"
    - "docker-{{ docker_edition }}-rootless-extras"
docker_packages_state: present
```

La variable `docker_edition` debe ser `ce` (Community Edition) o `ee` (Enterprise Edition). También puedes especificar una versión específica de Docker para instalar usando el formato específico de la distribución: 
Red Hat/CentOS: `docker-{{ docker_edition }}-<VERSIÓN>` (Nota: debes añadir esto a todos los paquetes);
Debian/Ubuntu: `docker-{{ docker_edition }}=<VERSIÓN>` (Nota: debes añadir esto a todos los paquetes).

Puedes controlar si el paquete se instala, desinstala o se actualiza a la última versión configurando `docker_packages_state` como `present`, `absent` o `latest`, respectivamente. Ten en cuenta que el demonio de Docker se reiniciará automáticamente si el paquete de Docker se actualiza. Esto es un efecto secundario de ejecutar todos los manejadores notificados por este y otros roles hasta este punto en la ejecución.

```yaml
docker_obsolete_packages:
  - docker
  - docker.io
  - docker-engine
  - docker-doc
  - docker-compose
  - docker-compose-v2
  - podman-docker
  - containerd
  - runc
```

`docker_obsolete_packages` para diferentes familias de sistemas operativos:

- [`RedHat.yaml`](./vars/RedHat.yml)
- [`Debian.yaml`](./vars/Debian.yml)

Lista de paquetes que se desinstalarán antes de ejecutar este rol. Consulta las [instrucciones de instalación de Docker](https://docs.docker.com/engine/install/debian/#uninstall-old-versions) para obtener una lista actualizada de paquetes antiguos que deben eliminarse.

```yaml
docker_service_manage: true
docker_service_state: started
docker_service_enabled: true
docker_restart_handler_state: restarted
```

Variables para controlar el estado del servicio `docker` y si debe iniciarse al arrancar el sistema. Si estás instalando Docker dentro de un contenedor Docker sin systemd o sysvinit, debes establecer `docker_service_manage` en `false`.

```yaml
docker_install_compose_plugin: true
docker_compose_package: docker-compose-plugin
docker_compose_package_state: present
```

Opciones de instalación del complemento Docker Compose. Estas difieren de las siguientes en que docker-compose se instala como un complemento de Docker (y se usa con `docker compose`) en lugar de un binario independiente.

```yaml
docker_install_compose: false
docker_compose_version: "v2.32.1"
docker_compose_arch: "{{ ansible_architecture }}"
docker_compose_url: "https://github.com/docker/compose/releases/download/{{ docker_compose_version }}/docker-compose-linux-{{ docker_compose_arch }}"
docker_compose_path: /usr/local/bin/docker-compose
```

Opciones de instalación de Docker Compose.

```yaml
docker_add_repo: true
```

Controla si este rol añadirá el repositorio oficial de Docker. Establécelo en `false` si deseas usar los paquetes de Docker predeterminados de tu sistema o gestionar el repositorio de paquetes por tu cuenta.

```yaml
docker_repo_url: https://download.docker.com/linux
```

La URL del repositorio principal de Docker, común para sistemas Debian y RHEL.

```yaml
docker_apt_release_channel: stable
docker_apt_arch: "{{ 'arm64' if ansible_architecture == 'aarch64' else 'amd64' }}"
docker_apt_repository: "deb [arch={{ docker_apt_arch }}{{' signed-by=/etc/apt/keyrings/docker.asc' if add_repository_key is not failed}}] {{ docker_repo_url }}/{{ ansible_distribution | lower }} {{ ansible_distribution_release }} {{ docker_apt_release_channel }}"
docker_apt_ignore_key_error: True
docker_apt_gpg_key: "{{ docker_repo_url }}/{{ ansible_distribution | lower }}/gpg"
docker_apt_filename: "docker"
```

(Usado solo para Debian/Ubuntu.) Puedes cambiar el canal a `nightly` si deseas usar la versión Nightly.

Puedes cambiar `docker_apt_gpg_key` a una URL diferente si estás detrás de un firewall o proporcionas un espejo confiable. Generalmente, esto se hace en combinación con el cambio de `docker_apt_repository`. La variable `docker_apt_filename` controla el nombre del archivo de la lista de fuentes creado en `sources.list.d`. Si estás actualizando desde una versión anterior (<7.0.0) de este rol, deberías cambiar esto al nombre del archivo existente (por ejemplo, `download_docker_com_linux_debian` en Debian) para evitar conflictos en las listas.

```yaml
docker_yum_repo_url: "{{ docker_repo_url }}/{{ (ansible_distribution == 'Fedora') | ternary('fedora','centos') }}/docker-{{ docker_edition }}.repo"
docker_yum_repo_enable_nightly: '0'
docker_yum_repo_enable_test: '0'
docker_yum_gpg_key: "{{ docker_repo_url }}/{{ (ansible_distribution == 'Fedora') | ternary('fedora', 'centos') }}/gpg"
```

(Usado solo para RedHat/CentOS.) Puedes habilitar el repositorio Nightly o Test configurando las variables respectivas en `1`.

Puedes cambiar `docker_yum_gpg_key` a una URL diferente si estás detrás de un firewall o proporcionas un espejo confiable. Generalmente, esto se hace en combinación con el cambio de `docker_yum_repository`.

```yaml
docker_users:
  - user1
  - user2
```

Lista de usuarios del sistema que se añadirán al grupo `docker` (para que puedan usar Docker en el servidor).

```yaml
docker_daemon_options:
  storage-driver: "overlay2"
  log-opts:
    max-size: "100m"
```

Opciones personalizadas para `dockerd` que se configuran a través de este diccionario que representa el archivo JSON `/etc/docker/daemon.json`.

```yaml
docker_service_settings:
  - HTTP_PROXY=http://proxy.example.com:80
  - HTTPS_PROXY=https://proxy.example.com:443
  - NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
```

Configuración personalizada del servicio Docker. Debería usarse solo para configuraciones de proxy `HTTP/HTTPS`.

```yaml
docker_custom_registries:
  - host: "registry.prd.example.com"
    ca_file: "registry-prd-example-ca.crt"
  - host: "registry.dev.example.com"
    ca_file: "registry-dev-example-ca.crt"
```

Registros privados de Docker confiables con Autoridades de Certificación (CAs) personalizadas. Coloca los archivos de CA en el directorio `files/` de tu rol o playbook. Cada CA se instalará en `/etc/docker/certs.d/<host>/ca.crt`.

## Uso con Ansible (y la biblioteca Python `docker`)

Muchos usuarios de este rol desean usar Ansible para construir imágenes de Docker y gestionar contenedores en el servidor donde está instalado Docker. En este caso, puedes añadir fácilmente la biblioteca Python `docker` usando el rol `geerlingguy.pip`:

```yaml
- hosts: all

  vars:
    pip_install_packages:
      - name: docker

  roles:
    - geerlingguy.pip
    - geerlingguy.docker
```

## Dependencias

Ninguna.

## Ejemplo de Playbook

```yaml
- hosts: all
  roles:
    - afreisinger.docker
```

## Licencia

MIT / BSD
