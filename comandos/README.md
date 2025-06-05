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

### hydra
```
hydra -L usuarios.txt -p purpl3 -I ssh://172.17.0.2 -V -F -u
```

###   Find
```
find / -perm -4000 -user root 2>/dev/null
```


