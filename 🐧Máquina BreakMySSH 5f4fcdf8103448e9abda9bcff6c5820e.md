# 🐧Máquina BreakMySSH

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/6932460f-f0ab-45c8-b986-262628eeea8a.png)

**Índice**

# Requisitos previos.

---

### 💿 Instalar Docker.

Para utilizar las máquinas de DockerLabs, hay que instalar Docker en el sistema operativo.

```bash
sudo apt install docker.io
```

![instalarDocker.jpeg](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/instalarDocker.jpeg)

### 🖥️ Desplegar máquinas vulnerables.

Al descomprimir el archivo.tar obtenemos dos archivos: 

- auto_deploy.sh
- maquinaVulnerable.tar

```bash
sudo bash auto_deploy.sh maquinaVulnerable.tar
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/4ea1c794-cb4f-442d-bdee-49902a27a101.png)

### 🚮 Eliminar máquinas vulnerables.

Al finalizar, eliminamos el laboratorio con la combinación de teclas: “Ctrl + c”

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/326748d7-855e-4464-b984-9c24497538ac.png)

# Reconocimiento

---

### 🕵‍♂️Verifico accesibilidad con el comando ping.

El comando “**ping -c 1**” enviando un solo paquete al host de destino.

```bash
ping -c 1 172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/6d779b3d-db19-48d4-bbfb-f47f47738759.png)

La **ttl = 64** nos indica que se trata de una máquina Linux.

### 🕵‍♂️Escaneo de puertos.

Con la herramienta **nmap realizo un escaneo:**

```bash
sudo nmap -p- --open -sS -sC -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN scanBreakMySSH
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

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/3938a285-0966-4d75-bc70-610e54738f49.png)

# Explotación

---

### 🕵🏼‍♂️ Puerto 22, protocolo SSH

Busco vulnerabilidades de versión OpenSSH 7.7

```bash
metasploit openssh 7.7
```

La versión OpenSSH 7.7 es vulnerable a la enumeración de usuarios. Sin embargo, no he logrado realizar ninguna enumeración.

### 🕵🏼‍♂️ Ataque de fuerza bruta con Hydra.

Supongo que usuario **root** puede ser un usuario válido del sistema. Intento un ataque de fuerza bruta al usuario **root** con **hydra**.

```bash
sudo hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2

```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/96b8ab1f-6445-49d2-8215-ede0c43f0efd.png)

El usuario es válido y encuentro la contraseña **estrella.**

# Intrusión

---

### 🔓Accediendo al sistema.

Al obtener usuario y contraseña, ingreso por el puerto 22, protocolo SSH.

```bash
ssh root@172.17.0.2
```

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/b6d45e11-407a-46d3-8f9c-73eedf7a4843.png)

Con el comando ***whoami*** verifico quién soy para ver si tengo acceso o no, como usuario root.

![Untitled](%F0%9F%90%A7Ma%CC%81quina%20BreakMySSH%205f4fcdf8103448e9abda9bcff6c5820e/49f04a1a-0b79-4586-b99d-146c28212da7.png)

---