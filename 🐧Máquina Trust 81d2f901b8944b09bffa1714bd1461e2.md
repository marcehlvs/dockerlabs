# 🐧Máquina Trust

![trust.jpeg](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/trust.jpeg)

**Índice**

# Requisitos previos.

---

### 💿 Instalar Docker.

Para utilizar las máquinas de DockerLabs, hay que instalar Docker en el sistema operativo.

Agrega más detalles aquí…

```bash
sudo apt install docker.io
```

![instalarDocker.jpeg](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/instalarDocker.jpeg)

### 🖥️ Desplegar máquinas vulnerables.

Al descomprimir el archivo.tar obtenemos dos archivos: 

- auto_deploy.sh
- maquinaVulnerable.tar

```bash
sudo bash auto_deploy.sh maquinaVulnerable.tar
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/4ea1c794-cb4f-442d-bdee-49902a27a101.png)

### 🚮 Eliminar máquinas vulnerables.

Al finalizar, eliminamos el laboratorio con la combinación de teclas: “Ctrl + c”

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/326748d7-855e-4464-b984-9c24497538ac.png)

# Reconocimiento

---

### 🕵‍♂️Verifico accesibilidad con el comando ping.

El comando “**ping -c 1**” enviando un solo paquete al host de destino.

```bash
ping -c 1 172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/6d779b3d-db19-48d4-bbfb-f47f47738759.png)

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

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/e9536350-e7b6-41d3-95f4-4d5155dff4e7.png)

# Explotación

---

### 🕵🏼‍♂️ Puerto 80

Ingresamos y por defecto corre una plantilla de Apache2. Busco información relevante en el código HTML con la combinación de teclas “ctrl + u” pero no encuentro nada importante.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/Untitled.png)

El siguiente paso es encontrar subdirectorios realizando ***fuzzing***  web con la herramienta ***gobuster.***

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh,html
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/8bc67b0f-ca6c-4c6b-a872-f993779d8d09.png)

Encuentro el subdirectorio ***/secret.php.*** Me dirijo ahí:

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/Untitled%201.png)

Obtengo información sobre un “posible” usuario: Mario.

### 🕵🏼‍♂️ Puerto 22. Ataque de fuerza bruta.

- Al encontrar el usuario Mario, resta realizar un ataque de fuerza bruta al protocolo SSH con la herramienta ***hydra***.
    
    ```bash
    sudo hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
    ```
    
    ![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/13b58196-1851-47a5-899c-255901c5d04b.png)
    
    Encuentro la contraseña para el usuario mario.
    

# Intrusión

---

### 🔓Accediendo al sistema.

Al obtener usuario y contraseña, ingreso por el puerto 22, protocolo SSH.

```bash
ssh mario@172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/ac13200f-10da-43d3-a543-c1fa9dd1cea2.png)

Con el comando ***whoami*** verifico quién soy para ver si tengo acceso o no, como usuario root.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/fe5b1c5a-a257-42e5-a26d-c8fbaadf828f.png)

### 🔓Escalada de privilegios.

Una vez hayamos obtenido acceso remoto al servidor, es necesario proceder con la escalada de privilegios, para poder convertirnos en el usuario root del sistema y así contar con todo tipo de permisos sobre el mismo.

Con el comando ***sudo -l*** voy a comprobar posibles formas de escalar privilegios.

```bash
mario@6780e3f83dab:~$ sudo -l
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/b40aafb7-66f1-4e75-8421-c3b7c66dc2c3.png)

En [https://gtfobins.github.io/](https://gtfobins.github.io/) puedo ver instrucciones para poder escalar privilegios utilizando binarios. Teniendo en cuenta que el usuario **mario** puede ejecutar el binario ***vim***. Lo busco en el apartado de **sudo.**

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/5ae74dea-3229-4d95-a5e8-5e9dee857484.png)

```bash
sudo /usr/bin/vim -c ':!/bin/sh'
```

Al ejecutar este comando ya dejo de ser el usuario **mario** para pasar a ser el usuario **root.**

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20Trust%2081d2f901b8944b09bffa1714bd1461e2/3e6f9ac3-0c27-4dad-b637-532329ffa1ac.png)

---