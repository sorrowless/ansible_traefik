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
