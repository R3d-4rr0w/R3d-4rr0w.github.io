---
title: Desafio 2 - VM 04
author: R3d-4rr0w
date: 2022-04-08 17:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, FTP, Mod_Copy]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM4"
---

# VM 4 - JOY

---

## [VM 4 - JOY](https://www.vulnhub.com/entry/digitalworldlocal-joy,298/)

---

## Reconhecimento:

NMAP:

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -p- -Pn -v 192.168.56.7
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 14:39 EDT
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.2.10
22/tcp  open  ssh         Dropbear sshd 0.34 (protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
80/tcp  open  http        Apache httpd 2.4.25
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
465/tcp open  smtp        Postfix smtpd
587/tcp open  smtp        Postfix smtpd
993/tcp open  ssl/imap    Dovecot imapd
995/tcp open  ssl/pop3    Dovecot pop3d
Service Info: Hosts: The,  JOY.localdomain, 127.0.1.1, JOY; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vamos ver se o FTP aceita login como anonymous:

Para isso vamos usar o módulo auxiliary/scanner/ftp/anonymous do metasploit, para automatizar

Após inserir as opções que são necessárias, vamos rodar o scan.

Podemos ver que o alvo aceita conexão com o usuário anonymous no serviço de FTP.

<img src="/img/desafio2/vm4/mod_copy.png">

E assim conseguimos o acesso ao FTP.

<img src="/img/desafio2/vm4/mod_copy1.png">

Dentro da pasta raiz do ftp, temos os diretórios:

<img src="/img/desafio2/vm4/mod_copy2.png">

O diretório download está vazio.

Dentro do upload temos:

<img src="/img/desafio2/vm4/mod_copy3.png">

Logo no primeiro arquivo chamado “directory”, temos como se fosse o log do comando ls -lha dentro da pasta home do usuário Patrick:

E um arquivo chamado version_control que chamou atenção

<img src="/img/desafio2/vm4/mod_copy4.png">

---

## Exploração mod_copy

Vamos acessar o FTP  através do netcat e usar o mod_copy para copiar o arquivo version_control do diretório do usuário Patrick para o diretório do FTP.

<img src="/img/desafio2/vm4/mod_copy5.png">

Vamos acessar novamente o FTP e fazer o download do arquivo que copiamos

<img src="/img/desafio2/vm4/mod_copy6.png">

Ao acessar o arquivo, podemos ver uma informação interessante, dizendo que o diretório da aplicação web foi alterado e também a versão real do ProFTPd, que no scan do nmap estava diferente.

<img src="/img/desafio2/vm4/mod_copy7.png">

Dentro do metasploit, existe um exploit para essa versão do ProFTPd que explora o  mod_copy e nos garante um RCE.

<img src="/img/desafio2/vm4/mod_copy8.png">

Vamos configurar o exploit, lembrando que o diretório da aplicação web foi alterado e vamos utilizar o payload reverse_python, para conseguir a conexão reversa.

<img src="/img/desafio2/vm4/mod_copy9.png">

Após executar o exploit, temos a conexão reversa como usuário www-data:

<img src="/img/desafio2/vm4/mod_copy10.png">

Dentro do diretório que temos acesso, existe uma pasta chamada ossec e dentro dela temos alguns arquivos e um chamou atenção.

<img src="/img/desafio2/vm4/mod_copy11.png">

De dentro dele temos as credencias do usuário Patrick:

<img src="/img/desafio2/vm4/mod_copy12.png">

Estamos com a reverse_shell que conseguimos através do metasploit, mas ela é instavel.

Vamos alterar para o bash através do modulo pty do python 3 usando o comando 

python3 -c ‘import pty;pty.spawn(”/bin/bash”)’

<img src="/img/desafio2/vm4/mod_copy13.png">

Agora vamos alterar o usuário para o patrick, que não tem privilégios de root.

<img src="/img/desafio2/vm4/mod_copy14.png">

---

## Escalagem de Privilégio

Vamos ver quais comandos ele pode executar como root.

<img src="/img/desafio2/vm4/mod_copy15.png">

O usuário pode executar o arquivo test, dentro da pasta script que está no diretório raiz dele, mas ao tentar acessar o arquivo, vemos que não temos acesso para isso.

Podemos explorar o mod_copy para substituir o arquivo test.

Vamos criar um arquivo utilizando a linguagem  AWK para chamar o /bin/bash usando o módulo de sistema e nomear como test, que é o arquivo que o usuário patrick consegue executar como root

<img src="/img/desafio2/vm4/mod_copy16.png">

E vamos usar o FTP para fazer o upload desse arquivo

<img src="/img/desafio2/vm4/mod_copy17.png">

Agora vamos logar no FTP através do netcat e usar o mod_copy para copiar o arquivo test para a pasta script.

<img src="/img/desafio2/vm4/mod_copy18.png">

Agora vamos executar o arquivo através da shell do metasploit para escalar privilégio.

<img src="/img/desafio2/vm4/mod_copy19.png">)