# En el primer ejercicio nos avisan de un probable ataque de fuerza bruta y nos regalan estos datos:

You’ve just started your first shift as a SOC analyst at an MSSP. Only a few minutes have passed since an alert about a possible brute force.
attack appeared on the platform.
    

### Alert Details:
```slp    
    Alert Name: Brute Force Activity Detection
    Time: 17/09/2025 9:00:21 AM
    Target Host: tryhackme-2404
    Source IP: 10.10.242.248
    Your job is to investigate this activity and decide whether it should be considered suspicious.
```





## Para contestar la primera pregunta usamos la línea: 

     
    index="linux-alert" src_ip="10.10.242.248" source="auth.log" Failed "john.smith"

    R= 500

## Segunda pregunta:
La segunga pregunta nos pide el tiempo que duró el ataque, se responde de manera muy simple, ya que con el filtro de los intentos puesto, en las graficas podemos observar solo 5 líneas y cada línea nos dice que es un minuto. OPCIONAL: también podrías escribir la siguiente busqueda para llegar a la misma respuesta. 

```slp
index="linux-alert" src_ip="10.10.242.248" "Failed password"
stats min(_time) as start max(_time) as end
 eval duration_minutes = (end - start) / 60
```
    R= 5 mins

## La tercera pregunta: 
Nos solicita saber a que usuario escaló privilegios "john.smith"    Es muy simple contestar esto, ya que solo buscando el index correcto en la barra lateral de busqueda encontramos la opcion de actions, dentro   
de la cual tenemos "success". Le damos click y ahí tenemos los comandos clarosla respuesta de la escalada de privilegios a root.
    R= Root

## La cuarta...
y ultima de esta tarea nos pide  a que nombre de usuario se cambió para persistir. Podriamos buscar directamente el comando adduser o en la barra lateral>actions podemos encontrar la accion con concepto "create". ahí encontramos la respuesta.
R= system-utm
