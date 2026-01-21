

# Enumeración 

Vamos a empezar con un sencillo escaneo nmap, para enumerar primeramente puertos abiertos que corren en la máquina.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN info_ports
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-21 20:20 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
12345/tcp open  netbus
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.15 seconds

  </code>
</pre>

-p- : 
--open:



Bien vemos los puertos 22 donde corre ssh y el 12345 donde corre netbus. Vamos a realizar un segundo escaneo para enumerar versiones y servicios que corren en dichos puertos.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p22,12345 -sCV 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-21 20:23 CET
Nmap scan report for buffered.dl (172.17.0.2)
Host is up (0.000043s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 4f:3f:8c:fb:88:da:ea:37:d6:9f:c3:bd:f4:8e:18:1b (ECDSA)
|_  256 2e:a1:36:ff:8b:bb:0d:b3:c8:cb:4a:81:cb:37:77:31 (ED25519)
12345/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.11.2)
|_http-server-header: Werkzeug/2.2.2 Python/3.11.2
|_http-title: Site doesn't have a title (application/json).
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.85 seconds

  </code>
</pre>

Vemos que tenemos Openssh 9.2p1 por ahí no podemos hacer mucho vamos a acceder desde el navegador al puerto 12345 donde corre Werkzeug


