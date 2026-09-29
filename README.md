# ansible-roboshop

Ansible playbooks that provision and configure the [RoboShop](https://github.com/uday1bhanu/roboshop) microservice demo on AWS (RHEL/CentOS stream / Amazon Linux style hosts).

Components: MongoDB, Redis, MySQL, RabbitMQ, catalogue, user, cart, shipping, payment, frontend (nginx).

## What was hardened

- Inventory uses SSH key auth — no plaintext passwords committed
- Service units load config from `/etc/roboshop/*.env` (mode `0600`), not hardcoded IPs in unit files
- Hostnames / secrets live in `group_vars/all.yml` (override locally; do not commit real passwords to public forks)
- Markdown junk (`// highlight-start`) that would break systemd was removed
- Empty / junk files (`redis.yaml`, `results.json`) removed

## Prerequisites

```bash
# Ansible 2.14+ with collections
ansible-galaxy collection install amazon.aws community.general community.mysql community.rabbitmq

# AWS credentials for 01-ec2-r53.yaml
export AWS_PROFILE=...
```

Edit before first run:

1. `group_vars/all.yml` — `ami_id`, `sg_id`, `zone_id`, `domain_name`, passwords
2. `ansible.cfg` — `private_key_file`
3. `inventory.ini` — hostnames or private IPs of your instances

## Run order

```bash
# 1) Optional: create EC2 + Route53 from the control node
ansible-playbook 01-ec2-r53.yaml

# 2) Data tier
ansible-playbook 02-mongodb.yaml 03-redis.yaml 04-mysql.yaml 05-rabbitmq.yaml

# 3) App tier
ansible-playbook 06-catalogue.yaml 07-user.yaml 08-cart.yaml 09-shipping.yaml 10-payment.yaml

# 4) Edge
ansible-playbook 11-frontend.yaml
```

Or all application playbooks after hosts exist:

```bash
ansible-playbook 02-mongodb.yaml 03-redis.yaml 04-mysql.yaml 05-rabbitmq.yaml \
  06-catalogue.yaml 07-user.yaml 08-cart.yaml 09-shipping.yaml 10-payment.yaml \
  11-frontend.yaml
```

## Layout

```
01-ec2-r53.yaml … 11-frontend.yaml   numbered playbooks
group_vars/all.yml                   shared hostnames + secrets
inventory.ini                        hosts (DNS or IP)
ansible.cfg                          key-based SSH defaults
templates/nginx.conf.j2              frontend reverse-proxy
*.service                            systemd units (EnvironmentFile-based)
```

## Security notes

- Default demo passwords in `group_vars/all.yml` are for lab use only. For anything shared, move secrets to Ansible Vault.
- Redis / Mongo listen on `0.0.0.0` as required by the RoboShop architecture — restrict with security groups.
- Prefer private Route53 zones (`*.roboshop.internal`) over public IPs in service config.

## License

MIT — see [LICENSE](LICENSE).
