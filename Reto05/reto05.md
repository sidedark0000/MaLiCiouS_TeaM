## Descripción:

Habláis con el director del instituto y os permite acceder al contenido del disco. Al entrar encontráis un pcap. ! Qué es un pcap ! os preguntáis. No lo sabemos pero habrá que investigar. 

Pregunta:  

Los datos os suenan codificados en hexadecimal pero ... ¿Cómo recomponer estas piezas de cada petición? Además,hay mucho ruido de navegación en el fichero.  


Datos proporcionados: 

Fichero pcap. 



## Solución

Para solventarlo, la clave del enunciado es que "hay mucho ruido de navegación" por esto se decide rescatar las trazas de consultas por medio de DNS

para ello, abrimos el Wireshark.
utilizaremos el filtro dns.a y veremos que existen muchas queries dns.
podemos también hacer un análisis de las queries DNS que nos facilita el propio wireshark de la siguiente forma, 
Nos vamos al menú de estadística y seleccionamos DNS.

En esta nueva ventana que nos abre el wireshark, podemos ver datos sobre el tipo de queryies que se hacen, la longitud de los paquetes etc.
algo que a la hora de buscar patrones "fuera de lo común" es la longitud del nombre.
De aquí podemos extraer que el nombre más corto tiene 10 caracteres en la pregunta al DNS en este caso  será a.teads.tv y el más largo contiene 74 caracteres.

como no, wireshark tiene un filtro para esto: 

dns.qry.name.len == 74  ó  dns.qry.name.len eq 74

ambos filtraran unas peticiones dns, veamos si es lo que buscamos: 

Seleccionamos el primer paquete y sacamos del menú contextual el valor del la query: 
```
Frame 6555: 157 bytes on wire (1256 bits), 157 bytes captured (1256 bits) on interface eth0, id 0
Ethernet II, Src: VMware_63:74:a4 (00:0c:29:63:74:a4), Dst: VMware_f5:3d:ad (00:50:56:f5:3d:ad)
Internet Protocol Version 4, Src: 192.168.2.135, Dst: 192.168.2.2
User Datagram Protocol, Src Port: 41126, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x4adb
    Flags: 0x0120 Standard query
        0... .... .... .... = Response: Message is a query
        .000 0... .... .... = Opcode: Standard query (0)
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... .0.. .... = Z: reserved (0)
        .... .... ..1. .... = AD bit: Set
        .... .... ...0 .... = Non-authenticated data: Unacceptable
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        45737465206669636865726f206573206d757920696d706f7274616e7465.localghost.es: type A, class IN
            *Name: 45737465206669636865726f206573206d757920696d706f7274616e7465.localghost.es*
            [Name Length: 74]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
    [Response In: 6556]
```


Nos lo llevamos a la web de [ciberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')&input=NDU3Mzc0NjUyMDY2Njk2MzY4NjU3MjZmMjA2NTczMjA2ZDc1NzkyMDY5NmQ3MDZmNzI3NDYxNmU3NDY1LmxvY2FsZ2hvc3QuZXM) y vemos que nos da: 



Si copiamos todo el dominio hay caracteres que no corresponden estar ahí así que vayamos a algo más fino. 

ya sabemos que esto nos da una frase legible, por lo que vamos a sacar todos los demás a un fichero y desde ahí vamos a tratarlos. 

Bien ya tenemos todos los nombres de dominio en un fichero (una línea en cada uno, yo he llamado al fichero nombres.txt) ahora vamos a quitar lo que no queremos que es ".localghost.es" para ello vamos a usar la propia bash y los comandos internos del sistema: 

`cat nombres.txt |awk -F. '{print $1}'`

pero esto sigue siendo ilegible, ¿que hago? existe un comando en Linux que es xxd como dice el manual "xxd - make a hexdump or do the reverse" "haz un dump en hexadecimal o haz la inversa" 
leyendo un poco más de él, vemos que con -r (revert) y pone que "convierte hexdumps en binario" 
pues vamos al lio no? 
Usaremos el comando anterior para pasarle a xxd todas y cada una de las lineas que se han obtenido. 


```bash
cat nombres.txt |awk -F. '{print $1}' | xxd -r 
```

lo sé, no os sale nada por pantalla ¿verdad? no os preocupéis, para eso tenemos la opción -p que es básicamente la opción de sacar el output en texto plano-

```bash
cat nombres.txt |awk -F. '{print $1}' | xxd -r -p 
```


eh! espera veo el texto que han escondido pero ¿y mi flag? 

Bien, aquí tenemos que volver al wireshark, y aplicar el filtro que por el anterior, podemos deducir.

`dns.opt`

este filtro lo he seleccionado al ver que en el filtro de la longitud se veía un A y luego tras el dominio un OPT, buscando más información sobre esto, https://www.wireshark.org/docs/dfref/d/dns.html 

efectivamente hay un filtro para ello así que lo aplicamos, y podemos ver que ahora tenemos una linea más

realizamos los pasos anteriores sobre el fichero, y volvemos a ejecutar el comando: 

```bash 
cat nombres.txt |awk -F. '{print $1}' | xxd -r -p
```

he ahí la flag.