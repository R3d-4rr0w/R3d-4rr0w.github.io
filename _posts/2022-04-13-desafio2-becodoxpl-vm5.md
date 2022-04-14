---
title: Desafio 2 - VM 05
author: R3d-4rr0w
date: 2022-04-13 17:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, Brute Force, Backdoor]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM5"
---

# VM 5 - Violator

---

## [VM 5 - Violator](https://www.vulnhub.com/entry/violator-1,153/)

---

## Reconhecimento

NMAP:

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn 192.168.56.8 -v
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-11 08:42 EDT
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5rc3
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
Service Info: OS: Unix
```


Temos um serviço FTP rodando na porta 21, vamos usar o script do metasploit para verificar se conseguimos fazer login com o usuário anonymous.

<img src="/img/desafio2/vm5/aviator.png">

Como podemos ver, não aceita anonymous login.

A versão do ProFTPD rodando é a 1.3.5, nela existe uma falha no módulo [mod_copy](http://www.proftpd.org/docs/contrib/mod_copy.html), que permite qualquer usuário copiar arquivos sem precisar de permissão.


💡 [ProFTPD Vulnerability Lets Users Copy Files Without Permission] https://www.bleepingcomputer.com/news/security/proftpd-vulnerability-lets-users-copy-files-without-permission/)


Vamos acessar com o netcat e copiar o arquivo /etc/passwd para ver quais usuários existem.

<img src="/img/desafio2/vm5/aviator 1.png">

Agora podemos acessar o arquivo via browser, através do serviço web que está rodando na máquina.

<img src="/img/desafio2/vm5/aviator 2.png">

Já temos os usuários existentes no servidor.

Ao abrir a página inicial do serviço web, vemos que no rodapé existe uma url.

<img src="/img/desafio2/vm5/aviator 3.png">

A url leva para a página da WikiPedia do [Album Violator](https://en.wikipedia.org/wiki/Violator_(album)) do grupo [Depeche Mode](https://en.wikipedia.org/wiki/Depeche_Mode).

Vemos que os usuários que achamos no passwd, são os nomes dos integrantes do grupo.

<img src="/img/desafio2/vm5/aviator 4.png">

Vemos também as músicas desse álbum.

<img src="/img/desafio2/vm5/aviator 5.png">

Vamos usar essas informações para criar uma wordlist personalizada e tentar fazer um bruteforce no serviço FTP, para conseguir a senha dos usuários que encontramos.

Para isso vamos usar o hydra.

<img src="/img/desafio2/vm5/aviator 6.png">

---

## Exploração mod_copy

Conseguimos as credenciais dos usuários do servidor.

Agora  vamos usar o metasploit para explorar a falha do mod_copy de forma automatizada e conseguir uma shell reversa.

Configurações do exploit.

<img src="/img/desafio2/vm5/aviator 7.png">

Ao executar o exploit, conseguimos a shell reversa.

<img src="/img/desafio2/vm5/aviator 8.png">

Vamos criar nossa shell interativa com o python pty e mudar para algum usuário que listamos no bruteforce.

<img src="/img/desafio2/vm5/aviator 9.png">

---

## Escalagem de Privilégio

O usuário não tem acesso root, vamos usar o comando sudo -l para ver os comandos que ele pode executar como root. 

<img src="/img/desafio2/vm5/aviator 10.png">

O usuário pode iniciar um ProFTPD, vamos até o diretório desse serviço e ver a versão.

<img src="/img/desafio2/vm5/aviator 11.png">

O ProFTPD 1.3.3c, é uma versão não oficial que contém um backdoor no seu código.

<aside>
💡 [P**roFTPD-1.3.3c-backdoor**](https://www.aldeid.com/wiki/Exploits/proftpd-1.3.3c-backdoor)

</aside>

Agora vamos ver arquivo de configuração do serviço.

<img src="/img/desafio2/vm5/aviator 12.png">

Dentro do arquivo de configuração, fala que o serviço será executado em [localhost](http://localhost) na porta 2121

<img src="/img/desafio2/vm5/aviator 13.png">

Vamos ver se existe algum serviço rodando no [localhost](http://localhost) com o netstat usando as flags:

> -a (all) - Mostra todas as conexões
> 

> -n (numeric) - Mostra o IP, sem resolver o nome do DNS
> 

> -t (tcp) - Mostra apenas os protocolos TCP.
> 

> -p (program) - Mostra o ID do processo/programa que está usando a conexão.
> 

<img src="/img/desafio2/vm5/aviator 14.png">

Não existe nenhum serviço rodando no [localhost](http://localhost) do servidor.

Após rodar o script que o usuário tem permissão e ao verificar o netstat novamente, podemos ver que o serviço foi criado.

<img src="/img/desafio2/vm5/aviator 15.png">

Ao tentar acessar a porta 2121 com o netcat, retorna um Connection Refused, pois ele tem acesso apenas pelo [localhost](http://localhost) do servidor alvo.

<img src="/img/desafio2/vm5/aviator 16.png">

Podemos fazer o pivot e ter acesso ao localhost.

Dentro do meterpreter, existe uma opção de port forwarding, para redirecionamento de portas.

Vamos jogar essa sessão para background e abrir uma sessão com meterpreter.

Para isso vamos usar o módulo shell_to_meterpreter do metasploit e fazer o upgrade da sessão que já temos para o meterpreter.

<img src="/img/desafio2/vm5/aviator 17.png">

Dentro do help do meterpreter, podemos ver o portfwd, que vamos usar para fazer o port forwarding.

<aside>
💡 [Meterpreter -  Port Forwarding](https://www.offensive-security.com/metasploit-unleashed/portfwd/)

</aside>

<img src="/img/desafio2/vm5/aviator 18.png">

Vamos adicionar o portforwarding, para que o que for executado no nosso [localhost](http://localhost) na porta 2121, seja executado no servidor alvo também.

<img src="/img/desafio2/vm5/aviator 19.png">

Agora podemos usar o backdoor que existe na versão 1.3.3c do ProFTPD.

<img src="/img/desafio2/vm5/aviator 20.png">

Após executar o exploit, recebemos a conexão reversa do backdoor existente e temos acesso root no alvo.

<img src="/img/desafio2/vm5/aviator 21.png">