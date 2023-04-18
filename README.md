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
