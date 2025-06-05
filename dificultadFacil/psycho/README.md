# 🧠 psycho

![IMAGEN_MAQUINA_PORTADA](./imagenes/portadaMaquina.png)
> 💡 NOTA:  Al final suelo dejar una lista con los comandos, herramientas y servicios que he usado durante el laboratorio, y los explico un poco por si alguien quiere repasarlos o usarlos como referencia.
---

## 🔎 Fase 1 - Reconocimiento

Para comenzar, lanzamos un escaneo de puertos con **nmap** para ver qué servicios están expuestos por la máquina.

![IMAGEN NMAP](./imagenes/nmap.png)

Se detectan los puertos **22 (SSH)** y **80 (HTTP)**.  
Sin credenciales no podemos hacer mucho con SSH, así que pasamos al puerto 80.

Lo primero que solemos hacer es un **fuzzing web** para encontrar directorios ocultos.

![IMAGEN GOBUSTER](./imagenes/fuzzingWEB.png)

No aparece gran cosa, así que entramos directamente a la página principal: `index.php`.

![IMAGEN PAGINA PRINCIPAL](./imagenes/imagenPrincipal.png)

Como es habitual, miramos el código fuente de la web (Ctrl + U) para ver si hay comentarios sospechosos o algún dato útil.

![IMAGEN CODIGO FUENTE](./imagenes/codigo.png)

No encontramos nada especial, pero hay un **error visible** al final de la página, lo que sugiere que podría estar esperando un parámetro específico. Esto nos hace pensar en una posible **vulnerabilidad LFI** (Local File Inclusion), que básicamente permite leer archivos del sistema si un parámetro no está bien filtrado.

Probamos con un **fuzzing a parámetros** usando ffuf para ver si alguno activa esa vulnerabilidad.

![IMAGEN FFUF](./imagenes/ffuf.png)

¡Bingo! Aparece un parámetro llamado `secret`.

---

## 💥 Fase 2 - Explotación

Con esto, accedemos al navegador y probamos añadiendo `?secret=` al final de la URL. Por ejemplo:
``
http://IP/index.php?secret=/etc/passwd
``

![IMAGEN /ETC/PASSWD](./imagenes/LFI_funciona.png)

En el archivo `/etc/passwd` vemos que hay dos usuarios interesantes: **luisillo** y **vaxei**.

Volviendo al puerto 22, podríamos intentar un ataque de fuerza bruta... pero mejor buscamos algo más limpio. Vamos a ver si podemos acceder a la clave privada **id_rsa** de alguno de estos usuarios.

Probaremos con:
``
http://IP/index.php?secret=/home/vaxei/.ssh/id_rsa
``

![IMAGEN ID_RSA](./imagenes/id_rsa_vaxei.png)

Conseguimos la clave privada de **vaxei**, la copiamos, la guardamos en un archivo, le damos permisos (`chmod 600`) y probamos acceder por **SSH**.

![IMAGEN SSH VAXEI](./imagenes/ssh1.png)

---

## 🧗 Fase 3 - Escalada de privilegios

Ya dentro de la máquina como usuario **vaxei**, toca escalar privilegios para tomar el control total.  
Lo primero es ver qué binarios podemos ejecutar con `sudo -l`.

Este comando nos muestra qué acciones podemos hacer con privilegios elevados.

![IMAGEN SUDO -L VAXEI](./imagenes/sudo-l_vaxei.png)

Además, ejecutamos una serie de comandos para que la terminal se vea mejor (export de TERM y SHELL, etc).

Según lo que nos permite `sudo`, podemos ejecutar **perl** como el usuario **luisillo**.  
Usamos [GTFOBins](https://gtfobins.github.io/) (página muy útil que muestra formas de escalar privilegios usando binarios comunes) para ver cómo explotar `perl`.

![IMAGEN PERL ESCALADA](./imagenes/login_luisillo.png)

Una vez somos **luisillo**, hacemos otra vez `sudo -l`.

![IMAGEN SUDO -L LUISILLO](./imagenes/sudo-l_luisillo.png)

Nos dice que podemos ejecutar un script Python como **cualquier usuario**, incluso **root**.

Primero, revisamos los permisos del script original en `/opt`, ya que, si es posible modificarlo o eliminarlo para crear uno nuevo, podremos ahorrar trabajo.

![IMAGEN PERMISOS SCRIPT.PY](./imagenes/permisosScript.png)

Vemos que no podemos modificar el `.py`, por lo que vamos a probar con los permisos del directorio padre. 

![IMAGEN PERMISOS OPT](./imagenes/permisos-directorio.png)

Vemos que **el directorio `/opt` es escribible**. Eso nos permite **borrar el script** y crear uno con el mismo nombre que nos dé una shell como root.

Creamos nuestro propio `paw.py` con el siguiente contenido:

```python
import os
os.system("/bin/bash")
```
![NUEVO SCRIPT](./imagenes/nuevoScript.png)

Ejecutamos el script usando sudo con el usuario root, y obtendremos acceso root completo.

![NUEVO SCRIPT EJECUCCION](./imagenes/ejecuccion_script.png)

Máquina comprometida 🔓

---

## 🏁 Conclusión

En este reto hemos visto:
  
  - Cómo detectar y explotar una vulnerabilidad LFI.

  - Cómo escalar privilegios desde un usuario normal usando sudo + binarios vulnerables (perl).

  - Cómo hacer movimiento lateral a otro usuario (vaxei → luisillo).

  - Y cómo explotar permisos de escritura en un directorio para ejecutar un script como root.

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

## 🛠️ Herramientas, comandos y servicios usados

### 🔍 Nmap
```
nmap -p- --open -A -sS -Pn -n <IP>
Escanea puertos y detecta servicios.

    -p-: escanea todos los puertos.
    
    --open: muestra solo los puertos abiertos.
    
    -A: detección de sistema y versión.
    
    -sS: escaneo SYN, más sigiloso.
    
    -Pn: no hace ping previo.
    
    -n: no resuelve nombres DNS. 
```

### 🪓 Gobuster
```
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,py,html,txt,js
Hace fuzzing para encontrar directorios o archivos ocultos.
    -w: diccionario.
    -x: extensiones que queremos probar.
```
###🏹 Ffuf
```
ffuf -c -t 200 -fc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://<IP>/index.php?FUZZ=/etc/passwd"
Fuzzing para probar parámetros o rutas.
    -fc 404: ignora errores 404.
    
    -t 200: usa 200 hilos (va más rápido).
    
    -u: URL con FUZZ donde se probarán los valores.
```

