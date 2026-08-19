# Дело "О настройке системы"

## Цель

Развернуть изолированный полигон из трёх виртуальных машин на UTM: атакующая сторона, уязвимая цель, защитная сторона со сбором логов

## Архитектура

|VM|роль|IP|ресурсы|
|---|---|---|---|
|Kali Linux|атака|192.168.100.10/24|8 GB RAM, 4 CPU, 60 GB|
|Metasploitable 2|цель|192.168.100.20/24|1 GB RAM, 1 CPU, 10 GB|
|ubuntu-defender|защита|192.168.100.30/24|2 GB RAM, 2 CPU, 20 GB|

Сеть — UTM Host Only, без выхода в интернет и без доступа из внешней сети

---

## Ход работы

### Установка Kali Linux

VM создана в режиме **Virtualize** (нативная ARM64-виртуализация). Установлен стандартный набор — Xfce, top10 tools. Проблем не возникло

### Установка Metasploitable 2

**Симптом:** При загрузке VM в режиме Virtualize — чёрный экран `UEFI Interactive Shell`, система не находит загрузочный том

**Причина:** Metasploitable 2 — система 2010 года на классической схеме BIOS/MBR. ARM64-виртуализация в UTM работает только через UEFI, у ARM нет концепции legacy BIOS. Диск в формате MBR физически не может загрузиться через UEFI-путь

**Решение:** VM пересоздана в режиме **Emulate**, архитектура x86_64. Это включает полноценную программную эмуляцию x86-процессора, поддерживающую classic BIOS boot

**Побочный инцидент:** После смены архитектуры диск был подключён с интерфейсом IDE — на ARM64/Virtualize эта шина не существует физически, ошибка `Bus 'ide.0' not found`. После перехода на Emulate x86_64 IDE стал доступен штатно

**Финальная конфигурация:** Emulate, x86_64, UEFI boot выключен, Boot Device — Drive Image, диск подключён через Import Drive (не создан заново)

### Установка ubuntu-defender

Ubuntu Server 26.04 LTS ARM64, установка через Subiquity без отклонений от стандартного процесса. OpenSSH включён на этапе установки

---

## Инциденты сетевой настройки

### Инцидент 1 — DHCP не работает в Host Only

**Симптом:** После переключения сети всех трёх VM на Host Only — Kali и Metasploitable остались без IPv4-адреса, только link-local IPv6

**Причина:** UTM Host Only не поднимает DHCP-сервер

**Решение:** Статическая адресация на всех трёх машинах:

- Kali — через NetworkManager (`nmcli con mod`)
- Metasploitable — `/etc/network/interfaces`
- ubuntu-defender — Netplan (`/etc/netplan/00-installer-config.yaml`, интерфейс `enp0s1`)

### Инцидент 2 — Permission denied на приём логов

**Симптом:** rsyslog на ubuntu-defender запущен (`active running`), но лог не пишется: `open error: Permission denied`

**Причина:** Демон работает от непривилегированного пользователя (`syslog`), директория `/var/log/remote/` создана через `sudo mkdir` и принадлежит root

**Решение:**
```bash
sudo chmod 777 /var/log/remote
sudo systemctl restart rsyslog
```

### Инцидент 3 — зависший dhclient сбрасывает статику

**Симптом:** После временного переключения Metasploitable на Shared Network (для установки rsyslog через apt-get) и возврата на статику — интерфейс периодически терял `192.168.100.20`, в логе непрерывные `DHCPDISCOVER`

**Причина:** Процесс `dhclient3`, запущенный во время работы на Shared Network, не завершился при возврате на статический конфиг и продолжал фоновые попытки получить адрес по DHCP

**Решение:**
```bash
ps aux | grep dhclient
sudo kill <PID>
sudo /etc/init.d/networking restart
```

### Инцидент 4 — nmap зависает на неопределённое время

**Симптом:** `ping` до Metasploitable проходит стабильно, `nc` подключается к портам напрямую, но любой `nmap`-скан (включая `-Pn`) висит без ответа неограниченное время

**Причина:** В изолированной сети нет DNS-сервера. nmap по умолчанию выполняет reverse DNS lookup перед сканированием и ожидает таймаут DNS-запроса на каждый хост

**Решение:** Флаг `-n` — отключает DNS-резолвинг:
```bash
nmap -n -Pn 192.168.100.20
```

Скан отработал за 0.11 секунды вместо бесконечного ожидания

---

## Результат

```
Nmap scan report for 192.168.100.20
Host is up (0.00061s latency).
Not shown: 82 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
513/tcp  open  login
514/tcp  open  shell
2049/tcp open  nfs
2121/tcp open  ccproxy-ftp
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
6000/tcp open  X11
8009/tcp open  ajp13
```

rsyslog-форвардинг с Metasploitable на ubuntu-defender подтверждён — события системного уровня (cron, auth) поступают в `/var/log/remote/metasploitable.log` в реальном времени
## Стек

`UTM` `Kali Linux` `Metasploitable 2` `Ubuntu Server` `rsyslog` `nmap`