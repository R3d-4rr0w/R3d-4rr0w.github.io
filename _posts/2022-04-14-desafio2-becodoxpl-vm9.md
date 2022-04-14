---
title: Desafio 2 - VM 09
author: R3d-4rr0w
date: 2022-04-14 06:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, Drupal, Backdoor]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM9"
---

# VM 9 - DC

---

## [VM 9 - DC](https://www.vulnhub.com/entry/dc-1-1,292/#download)

---

## Reconhecimento

NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -v 192.168.56.12
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-12 08:05 EDT
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
80/tcp  open  http    Apache httpd 2.2.22 ((Debian))
111/tcp open  rpcbind 2-4 (RPC #100000)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Acessando a porta 80 do serviço web no browser, vemos a página inicial do Drupal.

<img src="/img/desafio2/vm9/dc.png">)

Olhando o código fonte, conseguimos ver  versão do drupal que está sendo usada.

<img src="/img/desafio2/vm9/dc 1.png">

Ao procurar por Drupal no searchsploit, vemos que tem um exploit que explora um SQL Injection.

<img src="/img/desafio2/vm9/dc 2.png">

---

## Exploração Drupalgeddon

>**💡 [Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)](https://www.exploit-db.com/exploits/34984)**


O exploit insere um novo usuário no banco de dados do Drupal, permitindo nosso acesso a aplicação.

Como o código foi escrito na versão 2.x do Python, teremos que executar com a mesma versão, pois tem algumas diferenças de sintaxe para o Python 3.x

<img src="/img/desafio2/vm9/dc.png 3">

Agora vamos fazer login com o usuário que criamos usando o exploit.

<img src="/img/desafio2/vm9/dc 4.png">

Conseguimos fazer o login na aplicação.

<img src="/img/desafio2/vm9/dc 5.png">

---

## Backdoor em PHP no Tema

Podemos tentar inserir um backdoor no template principal de algum tema, para termos uma reverse shell.

Vamos em appearance para ver os temas que estão instalados na plataforma.

<img src="/img/desafio2/vm9/dc 6.png">

Vamos baixar um tema novo para inserir o backdoor na [plataforma](https://www.drupal.org/project/project_theme).

Dentro da pasta do tema baixado, temos os arquivos que fazem parte dele.

O template.php é executado pela plataforma para carregar o novo tema, então vamos inserir um backdoor no código dele, usando a função [exec()](https://www.php.net/manual/en/function.exec.php) do php.

<img src="/img/desafio2/vm9/dc 7.png">

O código do template.php com o backdoor.

<img src="/img/desafio2/vm9/dc 8.png">

Para carregar o tema, temos que compactar a pasta e assim conseguimos instalar

<img src="/img/desafio2/vm9/dc 9.png">

<img src="/img/desafio2/vm9/dc 10.png">

Podemos ver que o tema com nosso backdoor foi instalado.

<img src="/img/desafio2/vm9/dc 11.png">

Vamos deixar o netcat esperando a conexão e selecionar o tema.

<img src="/img/desafio2/vm9/dc 12.png">

Assim conseguimos  a shell.

<img src="/img/desafio2/vm9/dc 13.png">

Agora podemos carregar a shell TTY usando o python.

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

---

## Escalagem de Privilégio

Ao usar o comando sudo -l, retorna que o comando não existe.

Vamos usar o find para encontrar arquivos que possamos executar com permissão de root.

```bash
find / -perm -u=s -type f 2>/dev/null
```

O usuário tem permissão para executar esses arquivos como root.

<img src="/img/desafio2/vm9/dc 14.png">

Podemos ver que pode usar o /usr/bin/find, dentro desse comando existe uma funcão exec, que permite executar comandos do sistema.

Vamos chamar o /bin/sh com a função -exec do find, assim ele vai executar o find como o root e consequentemente chamar o dash como root.

```bash
/usr/bin/find -exec "/bin/sh" \;
```

Assim conseguimos escalar o privilégio.

<img src="/img/desafio2/vm9/dc 15.png">

Agora podemos acessar o diretorio /root e ler a flag que está la.

<img src="/img/desafio2/vm9/dc 16.png">