# 🖥️ Consolelog

![IMAGEN_MAQUINA_PORTADA](./images/portada.png)
> 💡 NOTA:  En mi [repositorio dockerlabs](https://github.com/damcorbor/dockerlabs/tree/main/comandos)  suelo ir dejando una lista con los comandos, herramientas y servicios que he ido usando durante los laboratorios, y los explico un poco por si alguien quiere repasarlos o usarlos como referencia.
---

## 🕵️ Reconocimiento

Arrancamos con un escaneo usando **nmap** para ver qué puertos y servicios están activos.

![nmap](./images/nmap.png)

Detectamos los puertos **80 (HTTP)**, **3000 (HTTP)** y **5000 (SSH)** abiertos. Ejecutamos dos **gobuster**, uno para el puerto 80 y otro para el 3000, buscando posibles directorios ocultos.

![gobuster1](./images/gobuster1.png)  
![gobuster2](./images/gobuster2.png)

Accedemos a la web principal (puerto 80) y no muestra nada especialmente útil.

![web1](./images/web1.png)

Echando un vistazo al código fuente aparece un JS relacionado con autenticación.

![codigo1](./images/codigo1.png)

Navegando un poco más llegamos al directorio `/backend`, donde hay un archivo `.js` que contiene lo que parece ser una contraseña.

![codigo1_1](./images/codigo1_1.png)

## 💥 Explotación

Con la contraseña, lanzamos un ataque de fuerza bruta con **Hydra**. Usamos un diccionario de nombres de usuario y la contraseña encontrada antes.

![hydra](./images/hydra.png)

Hydra devuelve un acceso válido con el usuario **lovely**, así que probamos a conectarnos por SSH.

![login](./images/login.png)

## 🧗 Escalada de privilegios

Lanzamos `sudo -l` para ver qué binarios se pueden ejecutar con privilegios elevados.

![sudo-l](./images/sudo-l.png)

Vemos que el usuario puede ejecutar **nano** como cualquier otro usuario, sin necesidad de contraseña. Usando **GTFOBins**, aprovechamos esto para escalar privilegios.

![gtfobins](./images/gtfobins.png)  
![root](./images/root.png)

Máquina comprometida 🔓

---

## 🏁 Conclusión

En este reto hemos visto:

- Descubrir directorios y archivos interesantes a través de fuerza bruta con **gobuster**.
- Encontrar credenciales en código JavaScript expuesto en la parte web.
- Obtener acceso inicial mediante **fuerza bruta por SSH** con **Hydra** y las credenciales encontradas.
- Enumerar permisos con `sudo -l` y aprovechar la ejecución de **nano** como cualquier usuario para **escalar privilegios** a root usando técnicas de **GTFOBins**.

