```
Задание 1:
1) запустить контейнер с ubuntu, используя механизм LXC
2) ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает
3) добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно
4) при создании указать файл, куда записывать логи
5) после перезагрузки проанализировать логи

Задание 2*: настроить автоматическую маршрутизацию между контейнерами. Адреса можно взять: 10.0.12.0/24 и 10.0.13.0/24.
```

---

1) Установка LXC:
```
sudo apt-get update
sudo apt-get install lxc
```

2) Запуск контейнера с Ubuntu и ограничение ОЗУ до 256 Мб:
```
sudo lxc-create -n homework02 -t download -- -d ubuntu -r focal -a amd64 --limit memory=256MB
```

3) Создание файла конфигурации для контейнера в директории /etc/lxc/auto:
```
sudo nano /etc/lxc/auto/homework02.conf
```
Вставьте следующий текст в файл:
```
lxc.start.auto = 1
lxc.start.delay = 5
lxc.start.order = 100
lxc.autodev = 1
lxc.tty.max = 4
lxc.uts.name = homework02
lxc.cgroup.memory.limit_in_bytes = 268435456
```

4) Перезагрузка ОС:
```
sudo reboot
```

5) Проверка, что контейнер действительно запустился самостоятельно:
```
sudo lxc-ls --active
```

6) Указание файла для записи логов при создании контейнера:
```
sudo lxc-create -n homework02 -t download --logfile=/var/log/homework02.log -- --dist ubuntu --release focal --arch amd64
```


---

1) Создание двух контейнеров:
```
sudo lxc-create -n container1 -t download -- -d ubuntu -r focal -a amd64
sudo lxc-create -n container2 -t download -- -d ubuntu -r focal -a amd64
```

2) Настройка сетевых интерфейсов для контейнеров:
```
sudo nano /etc/network/interfaces
```
Вставьте следующий текст в файл:
```
auto lxcbr0
iface lxcbr0 inet static
    bridge_ports none
    bridge_fd 0
    address 10.0.12.1
    netmask 255.255.255.0

auto lxcbr1
iface lxcbr1 inet static
    bridge_ports none
    bridge_fd 0
    address 10.0.13.1
    netmask 255.255.255.0
```

3) Запуск контейнеров:
```
sudo lxc-start -n homework02_1
sudo lxc-start -n homework02_2
```

4) Настройка маршрутизации на хосте:
```
sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 ! -d 10.0.12.0/24 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 ! -d 10.0.13.0/24 -j MASQUERADE

sudo sysctl net.ipv4.ip_forward=1
```

5) Настройка маршрутизации в контейнерах:
```
sudo nano /etc/network/interfaces
```
Вставьте следующий текст в файл для контейнера 1:
```
auto eth0
iface eth0 inet static
    address 10.0.12.2
    netmask 255.255.255.0
    gateway 10.0.12.1

auto eth1
iface eth1 inet static
    address 10.0.13.2
    netmask 255.255.255.0

post-up route add -net 10.0.13.0/24 gw 10.0.12.1 dev eth0 || true

post-up echo "nameserver 8.8.8.8" >> /etc/resolv.conf || true

post-down route del -net 10.0.13.0/24 gw 10.0.12.1 dev eth0 || true

post-down sed -i '/nameserver 8\.8\.8\.8/d' /etc/resolv.conf || true
```

Вставьте следующий текст в файл для контейнера 2:
```
auto eth0
iface eth0 inet static
    address 10.0.13.3
    netmask 255.255.255.0
    gateway 10.0.13.

auto eth1
iface eth1 inet static
    address 10.

post-up route add -
post-up echo "nameserver 
post-down route del -
post-down sed -
```

6) Перезапуск сетевых интерфейсов и проверка маршрутизации:
```
sudo service networking restart
```