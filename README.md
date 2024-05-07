# DuckDNS ve Traefik ile Uzaktan Erişim kurulumu

## Önkabuller
- Ubuntu(Debian) ve Ssh erişimi
- Kurulu ve çalışan bir Docker
    ``` bash 
    $ curl -fsSL https://get.docker.com -o get-docker.sh
    $ sudo sh ./get-docker.sh --dry-run
    ```
- Statik ip ve modemde açık port
    Wireguard 51820
```
curl -fsSL https://get.casaos.io | sudo bash
```



## Plan
3. Traefik için network
    ```bash
    sudo docker network create traefik
    ```
4. Traefik düzenle ve çalıştır
3. Güncellemeler için Watchtower
4. Reverse Proxy Traefik
5. Dinamik dns update
6. Wireguard

Nth. Yeni servis eklemek