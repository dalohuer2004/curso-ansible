# Repositorio para Curso Ansible
Documentación para el curso de Ansible

# Directorio Debian ssh
Contiene un Dockerfile y los ficheros necesarios para crear una imagen Debian ssh.

Dentro se puede generar la imagen:

`$ docker build -t debian_ssh .`

Ejecutando con:

```
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name debian debian_ssh
```

# Directorio Ubuntu ssh
Contiene un Dockerfile y los ficheros necesarios para crear una imagen Ubuntu ssh.

Dentro se puede generar la imagen:

`$ docker build -t ubuntu_ssh .`

Ejecutando con:

```
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name ubuntu ubuntu_ssh
```

# Directorio Centos ssh
Contiene un Dockerfile y los ficheros necesarios para crear una imagen Centos ssh.

Dentro se puede generar la imagen:

`$ docker build -t centos_ssh .`

Ejecutando con:

```
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name centos centos_ssh
```


# Qué es Ansible?

Ansible es un software que automatiza el aprovisionamiento de software, la gestión de configuraciones y el despliegue de aplicaciones. Está categorizado como una herramienta de orquestación, muy útil para los administradores de sistema y DevOps.

En otras palabras, Ansible permite a los DevOps gestionar sus servidores, configuraciones y aplicaciones de forma sencilla, robusta y paralela

Ansible gestiona sus diferentes nodos a través de SSH y únicamente requiere Python en el servidor remoto en el que se vaya a ejecutar para poder utilizarlo. Usa YAML para describir acciones a realizar y las configuraciones que se deben propagar a los diferentes nodos.

<br>

# Requesitos de instalación

- Como primera medida se crea una clave pública y privada de ssh en el equipo que va a ser el nodo de control.
>nota: Sin escribir “passphrase”

`$ ssh-keygen`

Luego dar a todo lo que salga enter y listo.

<br>

- Luego se realiza la instalación de:

`$ apt install docker.io ansible -y`

>Nota: docker.io también realiza la instalación de `git` que se va a necesitar.

<br>

# Repositorio git del taller

- Se descarga el repositorio para el taller:

`$ git clone https://github.com/jp-cursos/curso-ansible.git`

<br>

- Cuando termina la descarga se genera un directorio llamado `curso-ansible` dentro del cual habrán tres directorios más que son `centos_ssh`, `debian_ssh` y `ubuntu_ssh` con el siguiente contenido:

***curso-ansible:***

    ├── centos_ssh
    │   ├── Dockerfile
    │   ├── run.sh
    │   └── set_root_pw.sh
    ├── debian_ssh
    │   ├── Dockerfile
    │   ├── run.sh
    │   └── set_root_pw.sh
    └── ubuntu_ssh
        ├── Dockerfile
        ├── run.sh
        └── set_root_pw.sh

<br>

# Construcción de las imagenes de docker

- Ya dentro del directorio `centos_ssh` se ejecuta la siguiente orden para construir la imagen de docker:

`$ docker build -t centos_ssh .`

<br>

- Ya dentro del directorio `debian_ssh` se ejecuta la siguiente orden para construir la imagen de docker:

`$ docker build -t debian_ssh .`

<br>

- Dentro del directorio `ubuntu_ssh` se ejecuta la siguiente orden para construir la imagen de docker:

`$ docker build -t ubuntu_ssh .`

<br>

# Iniciar las imagenes de docker

- Para correr el contenedor `centos` de docker se utiliza la siguiente instrucción:

~~~
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name centos centos_ssh
~~~

<br>

- Para correr el contenedor `debian` de docker se utiliza la siguiente instrucción:

~~~
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name debian debian_ssh
~~~

<br>

- Para correr el contenedor `ubuntu` de docker se utiliza la siguiente instrucción:

~~~
docker run -d  -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" --name ubuntu ubuntu_ssh
~~~


<br>

# Consultar la IP a los contenedores

<br>

- Saber la IP del contenedor `centos` y adicionarla al fichero hosts:

~~~
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' centos >> /etc/ansible/hosts
~~~

<br>
<br>

- Saber la IP del contenedor `debian` y adicionarla al fichero hosts:

~~~
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' debian >> /etc/ansible/hosts
~~~

<br>
<br>

- Saber la IP del contenedor `ubuntu` y adicionarla al fichero hosts:

~~~
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu >> /etc/ansible/hosts
~~~

<br>
<br>

# Inventarios

<br>

Ansible funciona con inventarios, y eso no es más que la lista de host o máquinas que se ha guardado en el fichero `hosts` de la siguiente ruta: 

`/etc/ansible/hosts`

<br>

- En el comando anterior donde se consultan las IP de los contenedores y se adicionan las mismas en el fichero `hosts` de ansible para luego conectar con los equipos, ese fichero está en la ruta mencionada en el parrafo anterior:

al final del fichero se va a encontrar los siguiente:

~~~
172.17.0.4
172.17.0.3
172.17.0.2
~~~

<br>

- Para mayor comodidad de administración y despliegue se puede organizar de la siguiente manera haciendo uso de vim o nano, para este caso se hace uso de vim:

`$ vim /etc/ansible/hosts`

~~~
[deb]
debian   ansible_host=172.17.0.2 ansible_python_interpreter=/usr/bin/python3
ubuntu   ansible_host=172.17.0.3 ansible_python_interpreter=/usr/bin/python3
[rhat]
centos   ansible_host=172.17.0.4 ansible_python_interpreter=/usr/bin/python3
~~~

>Notas: debido a que los contenedores son tan reducidos y básicos en el Dockerfile solo se dio la instrucción para que instale `python3` por tal motivo se debe especificar en una variable dentro del fichero hosts en cada linea de los equipos que se van a gestionar, también se puede hacer uso de `agrupación [deb] [rhat]` ó `alias debian, ubuntu, centos` dentro del fichero hosts de Ansible. 

<br>

### Inventarios personalizados

- Si se quiere se puede hacer uso de un inventario personalizado para un grupo de hosts y en ese caso se realizaría de la siguiente manera:

`$ ansible -i /ruta/fichero/hosts nombre_host -m ping`

Ejemplo, se crea un fichero hosts en la sigueinte ruta:

`~/curso-ansible/hosts`

~~~
root@cursoansible:~/curso-ansible# pwd
/root/curso-ansible
root@cursoansible:~/curso-ansible# cat hosts 
[deb]
debian ansible_host=172.17.0.2 ansible_python_interpreter=/usr/bin/python3
~~~

<br>


# Conexión SSH

- Antes de iniciar las tareas de gestión de Ansible se debe establecer conexión `SSH` con cada host, y así verificar la conexión y autenticidad del mismo; con la siguiente Instrucción para cada host se puede realizar:

`$ ssh 172.17.0.4`

`$ ssh 172.17.0.3`

`$ ssh 172.17.0.2`

Se debe dar `yes` a todas las conexiones para que la confianza quede establecida entre los hosts.


<br>

# Módulos de Ansible

<br>



## Módulo ping

El primer módulo que se puede usar para entrar en matería con ansible es `ping`, entonces se coloca `-m` para indicar que módulo se va a usar:

para estos ejemplos se hace uso del fichero `hosts` en la ruta por defecto que es `/etc/ansible/hosts`.

Ejemplos

- para hacer ping a todos:

`$ ansible all -m ping`

<br>

- para hacer ping a el grupo deb:

`$ ansible deb -m ping`

<br>

- para hacer ping a el grupo rhat:

`$ ansible rhat -m ping`

<br>

- para hacer ping solo a ubuntu, usando el alias:

`$ ansible ubuntu -m ping`

<br>

### Uso del fichero hosts en una ruta personalizada

- Se debe estar en la ruta donde se encuentra el fichero `hosts` para hacer uso del inventario personalizado y se hace de la siguiente manera:

>Ruta: ~/curso-ansible/hosts

`$ ansible -i hosts debian -m ping`

>Nota: ese sería el inventario para este proyecto personalizado de Ansible.

<br>

## Módulo setup

<br>

- Para ver las variables o información que ya a guardado Ansible de cada una de las máquinas se pude hace uso del siguiente comando:

`$ ansible all -m setup`

<br>
<br>

- usando el inventario personalizado sería de la siguiente manera:

`$ ansible -i hosts debian -m setup`

- Este comando muestra toda la información en variables y a ese contenido se le llama `facts` ó `hechos` que se puede utilizar al crear `templates` o al utilizar `playbooks`, con esto se pueden cambiar las variables entre una máquina y otra.

<br>
<br>

## Facts o hechos

- Para hacer uso de las opciones agregadas a un módulo de cada `facts` lo podemos hacer de la siguiente manera con la opción `-a`:


`$ ansible all -m setup -a "filter=ansible_all_ipv4_addresses"`

`$ ansible deb -m setup -a "filter=ansible_all_ipv4_addresses"`

`$ ansible rhat -m setup -a "filter=ansible_all_ipv4_addresses"`

<br>

## Ejecución de comandos en Ansible desde el equipo de control

- También se utiliza para ejecutar un comando dentro de la máquina haciendo uso de la opción `-a` así como se muestra:

`$ ansible deb -a uptime`

<br>

- Si se quiere ver los procesos de cada máquina lo realizamos de la siguiente manera:

`$ ansible all -a "ps aux"`

<br>

## Módulo apt

- Para la instalación de la aplicación `nginx` en el conjunto de máquinas `deb`, se puede hacer uso del módulo `apt` tal como se muestra en el ejemplo:


`$ ansible deb -m apt -a "name=nginx state=present"`

Sin embargo también se puede hacer la eliminación del paquete de la siguiente forma:

`$ ansible deb -m apt -a "name=nginx state=absent"`

<br>

## Módulo package

- Con el módulo `package` primero comprueba que distribución tiene el host y luego procede a instalar con el gestor de paquetes que le corresponde a cada una para así cumplir con lo requerido.

`$ ansible all -m package -a "name=nginx state=present"`

Al igual que para la instalación que se hizo con package se puede eliminar de la siguiente manera:

`$ ansible all -m package -a "name=nginx state=absent"`

<br>

## Módulo service

- Al haber instalado nginx se debe de iniciar el servicio, para tal efecto se hará para las máquinas del grupo deb de la siguiente manera ya que no se tiene todo el sysmted instalado:

`$ ansible deb -a "/etc/init.d/nginx start"`

También se puede hacer uso del siguiente comando con el módulo `service`  y el estado `started` para iniciar el servicio:

`$ ansible deb -m service -a "name=nginx state=started"`

<br>

>Notas: Para verificar que Nginx si inicio como servicio se puede hacer con el comando `curl` y la IP de la máquina a verificar de la siguiente manera:

<br>

`$ curl 172.17.0.2` para el host debian

`$ curl 172.17.0.3` para el host ubuntu

Con estos comandos se puede apreciar que carga la web de bienvenida de Nginx por la salida de la linea de comandos.

<br>

- O por el contrario si se quiere parar un servicio en este caso nginx se  utiliza el módulo `service` con el estado `stopped` y se realiza de la siguiente manera:

<br>

`$ ansible all -m service -a "name=nginx state=stopped"`

<br>


### Ejemplo de como iniciar un servicio en la máquina centos

- Para iniciar el servicio de nginx en la máquina `centos` sería algo distinto ya que no se conoce la ruta del servicio en esa distribución y se puede verificar de la siguiente manera:

`$ ansible centos -a "whereis nginx"`

~~~
root@cursoansible:~# ansible centos -a "whereis nginx"
centos | CHANGED | rc=0 >>
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz /usr/share/man/man3/nginx.3pm.gz
~~~

ya con esto listo se puede iniciar el servicio con el siguiente comando:

`$ ansible centos -a "/usr/sbin/nginx"`

También se puede hacer uso del siguiente comando para iniciar el servicio:

`$ ansible centos -m service -a "name=nginx state=started"`

<br>

>Notas: Para verificar que Nginx si inicio como servicio se puede hacer con el comando `curl` y la IP de la máquina a verificar de la siguiente manera:

`$ curl 172.17.0.4` para el host centos

<br>

