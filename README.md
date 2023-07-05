# config_keepalived_haproxy
- Ubuntu 22.04
- Keepalived 2.2.8
- Haproxy 2.8

## Config sysctl

Cho phếp gán Virtual IP lên interface, chúng ta cần vào file /etc/sysctl.conf và thêm dòng sau vào file sysctl.conf:

```
net.ipv4.ip_nonlocal_bind=1
```

## Install keepalived

To install keepalived, simply use the following command:
```
sudo snap install keepalived --classic
```

## Config keepalived

### Node 1
```
global_defs {
        router_id ka1                   # Khai bao route_id của keepalived
}

vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
}

vrrp_instance VI_1 {
        virtual_router_id 51
        advert_int 1
        priority 100
        state MASTER                    # Chi 1 MASTER conf lai la BACKUP
        interface eth0                  # thông tin tên interface của server, bạn dùng lệnh `ifconfig` để xem và điền cho đúng
        virtual_ipaddress {
                172.16.8.134 dev eth0   # Khai báo Virtual IP cho interface tương ứng
        }
        authentication {
                auth_type PASS
                auth_pass 12345678      # Password này phải khai báo giống nhau giữa các server keepalived, su dung 8 ky tu dau
        }
        track_script {
                chk_haproxy
        }
}
```
### Node 2
```
global_defs {
        router_id ka2                   #khai báo route_id của keepalived
}

vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
}

vrrp_instance VI_1 {
        virtual_router_id 51
        advert_int 1
        priority 99
        state BACKUP
        interface eth0                  # thông tin tên interface của server, bạn dùng lệnh `ifconfig` để xem và điền cho đúng
        virtual_ipaddress {
                172.16.8.134 dev eth0   # Khai báo Virtual IP cho interface tương ứng
        }
        authentication {
                auth_type PASS
                auth_pass 12345678      # Password này phải khai báo giống nhau giữa các server keepalived, su dung 8 ky tu dau
        }
        track_script {
                chk_haproxy
        }
}
```

## Run keepalived
```
systemctl start snap.keepalived.daemon
```

## Install haproxy

Vào https://haproxy.debian.net/ chọn đúng phiên bản muốn cái đặt. Sau đó copy & past!

## Check config Haproxy
haproxy -f /path/to/haproxy.cfg -c


## Tham khảo
- https://www.keepalived.org/
- https://haproxy.debian.net/#distribution=Ubuntu&release=jammy&version=2.8
