> ### Todos los ejercicios realizados a continuación se encuentran disponibles [***aquí***](https://overthewire.org/wargames/bandit/) y deben realizarse mediante conexión SSH.
> ### *Cada cierto tiempo las instrucciones de los ejercicios varían levemente agregando o quitando acciones, pero el desarrollo mostrado a continuación sirve como una base sólida sin importar los futuros cambios implementados en el Bandit game, pudiendo aplicar de una u otra forma todas las explicaciones de éste documento.*

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

~~~bash
nc -lvnp 6000
~~~
o
~~~bash
nc -nlvp 4646
~~~

Y pegas la contraseña de este nivel 20.

Ahora abres otra terminal, conectas al nivel 20 por ssh, ejecutas ./suconnect 6000 para que lea lo que has puesto en ese puerto y, si te dice correcto, vas al escuchador a ver la contraseña que te ha enviado.

+ ### **Nivel** 21
El texto del nivel dice: Un programa está corriendo a intervalos regulares con cron. Mira en /etc/cron.d/ para ver el comando que se está ejecutando.

Hay varios scripts en /etc/cron.d y he mirado el de bandit22, he examinado con un editor y veo en el código (es muy sencillo) que lo que hace es que vuelca la contraseña a un archivo en /tmp/ así que copio el /tmp/nombredearchivo y hago cat sobre él

+ ### **Nivel** 22
Hay un script de cron de bandit23, lo examino con cat o un editor de texto (nano o vim) y veo que ejecuta esto:

~~~bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
~~~

Cuyo resultado se añade a /tmp/ así que veo qué resultado arroja luego de transformar la cadena en un hash md5.
Ahora, en la terminal del nivel 22 a la que he conectado por ssh, hago cat /tmp/resultadodelscript y veo la contraseña que vuelca ahí.

+ ### **Nivel** 23
Hay que crear un script propio aquí, por suerte es relativamente sencillo.

Hay que ver el script que corre en cron y que comentan las instrucciones. Básicamente, el script dice que si meto otro script mío en /var/spool/bandit24/foo lo correrá.

En éste nivel es muy importante ir revisando siempre los permisos de los directorios y/o archivos a manipular.

Ese script .sh debe ser como el del nivel 21, un cat a la contraseña donde se guarda esta.

La cuestión es que debo crear un directorio de trabajo en /tmp en el que pueda guardar y cambiar permisos. Creo /tmp/dir_name y le doy permisos 777 porque me complico lo mínimo.
~~~bash
dir_name=$(mktemp -d)
~~~
Ahora, creo mi super script y lo edito, en mi caso utilizo vim. 

~~~bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/dir_name/pass.log
chmod o+rw /tmp/dir_name/pass.log
~~~
Ahora le doy permisos de ejecución a mi script o no corre.
~~~bash
chmod +x /tmp/dir_name/script.sh
~~~
Lo copio a la dirección que me dicen las instrucciones.
~~~bash
cp /tmp/dir_name/script.sh /var/spool/bandit24/foo/
~~~
Espero un tiempo a que cron actúe, lo corra,
pudiendo monitorear el proceso con watch (watch -n 1 ls -l), luego hago:
~~~bash
cat /tmp/dir_name/pass.log 
~~~
para ver la contraseña.

+ ### **Nivel** 24
Un demonio está escuchando en el puerto 30002. Si le das la contraseña de 23 un espacio y un pin de 4 dígitos te da la contraseña de 24.

Así que hay que hacer un ataque de fuerza bruta.

Para eso hacemos un script en bash que con, un bucle for, escriba todos los combos posibles a un fichero txt.
o podemos realizar el proceso directamente desde la terminal sin tocar un editor de texto, pero primero debemos crear un directorio en /tmp/ y posicionarnos dentro de él para luego 
~~~bash
for pin in {0000..9999}; do echo "VAfGXJ1PBSsPSnvsjI8p759leLZ9GGar $pin"; done > credentials.txt
~~~
mediante ésta acción volcamos el output del ciclo a un archivo para así poder ingresar mediante netcat
Ahora hemos de conectar con netcat y alimentarle el archivo creado con todas las combinaciones de 4 dígitos que ha creado mi script con el bucle for.

~~~bash
cat credentials.txt | nc localhost 30002 | grep -vE "Wrong|Please enter"
~~~
si no filtramos con grep irá dando un montón de errores hasta que encaja el código de 4 dígitos y arroja la contraseña, Pero al filtrar se mostrara limpiamente la cadena de caracteres deseada.

+ ### **Nivel** 25
Ingenioso, quizá demasiado para mí.

Al entrar por ssh@localhost con la key de bandit26 ejecuta un script en vez de bash que muestra un texto con el comando more y cierra con exit 0.

Así que hacemos muy pequeña la terminal para obligar a que se ejecute el more(ésta utilidad es la clave) y en ese momento pulsamos v y nos abre vim.

Desde vim abrimos /etc/bandit_pass/bandit26 para ver la contraseña. Abrir en Vim se hace con el comando :e

~~~bash
:e /etc/bandit_pass/bandit26
~~~

+ ### **Nivel** 26
Nos encontramos con lo mismo de antes. Usamos el truco de vim una vez disparamos el more con pantalla empequeñecida.

Vim puede ejecutar comandos de terminal, pero como la terminal del usuario es el asquete, cambiamos la terminal que usa Vim con el comando:

:set shell=/bin/bash
Bash
Ahora, vemos que hay un programa bandit27-do que ejecuta como bandit27 pero solo acepta una palabra como argumento. Poner más de una lo fastidia todo, así que, ¿cómo «metemos varias palabras en una»? Haciendo un script y alimentando su nombre al programa bandit27-do.

Así que hacemos un script con una única instrucción que sea

~~~bash
cat /etc/bandit_pass/bandit27
~~~
y lo alimentamos a bandit27-do
~~~bash
./bandit27-do script
~~~
Nota: no pongas el shebang inicial en el script (#!/bin/bash) como he hecho yo, que así no funciona.

+ ### **Nivel** 27
Hay un repositorio en ssh://bandit27-git@localhost/home/bandit27-git/repo y hay que clonarlo (recordemos que sólo nos dejará maniobrar con directorios dentro de /tmp). Hacemos:

~~~bash
git clone ssh://bandit27-git@localhost/home/bandit27-git/repo /tmp/directoriodetrabajo
~~~

Y hay un README con la contraseña que leemos para ver cuál es.

+ ### **Nivel** 28
Hay un repositorio en ssh://bandit28-git@localhost/home/bandit28-git/repo, así que lo clono como antes en un /tmp/directorio. La contraseña en el README está censurada.

Veo los cambios que se han hecho con

~~~bash
git log
~~~

Compruebo que hay uno que pone «fix leak information».

Puedo ver ese commit poniendo

~~~bash
git show [hash de ese *commit*]
~~~

Ahora sí aparece la contraseña.

+ ### **Nivel** 29
Hay un repositorio en ssh://bandit29-git@localhost/home/bandit29-git/repo y lo clono en /tmp/directoriodetrabajo como antes.

El README dice que no hay contraseñas en producción. Pero habrá otras ramas, seguramente.

Veo qué ramas hay con:
~~~bash
git branch -a
~~~
Compruebo que hay una que es remotes/origin/dev y pruebo con esa, así que la examino con:
~~~bash
git checkout -t remotes/origin/dev
~~~
La terminal dice que ha cambiado a esa rama y, cuando leo el README que hay ahí, está la contraseña.

