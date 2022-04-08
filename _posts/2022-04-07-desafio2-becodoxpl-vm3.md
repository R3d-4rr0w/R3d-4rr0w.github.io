---
title: Desafio 2 - VM 03
author: R3d-4rr0w
date: 2022-04-08 16:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, Drupal, Overlayfs]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM3"
---

# VM 3 - DROOPY: V0.2

---

## [VM 3 - ****DROOPY: V0.2****](https://www.vulnhub.com/entry/droopy-v02,143/)

---

## Reconhecimento:

NMAP :

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -p- -Pn 192.168.56.6 -v
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 12:35 EDT
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
```

Acessando a porta 80 no browser, temos uma página web que podemos ver que está usando o Drupal.

<img src="/img/desafio2/vm3/drupal0.png">

Acessando o código fonte dessa página, vemos que está usando a versão 7 do Drupal.

<img src="/img/desafio2/vm3/drupal1.png">

Após uma pesquisa no searchsploit, podemos que nessa versão do Drupal, existe uma vulnerabilidade conhecida como Drupalgeddon, que resulta em um RCE através de um SQL Injection. 

<img src="/img/desafio2/vm3/drupal2.png">

Vamos procurar o exploit dentro do metasploi para automatizar a exploração:

<img src="/img/desafio2/vm3/drupal3.png">

---

## Exploração via Metasploit

Agora vamos configurar o exploit, inserindo as informações necessárias.

<img src="/img/desafio2/vm3/drupal4.png">

Após rodar o exploit, temos uma sessão do meterpreter, mas não temos privilégios de root.

Vendo as informações do sistema, vemos que a versão do Kernel Linux é a 3.13.0, que é uma versão vulnerável a uma falha no overlayfs que permite a escalagem de privilégios localmente.

<img src="/img/desafio2/vm3/drupal5.png">

---

## Escalagem de Privilégios

Para isso, vamos jogar a sessão do meterpreter para o background e procurar pelo overlayfs no searchsploit.

<img src="/img/desafio2/vm3/drupal6.png">

Vamos copiar o arquivo do exploit e compilar usando o gcc. 

<img src="/img/desafio2/vm3/drupal7.png">)

Após compilado, precisamos enviar para o alvo, pois precisar ser executado localmente por ele.

Podemos fazer o upload via meterpreter no diretorio /tmp, pois temos acesso para isso.

<img src="/img/desafio2/vm3/drupal8.png">

Ou podemos subir um http server com python3 para hospedar o arquivo.

<img src="/img/desafio2/vm3/drupal9.png">

E usar o wget no alvo para fazer o download do exploit.

<img src="/img/desafio2/vm3/drupal10.png">

Agora vamos dar permissão de execução para o arquivo que enviamos.

Após a execução do exploit, conseguimos escalar o privilégio e nos tornar root.

<img src="/img/desafio2/vm3/drupal11.png">