---
title: Desafio 2 - VM 10
author: R3d-4rr0w
date: 2022-04-14 06:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, Log Poisoning]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM10"
---

# VM 10 - The Ether

---

## [VM 10 - The Ether](https://www.vulnhub.com/entry/the-ether-evilscience-v101,212/)

---

## Reconhecimento

NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -p- -v 192.168.56.13
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-12 09:57 EDT
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vamos abrir no browser e ver a aplicação web que está rodando.

<img src="/img/desafio2/vm10/ether.png">

Ao navegar na pagina, vemos que tem um parâmetro file recebendo um arquivo php, nesse caso o about.php

<img src="/img/desafio2/vm10/ether 1.png">

Vamos interceptar essa requisição com o burp, para manipular a requisição e tentar acessar algum arquivo.

Essa é a requisição, e a resposta já no repeater do burp.

<img src="/img/desafio2/vm10/ether 2.png">

Se inserirmos o ../../../etc/passwd no file, ele não retorna nenhum arquivo, mas retorna HTTP 200, e notamos que a linha 9 retorna em branco, que é onde deveria começar o arquivo lido.

<img src="/img/desafio2/vm10/ether 3.png">

Vimos que tem rodando um SSH, vamos tentar acessar os logs de autenticação.

<img src="/img/desafio2/vm10/ether 4.png">

Agora tentamos fazer login no SSH com um usuário qualquer, para ver se reflete no log que conseguimos ver.

<img src="/img/desafio2/vm10/ether 5.png">

<img src="/img/desafio2/vm10/ether 6.png">

---

## Exploração Log-Poisoning

A tentativa de login no SSH foi inserida no auth.log.

Como a página é em PHP, vamos inserir um código PHP, que vai receber um parâmetro para a função system, no login do SSH, para ver se ele vai executar o PHP.

<img src="/img/desafio2/vm10/ether 7.png">

<img src="/img/desafio2/vm10/ether 8.png">

Vamos adicionar nosso parâmetro cmd e enviar o comando ls para testar.

<img src="/img/desafio2/vm10/ether 9.png">

Podemos ver que está executando comandos do sistema e retornando o resultado na página.

Vamos criar um payload malicioso, para criar uma shell reversa usando o msfvenom.

<img src="/img/desafio2/vm10/ether 10.png">

Vamos usar o burp para encodar em url, pois se der espaço no parâmetro recebido pelo cmd, ele quebra o código e não executa

O request com o payload em URL vai ficar assim

<img src="/img/desafio2/vm10/ether 11.png">

Antes de enviar, vamos deixar o netcat ouvindo na porta 443, que é a mesma que usamos para criar o payload.

E ao enviar o request com o payload, temos o reverse shell no netcat.

<img src="/img/desafio2/vm10/ether 12.png">

---

## Escalagem de Privilégio

Vamos importar a shell TTY para ter uma shell interativa e usar o sudo -l para ver quais comandos o usuario www-data pode executar como root.

<img src="/img/desafio2/vm10/ether 13.png">

Podemos ver que ele pode executar um script em python que ao executar o arquivo, vemos que ele da a opção para abrir um dos logs listados e ao escolher um dos dois, ele retorna o log.

<img src="/img/desafio2/vm10/ether 14.png">

Vamos criar outro payload usando msfvenom, idêntico ao anterior, mas como já estamos usando a porta 443, vamos criar usando a porta 80, porque vamos usar um operador lógico, para fazer abrir como root o payload malicioso.

Vamos iniciar o http server do python para usarmos o wget no alvo e fazer o download do arquivo malicioso.

<img src="/img/desafio2/vm10/ether 15.png">

Podemos ver que nosso arquivo foi enviado e já está no alvo.

<img src="/img/desafio2/vm10/ether 16.png">

Vamos dar o máximo de permissão para conseguir executar ele.

<img src="/img/desafio2/vm10/ether 17.png">

Ao executar o script como root, ele vai perguntar qual log queremos acessar e quando colocarmos o log, vamos usar o operador lógico AND para que ele execute também nosso arquivo malicioso como root, fazendo nosso reverse shell abrir no netcat com privilégio elevado.

<img src="/img/desafio2/vm10/ether 18.png">