# 🧠 psycho

![IMAGEN_MAQUINA_PORTADA](imagen1.png)

---

## 🔎 Fase 1 - Reconocimiento

Para comenzar, lanzamos un escaneo de puertos con **nmap** para ver qué servicios están expuestos por la máquina.

![IMAGEN NMAP](imagen2.png)

Se detectan los puertos **22 (SSH)** y **80 (HTTP)**.  
Sin credenciales no podemos hacer mucho con SSH, así que pasamos al puerto 80.

Lo primero que solemos hacer es un **fuzzing web** para encontrar directorios ocultos.

![IMAGEN GOBUSTER](imagen3.png)

No aparece gran cosa, así que entramos directamente a la página principal: `index.php`.

![IMAGEN PAGINA PRINCIPAL](imagen4.png)

Como es habitual, miramos el código fuente de la web (Ctrl + U) para ver si hay comentarios sospechosos o algún dato útil.

![IMAGEN CODIGO FUENTE](imagen5.png)

No encontramos nada especial, pero hay un **error visible** al final de la página, lo que sugiere que podría estar esperando un parámetro específico. Esto nos hace pensar en una posible **vulnerabilidad LFI** (Local File Inclusion), que básicamente permite leer archivos del sistema si un parámetro no está bien filtrado.

Probamos con un **fuzzing a parámetros** usando ffuf para ver si alguno activa esa vulnerabilidad.

![IMAGEN FFUF](imagen6.png)

¡Bingo! Aparece un parámetro llamado `secret`.

---

## 💥 Fase 2 - Explotación

Con esto, accedemos al navegador y probamos añadiendo `?secret=` al final de la URL. Por ejemplo:

http://IP/index.php?secret=/etc/passwd


![IMAGEN /ETC/PASSWD](imagen7.png)

En el archivo `/etc/passwd` vemos que hay dos usuarios interesantes: **luisillo** y **vaxei**.

Volviendo al puerto 22, podríamos intentar un ataque de fuerza bruta... pero mejor buscamos algo más limpio. Vamos a ver si podemos acceder a la clave privada **id_rsa** de alguno de estos usuarios.

Probaremos con:

http://IP/index.php?secret=/home/vaxei/.ssh/id_rsa


![IMAGEN ID_RSA](imagen8.png)

Conseguimos la clave privada de **vaxei**, la copiamos, la guardamos en un archivo, le damos permisos (`chmod 600`) y probamos acceder por **SSH**.

![IMAGEN SSH VAXEI](imagen9.png)

---

## 🧗 Fase 3 - Escalada de privilegios

Ya dentro de la máquina como usuario **vaxei**, toca escalar privilegios para tomar el control total.  
Lo primero es ver qué binarios podemos ejecutar con `sudo -l`.

Este comando nos muestra qué acciones podemos hacer con privilegios elevados.

![IMAGEN SUDO -L VAXEI](imagen10.png)

Además, ejecutamos una serie de comandos para que la terminal se vea mejor (export de TERM y SHELL, etc).

Según lo que nos permite `sudo`, podemos ejecutar **perl** como el usuario **luisillo**.  
Usamos [GTFOBins](https://gtfobins.github.io/) (página muy útil que muestra formas de escalar privilegios usando binarios comunes) para ver cómo explotar `perl`.

![IMAGEN PERL ESCALADA](imagen11.png)

Una vez somos **luisillo**, hacemos otra vez `sudo -l`.

![IMAGEN SUDO -L LUISILLO](imagen12.png)

Nos dice que podemos ejecutar un script Python como **cualquier usuario**, incluso **root**.

Primero, revisamos los permisos del script original en `/opt`.

![IMAGEN PERMISOS SCRIPT.PY](imagen13.png)

Aunque no podemos modificar el `.py`, vemos que **el directorio `/opt` es escribible**.

![IMAGEN PERMISOS OPT](imagen14.png)

Eso nos permite **borrar el script** y crear uno con el mismo nombre que nos dé una shell como root.

Creamos nuestro propio `paw.py` con el siguiente contenido:

```python
import os
os.system("/bin/bash")

Ejecutamos el script usando sudo con el usuario root, y obtenemos acceso root completo.

🏁 Conclusión
Máquina comprometida 🔓
En este reto hemos visto:

Cómo detectar y explotar una vulnerabilidad LFI.

Cómo escalar privilegios desde un usuario normal usando sudo + binarios vulnerables (perl).

Cómo hacer movimiento lateral a otro usuario (vaxei → luisillo).

Y cómo explotar permisos de escritura en un directorio para ejecutar un script como root.

💡 Una máquina muy entretenida con pasos claros y técnicas útiles que se repiten mucho en CTFs y entornos reales.
