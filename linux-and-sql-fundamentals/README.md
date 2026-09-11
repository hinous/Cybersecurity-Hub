# Guia_Linux
Guía de Referencia Linux: 



Navegación del Sistema de Archivos

pwd: Imprime la ruta del directorio de trabajo actual.

ls: Lista los nombres de archivos y directorios en el directorio actual.

ls -a: Lista todos los archivos, incluyendo los archivos ocultos.

ls -l: Muestra permisos, propietario, grupo, tamaño y fecha de última modificación.

cd [directorio]: Cambia al directorio especificado.

cd ..: Retrocede un nivel en la jerarquía de directorios.

whoami: Devuelve el nombre del usuario con sesión activa.



Lectura de Archivos

cat [archivo]: Muestra el contenido completo de un archivo.

head [archivo]: Muestra las primeras 10 líneas de un archivo por defecto.

head -n [número] [archivo]: Muestra el número específico de líneas iniciales indicadas.

tail [archivo]: Muestra las últimas 10 líneas de un archivo por defecto.

tail -n [número] [archivo]: Muestra el número específico de líneas finales indicadas.

less [archivo]: Permite visualizar el contenido de un archivo una página a la vez.



Gestión del Sistema de Archivos

cp [origen] [destino]: Copia un archivo o directorio a una nueva ubicación.

mv [origen] [destino]: Mueve o renombra un archivo o directorio.

mkdir [nombre]: Crea un nuevo directorio.

rm [archivo]: Elimina o borra un archivo.

rmdir [directorio]: Elimina un directorio siempre que esté vacío.

touch [archivo]: Crea un nuevo archivo vacío.

nano [archivo]: Abre o crea un archivo en el editor de texto de línea de comandos.



Filtros y Búsqueda

find [ruta] -name "[nombre]": Busca archivos y directorios que coincidan con el nombre exacto (sensible a mayúsculas).

find [ruta] -iname "[nombre]": Busca archivos y directorios sin distinguir entre mayúsculas y minúsculas.

grep "[cadena]" [archivo]: Busca y devuelve las líneas que contienen una cadena de texto específica.

| (pipe): Envía la salida de un comando como entrada de otro para su procesamiento.



Usuarios y Permisos

sudo: Ejecuta comandos con privilegios elevados de superusuario.



chmod [modificador] [archivo]: Cambia los permisos de lectura (r), escritura (w) y ejecución (x) para usuario (u), grupo (g) u otros (o).



chown [usuario] [archivo]: Cambia el propietario de un archivo o directorio.



chown :[grupo] [archivo]: Cambia el grupo propietario de un archivo o directorio.

useradd [usuario]: Añade un nuevo usuario al sistema.

userdel [usuario]: Elimina un usuario del sistema.

usermod: Modifica las configuraciones de una cuenta de usuario existente.



Ayuda en Linux

man [comando]: Muestra el manual detallado de un comando específico.

whatis [comando]: Muestra una descripción breve del comando en una sola línea.

apropos [cadena]: Busca en las descripciones del manual comandos que contengan la palabra clave indicada.

