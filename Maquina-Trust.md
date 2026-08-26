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
En esta fase vamos a obtener rutas,direcciones y/o directorios que esten relacionadas con <code>IP 172.17.0.2</code><br>
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




















