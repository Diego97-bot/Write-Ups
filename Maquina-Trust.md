# Información general.
<b>Maquina:</b> [Trust](https://dockerlabs.es/).<br>
<b>Dificultad:</b> Facil. <br>
<b>Categoría:</b> Enumeración - Fuerza bruta - Elevación de previlegios.<br>
<b>Descripción:</b> Laboratorio para practicar enumeración web, fuerza bruta SSH con Hydra y escalada de privilegios abusando de sudoers.

<img width="991" height="675" alt="imagen" src="https://github.com/user-attachments/assets/f3f390f1-9c2c-44f0-b58f-8f79af255cc6" />


# Despliegue de la maquia.

Para desplegar esta maquina debemos extraer el contenido del zip descargado,<br>
una vez descomprimido el zip, deberemos abrir una terminal en la carpeta de la maquina.<br>
Ejecutamos el siguiente comando: <br>

```bash
auto_deploy.sh trust.tar
```

<img width="925" height="471" alt="imagen" src="https://github.com/user-attachments/assets/37a73f19-42ee-4d8c-9948-3b1a625753c9" />

Se ha desplegado nuestra maquina con la siguiente <code>IP <b>172.17.0.2</b></code>

# Fase de escaneo.

Como primera toma de contacto para cualquier maquina vamos a realizar ping:
```bash
ping -c 2 172.17.0.2
```
- -c: Establecemos el numero de veces que realiza el ping en este caso 2.<br>

<img width="543" height="201" alt="imagen" src="https://github.com/user-attachments/assets/bd43a7e8-45df-4e21-8bb6-3967f4edf363" />

Una vez comprabada la conexión procedemos a escanear la maquina, para ello usaremos la herramienta <b>Nmap</b><br>
para realizar el escaneo usarmeos el siguiente comando: <br> 
```bash
sudo nmap -p- -sS  -sC --min-rate 5000 -n -vvv -Pn 172.17.0.2 -oN escaneo 
```
- -p-: Escaneo de la totalidad de los puertos TCP (1-65535).

- -sS: Escaneo silencioso tipo TCP SYN (conexión semiabierta).

- -sC: Ejecución de scripts por defecto para fingerprinting y detección básica de vulnerabilidades.

- --min-rate 5000: Envío mínimo de 5,000 paquetes/seg para acelerar la ráfaga de escaneo.

- -n: Desactivación de la resolución DNS para optimizar tiempo.

- -Pn: Omisión de descubrimiento de host (asume que el objetivo está activo).

- -vvv: Salida detallada en tiempo real.

- -oN escaneo: Guardado de resultados en texto plano en el archivo escaneo.<br>

<img width="1031" height="87" alt="imagen" src="https://github.com/user-attachments/assets/745f2f34-a4a5-4d64-86eb-5551c73ae146" />

Una vez realizado el scaneo podemos comprobar que tenemos 2 puertos abiertos.<br>

<img width="1395" height="217" alt="imagen" src="https://github.com/user-attachments/assets/e4028179-bc8a-4021-8f32-60188565963b" />

| Puerto | Estado | Servicio |
|---|---|---|
| 22/tcp | abierto | SSH |
| 80/tcp | abierto | HTTP (Apache2, Debian) |

Accedemos al navegador y realizamos la busqueda de la <code>IP 172.17.0.2</code><br>

<img width="1918" height="732" alt="imagen" src="https://github.com/user-attachments/assets/ec4ac974-09fd-44ff-ab5e-be10adad5ebb" /><br>


Comprobamos que es una pagina donde hay un servicio de apache en funcionamiento

# Fase de enumeración
En esta fase vamos a obtener rutas,direcciones,directorios,archivos...etc que esten relacionadas con <code>IP 172.17.0.2</code><br>
Para ellos usaremos la herramienta <code><b>gobuster</b></code> con el siguiente comando:<br>

```bash
 gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -u http://172.17.0.2 -x txt,php,sql,py --exclude-length 10701 -t 50 
```
- gobuster dir: Activa el modo de escaneo de directorios y archivos web.

- -w /usr/.../directory-list-lowercase-2.3-small.txt: Define el diccionario de nombres de carpetas/archivos a probar (en este caso, la lista corta en minúsculas de Dirbuster).

- -u http://172.17.0.2: Especifica la URL objetivo a escanear.

- -x txt,php,sql,py: Busca archivos probando esas extensiones específicas al final de cada palabra del diccionario (ejemplo: admin.php, config.txt, db.sql).

- --exclude-length 10701: Ignora y oculta las respuestas de la página cuya longitud en bytes sea exactamente 10701 (muy útil para filtrar páginas falsas de error 404/200 personalizadas que ensucian el resultado).

- -t 50: Establece 50 hilos concurrentes para acelerar la velocidad del escaneo.

<img width="1352" height="404" alt="imagen" src="https://github.com/user-attachments/assets/f2348048-68df-4a11-8c22-66222cf19b58" />

Una vez terminado el escaneo podemos ver que ha econtrado un archivo llamado <code><b>secret.php</b></code><br>

Accedemos a la siguiente ruta a ver que nos muestra el fichero <code>http://172.17.0.2/secret.php</code><br>

<img width="1920" height="702" alt="imagen" src="https://github.com/user-attachments/assets/ffa1052c-cdfe-4775-9cb9-5b2a4d298381" />

Nos muestra un mensaje el cual nos da una pista, puede que el usuario que necesitamos sea <b>Mario</b><br>

# Ataque de fuerza bruta

Una vez obtenido el usuario necesitamos la pass del usuario para ello usaremos la "fuerza bruta" con la herramienta <code><b>Hydra</b></code> junto a un diccionario de posibles claves <code><b>Rockyou.txt</b></code><br> 

```bash
hydra -l mario -P rockyou.txt ssh://172.17.0.2 -v
```
 - hydra: Invoca la herramienta de pruebas de autenticación.

 - -l mario: Define un nombre de usuario único y fijo (mario) para la prueba. (Nota: Si fuera en mayúscula -L, indicaría un archivo con una lista de usuarios).

- -P rockyou.txt: Especifica el diccionario de contraseñas que se va a probar (en este caso, la famosa lista rockyou.txt).

- ssh://172.17.0.2: Establece el protocolo (SSH) y la dirección IP del objetivo.

- -v: Activa el modo detallado (verbose), mostrando en pantalla cada intento de combinación en tiempo real.

Una vez ejecutado el comando probara las posibles claves del usuario

<img width="794" height="449" alt="imagen" src="https://github.com/user-attachments/assets/d417b2f8-784d-4009-a8e6-2afebd065666" />

En este caso la contraseña que ha encontrado es <code><b>chocolate</b></code>

# Intrusión y elevación de permisos

Una vez obtenido el usuario y la contraseña podemos realizar la conexion mediante el protocolo <code>SSH</code><br>

```bash
ssh mario@172.17.0.2
```
Debemos responder <code><b>yes</b></code> a la siguiente pregunta <code>Are you sure you want to continue connecting (yes/no/[fingerprint])?</code><br>
introducimos la contraseña <code><b>chocolate</b></code><br>

<img width="849" height="313" alt="imagen" src="https://github.com/user-attachments/assets/be62f201-7b11-49c8-a59e-68e9ead6b9e3" />

Ya estamos dentro de la maquina de Mario, podemos comprobar que somos Mario con el comando:

```bash
whoami
```
Ahora comprobamos que puedo ejecutar como <code>root</code><br>
```bash
sudo -l
```
Vemos que podemos ejecutar el binario <code>vim</code> que es un editor de texto como <code>root</code><br>

<img width="956" height="143" alt="imagen" src="https://github.com/user-attachments/assets/00330344-2a7e-464d-971b-40689d69d664" />

para obtener los permisos de <code>root</code> ejecutamos vim como modo <code>root</code><br>

```bash
sudo -u root /usr/bin/vim
```
- -u: Usuario<br>

<img width="1914" height="669" alt="imagen" src="https://github.com/user-attachments/assets/b68cf1c0-7584-4f6c-8817-cb86df49f5d8" />

Una vez dentro del editor escribimos los siguiente<br>
```bash
:!/bin/bash
```
Esto nos permite salir de vim pero manteniendo los privilegios con los que lo hemos ejecutado es decir con permisos de root

<img width="1304" height="391" alt="imagen" src="https://github.com/user-attachments/assets/bd5c795f-eebb-4a6f-abc0-d11244faadec" />

Una vez ejecutado volvemos a la terminal y vemos que ha cambiado, ejecutamos <code>whoami</code> y podemos ver que somos el usuario root.

<img width="455" height="91" alt="imagen" src="https://github.com/user-attachments/assets/c6fede60-0bef-4fca-97fc-b52aa60dc5af" />






























