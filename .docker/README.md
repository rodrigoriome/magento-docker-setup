# Magento 2 Docker Setup

### Usage

`cd` into the Magento root then run the following command:

```sh
$ npx degit rodrigoriome/magento-docker-setup -f && sed -i -e "s|__MAGENTO__ROOT__|$(pwd)|" bin/sh
```

This will download the files of this repo into your project, putting files at their respective paths, and then set Magento root dir in the script (`./bin/sh`).

### Stack

- NGINX 1.18
- PHP 7.4.12
- MariaDB 10.4
- Elasticsearch 7.6.2
- N98 Magerun 2
- OPcache
- PhpMyAdmin
- Redis 6.0.9
- XDebug 2.8.1

### Requirements

- docker
- docker-compose

### Troubleshoot

- PhpMyAdmin error
    - Problem: `Cannot connect: invalid settings.`
    - Problem: `mysqli::real_connect(): (HY000/2002): Connection refused`
    - Solution: Wait for MySQL Server to start (try again in 1 minute).

- ElasticSearch permission error
    - Problem: `AccessDeniedException /usr/share/elasticsearch/data/nodes`
    - Solution: Fix the permissions of the host directory mapped to /usr/share/elasticsearch/data. (https://techoverflow.net/2020/04/18/how-to-fix-elasticsearch-docker-accessdeniedexception-usr-share-elasticsearch-data-nodes/)
        - Ex.: `sudo chown -R 1000:1000 ./docker/elasticsearch`
