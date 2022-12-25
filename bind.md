# Install bind 9 in AlmaLinux 8.7

## Подготовка сервера
Устанавливаем все обновления:
```
yum update
```
Устанавливаем утилиту для синхронизации времени:
```
yum install chrony
```
Настраиваем временную зону:
```
timedatectl set-timezone Europe/Moscow
```
[* в данном примере выбрано московское время.]

Разрешаем и запускаем сервис для синхронизации времени:
```
systemctl enable chronyd --now
```
Открываем порт в firewall:
```
firewall-cmd --permanent --add-port=53/udp
```
И перечитываем настройки сетевого экрана:
```
firewall-cmd --reload
```
## Установка и запуск BIND
Устанавливаем DNS-сервер следующей командой:
```
yum install bind
```
Разрешаем автозапуск:
```
systemctl enable named
```
Запускаем сервис имен:
```
systemctl start named
```
И проверяем, что он работает корректно:
```
systemctl status named
```

## Настройка DNS-сервера
Редактирование конфигурационного файла bind. 
Добавляем поддержку двух зон: `home.local` и `1.168.192.in-addr.arpa`.
```
vi /etc/named.conf
```
```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };

        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "home.local" IN {
        type master;
        file "home.local";
};

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "1";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Файлы описания зон находятся в директории /var/named.

home.local:
```
$TTL 86400
@       IN      SOA     controller.home.local. postmaster.example.com. (
                                2022122403      ;serial
                                3600            ;refresh
                                1800            ;retry
                                604800          ;expire
                                86400           ;minimum ttl
)
;
@       IN      NS      controller
;
controller      IN      A       192.168.1.210
master1         IN      A       192.168.1.211
master2         IN      A       192.168.1.212
master3         IN      A       192.168.1.213
worker1         IN      A       192.168.1.214
worker2         IN      A       192.168.1.215
worker3         IN      A       192.168.1.216
```
1:
```
$TTL 86400
@ IN SOA controller.home.local. postmaster.example.com. (
                                2022122403 ;Serial
                                3600 ;Refresh
                                1800 ;Retry
                                604800 ;Expire
                                86400 ;Minimum TTL
)
;
@ IN NS controller.home.local.
;
210     IN      PTR     controller.home.local.
211     IN      PTR     master1.home.local.
212     IN      PTR     master2.home.local.
213     IN      PTR     master3.home.local.
214     IN      PTR     worker1.home.local.
215     IN      PTR     worker2.home.local.
216     IN      PTR     worker3.home.local.
```