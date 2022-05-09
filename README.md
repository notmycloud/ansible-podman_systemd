# Ansible Podman_systemd
## Description
Configure Podman Systemd Units for system and rootless users.

## Installation
The following roles are also required. If I add them as a dependency, they will execute which fails.
- linuxhq.sysctl
- notmycloud.yaml2ini
- notmycloud.systemd_unit

IMPORTANT: Requires the installation of the `toml` python package. Install with pip.

## Usage
Provide the following variables for this role to operate correctly.
`PODMAN_SYSTEMD_INSTALL_COCKPIT` will install Cockpit with Podman support.
`PODMAN_SYSTEMD_ALLOW_LOWEST_PORT` will specify the lowest port that an unprivileged user can bind to. It is a debated thought that the low ports of 1024 or less are "secure" and should be limited to trusted users. If this is a multi-tenant server, it is recommended to skip this variable, however, if you manage this server entirely and setup users per service, this option can save you some headache. In my opinion, this can be set to 0 for a server that is not multi-tenant.
`PODMAN_SYSTEMD_ALLOW_PING` will allow unprivileged users to ping.
`PODMAN_SYSTEMD_DEPLOY` contains the bulk of the Podman configuration. Create a key for each user that will have a Systemd unit setup.
```
PODMAN_SYSTEMD_DEPLOY:
  root:
    etc...
  user1:
    etc...
  user2:
    etc...
```

## User configuration
Root will be configured under the default `/etc/systemd/system` directory wheras users will be configured under `~/.config/system/user` directory. In this example, we will configure for an unprivileged user.
```
PODMAN_SYSTEMD_DEPLOY:
  myuser:
    config:
      storage:
        storage:
          driver: "overlay"
        # key:value pairs that will be written as INI format for storage.conf. This will be placed in the appropriate directory for either a user or if the user is root, in the /etc configuration directory.
        # The key:value pairs should follow the format as specified by the notmycloud.yaml2ini role.
      containers:
        engine:
          network_cmd_options:
            - "allow_host_loopback=true"
            - "enable_ipv6=true"
          env: 
            - "TMPDIR=$HOME/.cache/tmp/"
        # key:value pairs that will be written as TOML format for containers.conf. This will be placed in the appropriate directory for either a user or if the user is root, in the /etc configuration directory.
      registries:
        unqualified-search-registries:
          - "docker.io"
          # - "quay.io"
          # - "registry.access.redhat.com"
        registry:
          # - prefix: "docker.io"
          #   location: "127.0.0.1:5000"
          #   insecure: true
          # In Nov. 2020, Docker rate-limits image pulling.  To avoid hitting these
          # limits while testing, always use the google mirror for qualified and
          # unqualified `docker.io` images.
          # Ref: https://cloud.google.com/container-registry/docs/pulling-cached-images
        registry.mirror:
          - location: "https://mirror.gcr.io"
        # key:value pairs that will be written as TOML format for registries.conf. This will be placed in the appropriate directory for either a user or if the user is root, in the /etc configuration directory.
      login: # See documentation for the containers.podman.podman_login module
        - registry: registry.mydomain.com
          username: registryuser
          password: super$3(r37Password
    systemd:
      enable_socket: # Default False, enables the Podman API socket for the user or if under root, system wide.
      containers:
        CONTAINERNAME: # Replace this with the name you desire for the container, this will also be utilized for the Systemd Unit file.
          podman_options:
            image:
            network: # Network string, defaults to slirp4netns:allow_host_loopback=true,enable_ipv6=true
            replace: # bool
            restart: # "always"|"no"|"on-failure"|"unless-stopped" default no
            remove: # Defaults to yes
            healthcheck:
              cmd:
              interval:
              retries:
              delay:
              timeout:
            environment: # Array of key value pairs
              key: "value"
            ports: # 0.0.0.0:hostport:containerport
            volumes: 
              - host:
                container:
                options:
                type: # file/directory
            labels: # Array of label strings
            other_options: # string of other options to pass to podman run
          service_options: # See notmycloud.systemd_unit variable depth equal to UNIT_NAME
      pods:
        PODNAME: # Replace this with the name you desire for the pod, this will also be utilized to prefix the associated containers.
          pod_service_options: # See notmycloud.systemd_unit variable depth equal to UNIT_NAME
          CONTAINERNAME:
            # Follow above PODMAN_SYSTEMD_DEPLOY.USERNAME.systemd.containers.CONTAINERNAME syntax
```

## Podman Socket

Root socket will be enabled at: `/run/podman/podman.sock`
Rootless socket will be enabled at : `/run/user/$UID/podman/podman.sock`

## Support
For support, please raise an issue and provide the following items
- Sample task/playbook to replicate your issue
- Resultant file that is created.
