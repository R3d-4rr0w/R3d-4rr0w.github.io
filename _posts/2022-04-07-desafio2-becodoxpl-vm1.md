---
title: Desafio 2 - VM 01
author: R3d-4rr0w
date: 2022-04-07 19:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, WordPress]
image: /img/desafio2/becoxpl-logo.png
alt: "Desafio 2 - VM 1"
---


# Desafio 02 - Beco do Exploit

Começa aqui o desafio 2 do Beco do Exploit, onde vamos explorar 30 máquinas ao todo, indo desde a exploração automatizada, fazendo uso de exploits existentes até o desenvolvimento de exploit para a exploração das VMs.

O desafio pode ser encontrado no [Canal do YouTube](https://www.youtube.com/watch?v=xnCS8fYfrjs&list=PLHBDBcFA_l_WBcUJWf8cp5BaPsUkquRQU&index=1).

---

## [VM 1 - Hacker Fest 2019](https://www.vulnhub.com/entry/hacker-fest-2019,378/)

---

Reconhecimento:

NMAP:

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -p- 192.168.56.4 -v
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-07 14:05 EDT
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.3
22/tcp    open  ssh      OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.25 ((Debian))
10000/tcp open  ssl/http MiniServ 1.890 (Webmin httpd)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Ao acessar a porta 80 pelo browser, percebemos logo que se trata de uma aplicação em WordPress.

![Untitled](img/desafio2/vm1/hackerfest-0.png)

Ao rodar o wpscan, temos um plugin interessante.

![Untitled](img/desafio2/vm1/hackerfest-1.png)

Procuramos por algo no metasploit com o comando search e encontramos o seguinte módulo:

![Untitled](img/desafio2/vm1/hackerfest-2.png)

Vamos preencher as opções com os dados :

![Untitled](img/desafio2/vm1/hackerfest-3.png)

Ao rodar o exploit, temos acesso ao hash da senha do usuário webmaster:

![Untitled](img/desafio2/vm1/hackerfest-4.png)

Vamos usar o john para quebra de hash.

![Untitled](img/desafio2/vm1/hackerfest-5.png)

Como visto no scan no nmap, temos um serviço de SSH rodando na porta 22.

Conseguimos fazer o login com o usuário webmaster utilizando a senha que conseguimos extrair do hash.

A flag de usuário já está no diretório que estamos.

![Untitled](img/desafio2/vm1/hackerfest-6.png)

Ao rodar o comando para ver o que o usuário pode fazer como root, descobrimos que ele tem acesso total, facilitando a escalagem de privilégios. 

![Untitled](img/desafio2/vm1/hackerfest-7.png)

E dentro do diretório /root , achamos a outra flag.

![Untitled](img/desafio2/vm1/hackerfest-8.png)

---

No scan no nmap, conseguimos ver que na porta 10000, existe um serviço Webmin versão 1.890 rodando.

Ao procurar no searchsploit, vimos que existe um módulo do metasploit, que possibilita um RCE.

![Untitled](img/desafio2/vm1/hackerfest-9.png)

Vamos buscar ele no metasploit e seleciona-lo.

![Untitled](img/desafio2/vm1/hackerfest-10.png)

Vendo as informações do módulo, vemos a CVE-2019-15107.

Ao olhar a CVE, vemos que ela foi feita para a versão 1.920 do Webmin, mas funciona também na versão 1.890, que é a que estamos explorando.

Teremos que forçar a execução desse exploit usando o force exploit.

![Untitled](img/desafio2/vm1/hackerfest-11.png)

Agora vamos preencher as opções do exploit

![Untitled](img/desafio2/vm1/hackerfest-12.png)

Ao executar o exploit, conseguimos o RCE e podemos ver que já estamos como root, não precisando escalar privilégios.

![Untitled](img/desafio2/vm1/hackerfest-13.png)