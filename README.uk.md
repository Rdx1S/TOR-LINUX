# 🧅 Проброс усього трафіку через Tor на Kali Linux

<div align="center">

[English](README.md) · **Українська**

Спрямуйте **весь** вихідний трафік на Kali Linux через мережу Tor і зробіть це налаштування стійким до перезавантажень за допомогою systemd-сервісу.

![Tor](https://img.shields.io/badge/Tor-7E4798?style=for-the-badge&logo=torproject&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)

</div>

> ⚠️ **Лише для освітнього та авторизованого використання.** Проброс усього трафіку через Tor суттєво змінює поведінку мережі. Використовуйте відповідально й лише на системах, якими ви володієте або маєте дозвіл тестувати.

---

## 📑 Зміст

- [1. Встановлення Tor](#1-встановлення-tor)
- [2. Налаштування Tor](#2-налаштування-tor)
- [3. Встановлення Privoxy](#3-встановлення-privoxy)
- [4. Увімкнення IP-форвардингу](#4-увімкнення-ip-форвардингу)
- [5. Застосування змін](#5-застосування-змін)
- [6. Перенаправлення трафіку через iptables](#6-перенаправлення-трафіку-через-iptables)
- [7–12. Збереження після перезавантаження](#712-збереження-після-перезавантаження)

---

## 1. Встановлення Tor

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install tor
```

---

## 2. Налаштування Tor

Відредагуйте конфігураційний файл Tor:

```bash
sudo nano /etc/tor/torrc
```

Розкоментуйте рядок `SocksPort 9050` (або вкажіть будь-який інший вільний порт) як порт SOCKS.

---

## 3. Встановлення Privoxy

```bash
sudo apt install privoxy
```

Відредагуйте конфігурацію Privoxy:

```bash
sudo nano /etc/privoxy/config
```

- Розкоментуйте `listen-address localhost:8118` і замініть `localhost` на `127.0.0.1`.
- Додайте такий рядок у **кінець** файлу, щоб перенаправляти трафік на Tor:

```
forward-socks5 / 127.0.0.1:9050 .
```

---

## 4. Увімкнення IP-форвардингу

Відредагуйте конфігурацію sysctl:

```bash
sudo nano /etc/sysctl.conf
```

Розкоментуйте рядок:

```
net.ipv4.ip_forward=1
```

---

## 5. Застосування змін

```bash
sudo sysctl -p
```

---

## 6. Перенаправлення трафіку через iptables

Додайте правило NAT, яке перенаправляє вихідний TCP-трафік через Privoxy:

```bash
sudo iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```

Тепер увесь ваш трафік має йти через Tor. Перевірте це, переглянувши свою публічну IP-адресу — вона має належати мережі Tor:

```bash
curl ident.me
```

---

## 7–12. Збереження після перезавантаження

Правило iptables скидається після перезавантаження. Загорніть його у скрипт і зареєструйте як systemd-сервіс, щоб воно запускалося автоматично.

**7. Створіть скрипт** у каталозі `/usr/local/bin` (стандартне місце для користувацьких бінарних файлів):

```bash
sudo nano /usr/local/bin/tort.sh
```

**8. Додайте туди такий вміст:**

```bash
#!/bin/bash
iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```

**9. Зробіть його виконуваним:**

```bash
sudo chmod ugo+x /usr/local/bin/tort.sh
```

**10. Створіть systemd-юніт**, який запускатиме скрипт:

```bash
sudo systemctl edit --force --full script.service
```

Команда відкриє текстовий редактор — вставте:

```ini
[Unit]
Description=My Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/local/bin/tort.sh

[Install]
WantedBy=multi-user.target
```

> 💡 У рядку `ExecStart` можна вказати або шлях до скрипта, або саму команду `iptables` напряму. Зі скриптом зручніше — його можна просто редагувати, додаючи команди, які ви хочете запускати під час старту системи.

**11. Додайте сервіс в автозапуск:**

```bash
sudo systemctl enable script
```

**12. Якщо systemd не бачить сервіс,** оновіть інформацію про юніти:

```bash
sudo systemctl daemon-reload
```
