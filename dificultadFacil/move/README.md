# 📦 Move

![IMAGEN_MAQUINA_PORTADA](./images/portada.png)
> 💡 NOTA:  En mi [repositorio dockerlabs](https://github.com/damcorbor/dockerlabs/tree/main/comandos)  suelo ir dejando una lista con los comandos, herramientas y servicios que he ido usando durante los laboratorios, y los explico un poco por si alguien quiere repasarlos o usarlos como referencia.
---

## 🕵️ Reconocimiento

Empezamos con un escaneo con **nmap** para identificar los servicios expuestos.

![nmap](./images/nmap.png)

Se detectan los puertos **22 (SSH)**, **80 (HTTP)** y **3000 (HTTP)**.  

De momento, sin credenciales poco se puede hacer con SSH, así que nos centramos en los servicios web.

Al entrar por el puerto **80**, aparece la página por defecto de Apache sin nada útil, ni siquiera en el código fuente.

![web1](./images/web1.png)

Lanzamos un **gobuster** para ver si hay rutas ocultas.

![gobuster1](./images/gobuster1.png)

Se encuentra un archivo en `/maintenance.html`.

![web1_1](./images/web1_1.png)

Dentro, aparece el siguiente mensaje:

> **Website under maintenance, access is in /tmp/pass.txt**

Esto lo dejamos apuntado para más adelante, podría ser útil.

Pasamos ahora a ver qué hay en el puerto **3000**, donde encontramos un panel de **Grafana**.

![grafana](./images/grafana.png)

Probamos con las credenciales por defecto `admin:admin`, y funciona. Una vez dentro del dashboard, comprobamos la versión de Grafana que está corriendo.

![dashboard](./images/dashboard.png)

Buscamos vulnerabilidades relacionadas con esa versión usando **searchsploit**.

![search](./images/search.png)

Se encuentra un exploit que permite leer archivos del sistema. Lo copiamos a nuestra máquina.

![locate](./images/locate.png)

---

## 💥 Explotación

Usamos el exploit para leer el archivo `/tmp/pass.txt` mencionado anteriormente, además de `/etc/passwd`.

![exploit](./images/exploit.png)

Con esto obtenemos credenciales válidas y nos conectamos por SSH.

![ssh](./images/ssh.png)

---

## ⬆️ Escalada de privilegios

Ejecutamos `sudo -l` para ver los binarios que se pueden ejecutar como root.

![sudo-l](./images/sudo-l.png)

Se nos permite ejecutar `python3` sobre un archivo `.py` como cualquier usuario y sin contraseña. Para comprobar si podemos modificarlo o borrarlo, listamos sus permisos.

![perm](./images/perm.png)

Confirmamos que tenemos permisos de escritura, así que editamos el script para ejecutar una reverse shell o simplemente abrir una shell como root.

![root](./images/root.png)

Máquina comprometida 🔓

## 🏁 Conclusión

En este reto hemos visto:

- Descubrimiento de servicios y rutas ocultas.
- Acceso a un panel de Grafana usando credenciales por defecto.
- Lectura de archivos sensibles desde una vulnerabilidad en Grafana.
- Escalada de privilegios editando un script con permisos y ejecutado con `sudo`.
