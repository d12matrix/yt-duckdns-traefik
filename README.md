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
```sh
sudo netplan try
```



## Plan
1. DuckDNS.org'da hesap ve domain alımı
2. Traefik için network
    ```bash
    $ sudo docker network create traefik
    ```
2. [Traefik](./traefik/docker-compose.yml) düzenle ve çalıştır
3. PiHole için hazırlık:
    SystemD yüzünden bazı ayarları değiştirmek gerekiyor.
    `/etc/systemd/resolved.conf.d/pihole.conf`
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
6. PiHole'a erişim için Hosts dosyasını editle.
7. PiHole'da domaini sunucu adresine yönlendir.
8. [Wg-Easy](./wgeasy/docker-compose.yml) düzenle ve çalıştır
9. [Watchtower](./watchtower/docker-compose.yml) çalıştır.
10. Dinamik DNS güncellemesi için [DuckDNS](./duckdns/docker-compose.yml) düzenle ve çalıştır. 
11. Yeni servis eklemek:
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
        - "traefik.http.middlewares.pihole-admin-redirect.addPrefix.prefix=/admin"
        - "traefik.http.routers.pihole.middlewares=pihole-admin-redirect"
        - "traefik.http.services.new-serviced.loadbalancer.server.port=8080"
        - "traefik.http.routers.new-service.service=new-service"
    ```

## Daha fazlası:
- wg-easy yerine Traefik
- Cloudflared
- Direkt bağlantı