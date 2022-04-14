---
title: Desafio 2 - VM 08
author: R3d-4rr0w
date: 2022-04-14 05:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM8"
---

# VM 8 - Sunset

---

## [VM 8 - Sunset](https://www.vulnhub.com/entry/sunset-1,339/)

---

## Reconhecimento

NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -v 192.168.56.11
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-11 19:34 EDT
PORT   STATE SERVICE VERSION
21/tcp open  ftp     pyftpdlib 1.5.5
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vamos ver se o FTP aceita o login com usuário anonymous.

<img src="/img/desafio2/vm8/sunset.png">

Conseguimos fazer o login no FTP como anonymous e vemos que dentro do diretório tem um arquivo chamado backup.

Vamos fazer download dele usando o comando get.

<img src="/img/desafio2/vm8/sunset 1.png">

Ao ver o conteúdo do arquivo, vemos que há alguns usuários junto com a hash da senha.

<img src="/img/desafio2/vm8/sunset 2.png">

---

## Exploração

Vamos usar o john para quebrar a hash e conseguir a senha.

Conseguimos a senha do usuário sunset.

<img src="/img/desafio2/vm8/sunset 3.png">

Vimos antes no scan do nmap, que tem um serviço de SSH rodando na porta 22 no servidor, vamos acessar usando as credencias do usuário sunset que conseguimos.

<img src="/img/desafio2/vm8/sunset 4.png">

---

## Escalagem de Privilégio

Conseguimos acessar o servidor usando o SSH.

Vamos usar o comando sudo -l para ver o que podemos fazer como root.

<img src="/img/desafio2/vm8/sunset 5.png">

O usuário consegue usar o editor de texto ED com privilégios de root.

>**💡[The GNU ed line editor](https://www.gnu.org/software/ed/manual/ed_manual.html)**


Vamos usar o editor de texto para adicionar um novo usuário com privilégios elevados no /etc/passwd

<img src="/img/desafio2/vm8/sunset 6.png">

Agora podemos trocar o usuário para o que criamos e ter privilégio elevado no servidor.

<img src="/img/desafio2/vm8/sunset 7.png">