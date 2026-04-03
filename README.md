sbog/traefik
============

Simple role to install traefik

#### Requirements

Ansible 2.4

#### Role Variables

If you use docker swarm, you should specify variables for the host on which
the container is supposed to be run, and also indicate the name of the swarm
manager through which the stack will be deployed in variable
`traefik_swarm_manager`

```yaml
traefik_swarm_manager: swarm-manager01
```

In common case, there are minimal sane config with variables you want to change:

```yaml
# There are vars for ACME DNS challenge. Read about it in documentation:
# https://doc.traefik.io/traefik/https/acme/#providers
traefik_environment_vars:
  - CF_API_EMAIL={{ vault_tls_host.acme_ch_dns_vars.CF_Email }}
  - CF_API_KEY={{ vault_tls_host.acme_ch_dns_vars.CF_Key }}

# Traefik network name. Even if it is default set as tf_net, it looks better
# to set it explicitly in host vars as long as services will need to be
# attached to it too
traefik_docker_network_name: tf_net
# Whether traefik will be installed in cluster
traefik_swarm_cluster: true
# If we use swarm, we need to specify manager node explicitly
traefik_swarm_manager: ru01.sbog.org
# At which name traefik dashboard will be available
traefik_web_host: traefik.vpn.sbog.ru

# Vars for basic auth - we do not want to show our api to open world
traefik_global_basicauth:
  http:
    middlewares:
      basic-auth:
        basicAuth:
          users:
            # Realpass: test
            # Was created by running `htpasswd -n user`
            - "user:$apr1$VQFhEy1g$hmCfpWzXYkT.qm8dqCz.w1"
```

This role uses two variables to build the final Traefik configuration:
traefik_config_default — base (shared) configuration
traefik_config — host- or group-specific overrides

They are merged using a deep (recursive) merge:

```yaml
- name: Merge traefik confs
  set_fact:
    traefik_config: "{{ traefik_config_default | combine(traefik_config, recursive=True) }}"
```

Example

```yaml
traefik_config_default:
  entryPoints:
    web:
      address: ":80"
  providers:
    docker: {}
  certificatesResolvers:
    acmeDNS:
      acme:
        dnsChallenge:
          resolvers:
            - 1.1.1.1:53
            - 8.8.8.8:53

traefik_config:
  entryPoints:
    web:
      address: ":8080"
  certificatesResolvers:
    acmeDNS:
      acme:
        dnsChallenge:
          resolvers:
            - 5.5.5.5:53
```

Result:

```yaml
traefik_config:
  entryPoints:
    web:
      address: ":8080"
  providers:
    docker: {}
  certificatesResolvers:
    acmeDNS:
      acme:
        dnsChallenge:
          resolvers:
            - 5.5.5.5:53
```

#### Dependencies

None

#### Example Playbook

```yaml
- name: Install and configure traefik
  hosts: traefik
  remote_user: root
  roles:
    - { role: sorrowless.traefik, tags: [ 'traefik' ] }
```

#### License

Apache 2.0

#### Author Information

Stan Bogatkin (https://sbog.org)
