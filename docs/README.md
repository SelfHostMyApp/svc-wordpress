## WordPress Service

WordPress 6.5 (PHP 8.2 Apache variant) service contract ready for rendering through the shared infrastructure roles. The defaults capture database wiring, persistent storage, runtime resources, and exports so multiple runtimes can be generated from the same variables.

### Runtime Coverage
- Proxmox LXC with Apache/PHP packages managed inside the container
- Docker Compose v2
- Podman Quadlet
- Kubernetes Deployment + Service + PVC + Secret
- Bare-metal systemd service (Apache)

### Dependency
- `mariadb` – ensures the MariaDB service is provisioned before WordPress. Override `wordpress_db_host`, `wordpress_db_name`, `wordpress_db_user`, and `wordpress_db_password` to align with the database credentials that service exports.

### Exports
```
WORDPRESS_URL={{ wordpress_site_url }}
WORDPRESS_PORT={{ wordpress_service_port }}
```

### Secrets
- `WORDPRESS_DB_PASSWORD` -> database password consumed by the container. Defaults to `WORDPRESS_DB_PASSWORD` environment lookup with a placeholder fallback.

### Health Check
`curl -fsS http://127.0.0.1:{{ wordpress_service_port }}/wp-login.php` confirms Apache/PHP renders the login page. Used for Compose healthchecks, Quadlet probes, Kubernetes readiness/liveness, and the post-deploy verification.

### Key Overrides
| Variable | Default | Purpose |
| --- | --- | --- |
| `wordpress_service_port` | `8080` | Published HTTP port |
| `wordpress_db_host` | `mariadb:3306` | Host:port of the backing MariaDB |
| `wordpress_db_name` | `appdb` | Schema used by WordPress |
| `wordpress_db_user` | `app` | Database user |
| `wordpress_data_volume` | `wordpress-data` | Persistent content volume name |
| `wordpress_container_vmid` | `210` | Proxmox VMID |
| `wordpress_container_ip` | `192.168.100.12` | LXC container address |
| `wordpress_container_storage_gb` | `20` | Storage allocation for uploads/plugins |
| `wordpress_kubernetes_namespace` | `apps` | Namespace for the Deployment/Service/PVC |

Adjust these in inventory to point WordPress at the desired database, runtime resources, and network settings. You can extend the defaults with any other keys supported by the shared templates (for example extra volumes or Kubernetes annotations).

### Usage
```yaml
- hosts: web_hosts
  roles:
    - role: svc-wordpress
      vars:
        runtime: kubernetes
        wordpress_site_url: https://blog.example.com
        wordpress_db_host: mariadb:3306
        wordpress_db_user: blog
        wordpress_db_password: "{{ vault_wordpress_db_password }}"
```
