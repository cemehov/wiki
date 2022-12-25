### Install bind 9 in AlmaLinux 8.7

Устанавливаем все обновления:

# yum update

Устанавливаем утилиту для синхронизации времени:

# yum install chrony

Настраиваем временную зону:

# timedatectl set-timezone Europe/Moscow

* в данном примере выбрано московское время. 

Разрешаем и запускаем сервис для синхронизации времени:

# systemctl enable chronyd --now

Открываем порт в firewall:

# firewall-cmd --permanent --add-port=53/udp

И перечитываем настройки сетевого экрана:

# firewall-cmd --reload

Устанавливаем DNS-сервер следующей командой:

# yum install bind

Разрешаем автозапуск:

# systemctl enable named

Запускаем сервис имен:

# systemctl start named

И проверяем, что он работает корректно:

# systemctl status named
