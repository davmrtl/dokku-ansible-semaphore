# dokku-ansible-semaphore

References : https://docs.ansible-semaphore.com/administration-guide/installation#docker

This project contains Ansible Semaphore that will be deployed as an app on a Dokku host.

## Run [Ansible Semaphore](https://github.com/ansible-semaphore/semaphore) on Dokku host

### 1. Create semaphore app

Installs Ansible Semaphore using [dokku-mariadb](https://github.com/dokku/dokku-mariadb)

```sh
# Create app
dokku apps:create semaphore

# Create database (mariadb)
dokku mariadb:create semaphore-db

# Link app to database
dokku mariadb:link semaphore-db semaphore

# Configure app env
dokku config:set semaphore SEMAPHORE_DB_USER=mariadb
dokku config:set semaphore SEMAPHORE_DB_PASS=<pass>
dokku config:set semaphore SEMAPHORE_DB_HOST=dokku-mariadb-semaphore-db
dokku config:set semaphore SEMAPHORE_DB_PORT=3306
dokku config:set semaphore SEMAPHORE_DB_DIALECT=mysql
dokku config:set semaphore SEMAPHORE_DB=semaphore-db
dokku config:set semaphore SEMAPHORE_PLAYBOOK_PATH=/tmp/semaphore/
dokku config:set semaphore SEMAPHORE_ADMIN_PASSWORD=changeme
dokku config:set semaphore SEMAPHORE_ADMIN_NAME=admin
dokku config:set semaphore SEMAPHORE_ADMIN_EMAIL=admin@localhost
dokku config:set semaphore SEMAPHORE_ADMIN=admin
dokku config:set semaphore SEMAPHORE_ACCESS_KEY_ENCRYPTION=gs72mPntFATGJs9qK0pQ0rKtfidlexiMjYCH9gWKhTU=
dokku config:set semaphore SEMAPHORE_LDAP_ACTIVATED='no'
```

### 2. Deploy semaphore app

```sh
dokku git:sync --build semaphore https://github.com/davmrtl/dokku-ansible-semaphore.git main
```

### 3. Configure networking on semaphore app

Map ports and add SSL using [dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt):
```sh
# Map ports
dokku ps:stop semaphore
dokku config:set semaphore DOKKU_PROXY_PORT_MAP="http:80:3000"

# Add SSL
dokku config:set --no-restart semaphore DOKKU_LETSENCRYPT_EMAIL=your@email.tld
dokku letsencrypt:enable semaphore

# Check if ports are mapped correctly
dokku config:get semaphore DOKKU_PROXY_PORT_MAP

# Should output: "http:80:9000 https:443:9000"
```

To re-deploy Ansible Semaphore, first stop the semaphore container (it's using the Docker socket)

```sh
dokku ps:stop semaphore
dokku git:sync --build semaphore https://github.com/davmrtl/dokku-ansible-semaphore.git main
```
