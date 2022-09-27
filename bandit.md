> Todos los ejercicios realizados a continuación se encuentran disponibles [***aquí***](https://overthewire.org/wargames/bandit/) y deben realizarse mediante conexión SSH

+ ### **Nivel** 0
La contraseña está en un archivo readme en el directorio home, no hay complicación, es leerlo en la terminal.
Lo más habitual es usar cat sobre ese fichero

~~~bash
cat readme
~~~

+ ### **Nivel** 1
El archivo tiene como nombre un guion: -

Para verlo, se tiene que abrir dando la ruta completa del archivo a la instrucción cat. Es decir:

~~~bash
cat ./-
~~~
~~~bash
cat $(pwd)/-
~~~

+ ### **Nivel** 2
El archivo tiene espacios en su nombre, así que para llamarlo una opción sería *escapar* esos espacios precediendo dicho espacio con \ cuando hagamos cat.

~~~bash
cat spaces\ in\ this\ filename
~~~
~~~bash
cat spaces*
~~~

+ ### **Nivel** 3
El archivo con la contraseña está oculto, así que hay que hacer un ls con -a (all) para ver todos los archivos, ocultos o no, de modo que abrimos el que nos indica y que se llama .inhere
~~~bash
cat .inhere
~~~
~~~bash
find . -type f | grep "hidden" | xargs cat
~~~

+ ### **Nivel** 4
Nos dice que la contraseña está en el directorio inhere, en el archivo que es legible por humanos.  
Usando

~~~bash
file ./*
cat ./inhere/-file07
~~~

La instrucción determina los tipos de los archivos que hay en el directorio y aparece el que es de tipo ASCII (legible por nosotros) y que queremos.

+ ### **Nivel** 5
Comienzan niveles que nos exigen usar la instrucción find, probablemente una de las más útiles y poderosas de la línea de comandos.

En este caso nos dice que el archivo es legible por humanos, no ejecutable y con tamaño de 1033 bytes. Usando:

~~~bash
find ./ -type f ! -executable -size 1033c | xargs cat 
~~~

lo podemos encontrar rápido y sin problemas. -type f significa que busque archivos (files) -size determina el tamaño del archivo, el sufijo c tras el número es indicativo de bytes.

+ ### **Nivel** 6
Nos dice que el archivo pertenece al usuario bandit7, al grupo bandit6 y tiene un tamaño de 33 bytes. De nuevo usamos find con los argumentos adecuados que buscan por usuario (-user) y tamaño deseado (-size):

~~~bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null | xargs cat
~~~
+ ### **Nivel** 7
Este nivel nos dice que la contraseña está en el archivo data.txt al lado de la palabra “millionth”. El comando grep es nuestro amigo.  
Usando alguna de las siguientes opciones válidas:

~~~bash
cat data.txt | grep "millionth" | awk '{print $2}'
~~~
~~~bash
cat data.txt | grep "millionth" | awk 'NF{print $NF}'
~~~
~~~bash
cat data.txt | grep "millionth" | xargs | cut -d ' ' -f 2
~~~

Encontramos lo que queremos.

Soy consciente de que podría usar grep únicamente (sin cat) alimentando el nombre de archivo tras la cadena que quiero encontrar, pero manías de novato las de usar cat y pasarlo a grep con una pipe.

+ ### **Nivel** 8
La contraseña está almacenada en el archivo data.txt y en la única línea que está solo una vez y por tanto no se repite.

Aquí nos serán útiles dos de las instrucciones que propone el nivel: sort y uniq.

Usando sort para ordenar las líneas y uniq -u para revelar la que se repita solo una vez, la encontraremos. La instrucción completa sería:

~~~bash
sort data.txt | uniq -u
~~~

+ ### **Nivel** 9
Nos dice que está en un archivo binario, en la parte legible para humanos y después de tres signos ===.

El comando strings es lo que queremos para encontrar lo legible por humanos en un binario y grep para encontrar los = de la pista:

~~~bash
strings data.txt | grep "===" | tail -n 1 | awk 'NF{print $NF}'
~~~
+ ### **Nivel** 10
El archivo data.txt está codificado en base64. Decodificarlo para la contraseña es muy sencillo.

~~~bash
cat data.txt | base64 -d | awk '{print $4}'
~~~

+ ### **Nivel** 11
El archivo data.txt está encriptado con el método rot13, que cambia una letra por la que está 13 posiciones más adelante en el alfabeto.

Miramos un poco qué comandos necesitamos para traducir eso a algo que entendamos y lo que queremos es tr (translate). El comando es:

~~~bash
cat data.txt | tr '[A-Za-z]' '[N-ZA-Mn-za-m]' | awk 'NF{print $NF}'
~~~

Quiere decir que las mayúsculas de la A a la Z se cambian empezando por la N, pasando por la Z y siguiendo por la A hasta terminar en la M.

+ ### **Nivel** 12
Las instrucciones nos dicen que es un hexdump de un archivo comprimido muchas veces. Sin embargo, nos encontramos con que ese archivo comprimido está en formato .txt.

*Como recomendación personal se puede copiar el contenido de data.txt para jugar con el archivo en local de una manera mas cómoda.*

Primero hemos de revertir el archivo a binario para poder manejarlo bien. Eso se hace con xxd -r (revertir)

~~~bash
cat data.txt | xxd -r | sponge data.txt
~~~

Ahora vamos usando el comando file para saber el tipo de archivo comprimido concreto que es y vamos renombrando y descomprimiendo con lo que corresponda (tar, gzip, bzip2).

Adicionalmente podemos ocupar la herramineta de 7z para mirar el contenido de  nuestro archivo.

~~~bash
file data.txt

mv data.txt data.gz

7z l data.gz
~~~

Por se deseas ver el comando compactado a modo de ejemplo aquí está el resultado:

~~~bash
(7z x data.gz  && 7z x data2.bin && 7z x data2 && 7z x data4.bin && 7z x data5.bin && 7z x data6.bin && 7z x data6 && 7z x data8.bin) &>/dev/null && cat data9.bin | awk 'NF{print $NF}'
~~~
Seguimos comprobando el tipo de archivo hasta que el comando file nos diga que es ASCII y por fin podemos leerlo.

+ ### **Nivel** 13
la contraseña está en /etc/bandit_pass/bandit14 pero pertenece al usuario bandit14 y solo lo puede leer él.

Comprobamos que tenemos una clave privada ssh en el directorio, así que podemos usarla para entrar por ssh en local con la identidad del usuario bandit14, así que hacemos:

~~~bash
ssh -i sshkey.private bandit@localhost -p 2220
~~~
Así conectamos como bandit 14, de manera que ya podemos hacer cat al archivo /etc/bandit_pass/bandit14

+ ### **Nivel** 14
La contraseña la consigues enviando la del nivel anterior al puerto 30000 de localhost. Conectamos ahí con:

~~~bash
telnet localhost 30000
~~~
o también 
~~~bash
nc localhost 30000
~~~
Pegamos la contraseña anterior una vez conectados para obtener la nueva.

+ ### **Nivel** 15
La contraseña la consigues enviando la del nivel anterior al puerto 30001 de localhost, pero encriptada por ssl, de modo que no nos vale conectar con Telnet, porque no encripta.

He visto que nombra s_client entre los comandos que nos serán útiles en este nivel. Lo que quiero, tras investigar un poco, es:

~~~bash
openssl s_client -connect localhost:30001
~~~
o también
~~~bash
ncat --ssl localhost 30001
~~~
Y pegamos la contraseña del nivel anterior para que nos dé la que queremos.

+ ### **Nivel** 16
Las credenciales se consiguen enviando la contraseña de este nivel a un puerto abierto en el rango 31000-32000. Primero hemos de ver quién está abierto y cuál habla SSL.

Para eso, utilizo nmap:

~~~bash
nmap --open -T5 -v -n -p 31000-32000 localhost
~~~

El puerto 31790 es el correcto que nos devuelve la rsa key que tenemos que pegar en un archivo tmp/sshkey.private con un editor de texto (nano o vim) por ejemplo.

Usamos /tmp porque es el directorio en el que podremos trabajar bien, sin problemas de permisos a la hora de crear archivos, etc.

~~~bash
nano /tmp/sshkey.private
~~~
o
~~~bash
nano /tmp/id_rsa
~~~

*Recuerda que las claves privadas deben tener el privilegio 600 para que solo el propietario tenga permisos para leer y manipular*

Ahora, conectamos por ssh con ese archivo que acabamos de crear (a través del parámetro -i) y con usuario y host bandit17@localhost

~~~bash
ssh -i /tmp/sshkey.private bandit17@localhost
~~~

+ ### **Nivel** 17
Uno fácil, hay dos archivos, la contraseña es la línea diferente que pertenece a password.new

Así que usamos el comando diff para encontrar la diferencia entre esos dos archivos.

~~~bash
diff password.old password.new
~~~

+ ### **Nivel** 18
Alguien ha modificado el .bashrc para echarte en cuanto te conectas. Para evitarlo hay que conectar como siempre por ssh y poner /bin/bash, de ese modo tienes shell

~~~bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 /bin/bash
~~~

+ ### **Nivel** 19
Hay un ejecutable y lo usamos como nos dicen las instrucciones de la habitación, primero el ejecutable y luego escribimos tras él en la misma línea el comando:

~~~bash
cat /etc/bandit_pass/bandit20
~~~

Que es al que nos permite acceder gracias a los permisos que nos otorga el ejecutable.

+ ### **Nivel** 20
Este tiene tela si eres muy novato aún en el tema de listeners. Hay un ejecutable ./suconnect que lee en el puerto que le digas y, si lo que lee es la contraseña del nivel 20, te envía el del 21.

O sea que hay que usar 2 terminales.

En la primera activas un escuchador de netcat en un puerto no usado, por ejemplo:

~~~basj
nc -lvnp 6000
~~~
o
~~~bash
nc -nlvp 4646
~~~

Y pegas la contraseña de este nivel 20.

Ahora abres otra terminal, conectas al nivel 20 por ssh, ejecutas ./suconnect 6000 para que lea lo que has puesto en ese puerto y, si te dice correcto, vas al escuchador a ver la contraseña que te ha enviado.

