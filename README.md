# TOR-LINUX
Настройка проброса всего трафика на Kali Linux через TOR и добавление команды в автозагрузку

Чтобы пробросить весь трафик в Kali Linux через Tor, вам нужно выполнить несколько шагов:

1. Установите Tor, выполнив следующую команду в терминале:

```
sudo apt update && sudo apt upgrade -y
sudo apt nstall tor
```
2. Настройте Tor, отредактировав файл конфигурации /etc/tor/torrc 

```
nano /etc/tor/torrc
```
Раскомментируйте строку #SocksPort 9050 и установите SocksPort 9050 (или любой другой свободный порт) в качестве порта SOCKS. 

3. Установите прокси-сервер privoxy:

```
sudo apt install privoxy
```
Отредактируйте файл конфигурации privoxy /etc/privoxy/config. 
```
nano  /etc/privoxy/config
```

Раскомментируйте строку listen-address localhost:8118 и замените localhost на 127.0.0.1.
Добавьте следующие строки в конец файла конфигурации privoxy:

```
forward-socks5 / 127.0.0.1:9050 .
```
4. Настройте маршрутизацию трафика

Отредактируйте файл конфигурации /etc/sysctl.conf и раскомментируйте строку net.ipv4.ip_forward=1.
```
nano /etc/sysctl.conf
```
5. Введите следующую команду для активации изменений в конфигурации:

```
sudo sysctl -p
```
6. Добавьте правило iptables для перенаправления трафика через прокси-сервер:
```
sudo iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```
Теперь весь ваш трафик должен быть перенаправлен через Tor. Вы можете проверить это, посетив сайт, который показывает ваш IP-адрес, и убедившись, что он отображает IP-адрес Tor сети или ввести команду в терминале:
```
curl ident.me
```
7. Создайте скрипт в папке /usr/local/bin. Эта папка обычно используется для хранения пользовательских бинарных файлов.
```
sudo nano /usr/local/bin/tort.sh
```
8. Добавьте туда следующий текст:
```
#!/bin/bash
iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```
9. Когда скрипт будет готов, сделайте его исполняемым:
```
sudo chmod ugo+x /usr/local/bin/tort.sh
```
10. В Systemd нет способа запускать все пользовательские скрипты в одном месте. Но вы можете создать юнит файл, который будет запускать ваш скрипт. Для этого используйте следующую команду:
```
sudo systemctl edit --force --full script.service
```
Команда откроет текстовый редактор, добавьте в него такое содержимое:
```
[Unit]
Description=My Script Service
After=multi-user.target
[Service]
Type=idle
ExecStart=/usr/local/bin/tort.sh
[Install]
WantedBy=multi-user.target
```
В строчке ExecStart можно прописать либо путь к скрипту, который надо выполнить, либо саму команду `iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118` но так удобнее, можно просто редактировать скрипт добавляя туда свои команды, которые вы хотели бы в дальнейшем запускать при запуске системы.

11. Теперь добавьте этот скрипт в автозагрузку:
```
sudo systemctl enable srcipt
```
12. Если Systemd не видит такого сервиса, обновите информацию о юнитах с помощью команды:
```
sudo systemctl daemon-reload
```





