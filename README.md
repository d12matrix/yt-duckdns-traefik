# DuckDNS ve Traefik ile Uzaktan Erişim kurulumu

## Önkabuller
- Ubuntu(Debian) ve Ssh erişimi
- Kurulu ve çalışan bir Docker
``` sh 
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh ./get-docker.sh --dry-run
```
- Statik ip ve modemde açık port
    Wireguard 51820

`/etc/netplan/01-network-manager-all.yaml`
```yaml
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     addresses: [192.168.1.1/24]
     gateway4: 192.168.1.1
     nameservers:
         addresses: [8.8.8.8,8.8.8.4]
```



## Plan
1. Traefik için network
    ```bash
    $ sudo docker network create traefik
    ```
2. [Traefik](./traefik/docker-compose.yml) düzenle ve çalıştır
3. PiHole için hazırlık:
    SystemD yüzünden bazı ayarları değiştirmek gerekiyor.
    `/etc/systemd/resolved.conf.d/adguardhome.conf`
    ```toml
    [Resolve]
    DNS=127.0.0.1
    DNSStubListener=no
    ```
    127.0.0.1'ı DNS olarak ayarla.
    ```sh
    mv /etc/resolv.conf /etc/resolv.conf.backup
    ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
    systemctl reload-or-restart systemd-resolved
    ```
4. [PiHole](./pihole/docker-compose.yml) düzenle ve çalıştır
5. PiHole için modemde DNS ayarını yap.
6. [Wg-Easy](./wgeasy/docker-compose.yml) düzenle ve çalıştır
7. [Watchtower](./watchtower/docker-compose.yml) çalıştır.
8. Yeni servis eklemek:
    ```yaml
    networks:
      traefik:
        external: true
    ...
    services:
      new-service:
        networks:
          - traefik
        ...
        labels:
        - "traefik.enable=true"
        - "traefik.http.routers.new-service.rule=Host(`new-service.d12matrix.duckdns.org`)"
        - "traefik.http.routers.new-service.entrypoints=websecure"
        - "traefik.http.services.new-serviced.loadbalancer.server.port=8080"
        - "traefik.http.routers.new-service.service=new-service"
    ```

## Daha fazlası:
- wg-easy yerine Traefik
- Cloudflared
- Direkt bağlantı