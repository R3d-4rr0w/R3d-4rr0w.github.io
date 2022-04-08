---
title: Desafio 2 - VM 02
author: R3d-4rr0w
date: 2022-04-08 15:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Struts]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM2"
---

# VM 2 - Struts S2-052

---

## [VM 2 - **Struts S2-052**](https://www.vulnhub.com/entry/pentester-lab-s2-052,206/)

---

## Reconhecimento

SCAN NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -p- 192.168.56.5 -v
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 08:35 EDT
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
```

Acessando a porta 80 pelo browser, temos uma aplicação web.

<img src="/img/desafio2/vm2/struts-0.png">

---

## Exploração

Como o LAB já está dizendo que é uma vulnerabilidade no Struts 2, economizamos tempo no reconhecimento e podemos ir direto para a exploração.

Procurando por exploits do struts 2 no metasploit, encontramos um RCE.

<img src="/img/desafio2/vm2/struts-1.png">

Vamos configurar o exploit com as informações que precisa. 

<img src="/img/desafio2/vm2/struts-2.png">

Ao executar o exploit conseguimos a conexão reversa.

Como visto, já estamos como root e não vamos precisar escalar privilégios.

<img src="/img/desafio2/vm2/struts-3.png">