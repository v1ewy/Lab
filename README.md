<div align="center"> <img src="./assets/matrix_rain.svg" width="700"/> </div> <br/> <div align="center">

[![Typing SVG](https://readme-typing-svg.demolab.com/?font=Fira+Code&weight=600&size=26&duration=3000&pause=900&color=FF0000&center=true&vCenter=true&width=650&lines=Lab;attack+%2B+defense+playground;break+it%2C+log+it%2C+understand+it)](https://git.io/typing-svg)

![Kali](https://img.shields.io/badge/Kali_Linux-000000?style=for-the-badge&logo=kalilinux&logoColor=ff0000) ![Metasploitable](https://img.shields.io/badge/Metasploitable-000000?style=for-the-badge&logo=linux&logoColor=ff0000) ![UTM](https://img.shields.io/badge/UTM-000000?style=for-the-badge&logo=apple&logoColor=ff0000)

</div> <br/>

## `./overview`

Домашний полигон для отработки атаки и защиты. Три виртуальные машины на UTM, изолированная сеть, без выхода наружу. Атакующая сторона — Kali. Цель — Metasploitable. Защитная сторона — отдельная VM, собирает логи и позже получит правила блокировки

<br/>

## `./architecture`

```
[Ubuntu-server] ───┐
		           ├── Host Only 192.168.100.0/24 ── [Kali]
[Metasploitable] ──┘  
```

|VM|роль|IP|ресурсы|
|---|---|---|---|
|Kali Linux|атака|192.168.100.10|8 GB RAM, 4 CPU|
|Metasploitable 2|цель|192.168.100.20|1 GB RAM, 1 CPU|
|ubuntu-defender|защита|192.168.100.30|2 GB RAM, 2 CPU|

<br/>

## `./labs`

| №   | writeup                                                                 | статус | техники                         |
| --- | ----------------------------------------------------------------------- | ------ | ------------------------------- |
| 01  | [Дело "О настройке системы"](./SystemSetup/README.md) | Complite      | UTM, network isolation, rsyslog |

<br/>

## `./stack`

![Kali](https://img.shields.io/badge/Kali_Linux-000000?style=for-the-badge&logo=kalilinux&logoColor=ff0000) ![Nmap](https://img.shields.io/badge/Nmap-000000?style=for-the-badge&logo=nmap&logoColor=ff0000) ![Metasploit](https://img.shields.io/badge/Metasploit-000000?style=for-the-badge&logo=metasploit&logoColor=ff0000) ![rsyslog](https://img.shields.io/badge/rsyslog-000000?style=for-the-badge&logo=linux&logoColor=ff0000)

<br/> <div align="center">

[![Footer typing SVG](https://readme-typing-svg.demolab.com/?font=Fira+Code&weight=500&size=16&duration=3000&pause=1000&color=FF0000&center=true&vCenter=true&width=520&lines=end+of+transmission;keep+building%2C+keep+breaking)](https://git.io/typing-svg)

</div>
