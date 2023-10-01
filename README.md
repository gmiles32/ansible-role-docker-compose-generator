# ansible-role-docker-compose-generator

A fork of [this repository](https://github.com/ironicbadger/ansible-role-docker-compose-generator). I really liked the way this role was set up and how it worked, but it didn't really fit into how my homelab is configured. I use individual `docker-compose.yml` files to keep services seperated, and the current role didn't really allow for multiple configurations. Piggy backing off of the previous role, I allowed for looping the same tasks, using a list of service names specified in the `group_vars` file. For each config, it will create a direcotry and write a `docker-compose.yml` to that directory on the remote machine.

Additionally, it is now possible to specifiy volume and network options in you config (WIP).

---

Pass this role a hash and it will generate a series of docker-compose.yml file. The following structure is supported and is designed to be passed to the role using `group_vars`.

Rendered files are output to the `output` directory.

---


```
---

# global vars
global_env_vars:
  - "PUID=1313"
  - "PGID=1313"
appdata_path: /opt/appdata
container_config_path: /config
container_data_path: /data
services:
  - list
  - of
  - services
services_config_path: /path/to/service/configs

# container definitions
containers:
  - service_name: letsencrypt
    active: true
    image: linuxserver/letsencrypt
    container_name: le
    ports:
      - 80:80
      - 443:443
    volumes:
      - "{{ appdata_path }}/letsencrypt/config:{{ container_config_path }}"
    restart: always
    depends_on:
      - unifi
      - nextcloud
      - quassel
    include_global_env_vars: true
    environment:
      - EMAIL=email@email.com
      - URL=some.tld
      - SUBDOMAINS=nc, irc, unifi
      - ONLY_SUBDOMAINS=true
      - DHLEVEL=4096
      - TZ=Europe/London
      - VALIDATION=http
    mem_limit: 256m
  - service_name: nextcloud
    active: true
    image: nextcloud
    container_name: nextcloud
    include_global_env_vars: false
    volumes:
      - "{{ appdata_path }}/nextcloud/html:/var/www/html"
      - "{{ appdata_path }}/nextcloud/apps:/var/www/html/custom_apps"
      - "{{ appdata_path }}/nextcloud/config:/var/www/html/config"
      - "{{ appdata_path }}/nextcloud/data:/var/www/html/data"
      - "{{ appdata_path }}/nextcloud/theme:/var/www/html/themes"
    mem_limit: 256m
    restart: "{{ unless_stopped }}"
    ports:
      - "8081:80"
  - service_name: unifi
    active: true
    image: linuxserver/unifi
    container_name: unifi
    mem_limit: 512m
    volumes:
      - "{{ appdata_path }}/unifi:{{ container_config_path }}"
    include_global_env_vars: true
    restart: "{{ unless_stopped }}"
  - service_name: quassel
    active: true
    image: linuxserver/quassel
    container_name: quassel
    include_global_env_vars: true
    volumes:
      - "{{ appdata_path }}/quassel:{{ container_config_path }}"
    mem_limit: 128m
    ports:
      - "4242:4242"

networks:
  - network_name: frontend
    external: true

volumes:
  - volume_name: tmp
    driver: local
    driver_opts:
      - name:
        value
```