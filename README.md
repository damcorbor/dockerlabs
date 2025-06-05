# 🐳 DockerLabs.es - Acceso

Bienvenido a este repositorio donde recopilo mis soluciones, apuntes y métodos de acceso a las máquinas de la plataforma [DockerLabs.es](https://dockerlabs.es).

Este repo es una forma personal de documentar lo aprendido, organizar los accesos a las máquinas, y compartir configuraciones útiles.

---

## 🧪 Cómo iniciar un laboratorio

La propia plataforma [DockerLabs.es](https://dockerlabs.es) explica cómo iniciar los laboratorios, pero aquí te dejo un resumen práctico para hacerlo por tu cuenta.

> ⚠️ **Recomendación previa:**  
> Usa un entorno Linux orientado a pentesting, como Kali.  
> En mi caso, utilizo una versión personalizada de **Savitar**, puedes buscarla fácilmente y descargarla si te interesa.

### Pasos para iniciar un laboratorio:

1. **Descarga el laboratorio en formato `.zip` desde DockerLabs.**  
   Normalmente lo encontrarás en el propio reto.


2. **Descomprime el archivo ZIP.**  
   Puedes hacerlo desde la terminal con el comando:

   ```bash
   unzip nombre_laboratorio.zip
   ```
3. **Ejecuta el laboratorio.**  
  Una vez descomprimido suelen haber unos 2 archivos dentro, un ``auto_deploy.sh`` (script de despliegue) y el  ``laboratorio.tar`` (imagen del laboratorio).
  Para iniciar el laboratorio, ejecuta:
   ```
   sudo bash auto_deploy.sh laboratorio.tar
   ```
4. **Obtén la IP del laboratorio.**
   Una vez iniciado, el script suele mostrarte la IP del contenedor en la propia terminal.
