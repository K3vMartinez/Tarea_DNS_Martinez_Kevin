# Servicio de nombres de dominios
## Bind 9
* https://www.isc.org/bind/
* https://bind9.readthedocs.io/
* https://www.fpgenred.es/DNS/index.html
## Infraestructura
Reutilizaremos las MV de la práctica de **ssh**. Dos MV dentro de una **red NAT**:
* **Servidor**: con un Ubuntu server sin entorno gráfico.
    * Usuario: **sergio**, contraseña: **sergio**.
* **Casa**: con un Lubuntu con el entorno gráfico por defecto (LXQt).
    * Usuario: **carmen**, contraseña: **carmen**.

Desde el equipo **Casa** nos conectaremos al equipo **Servidor** mediante una conexión **ssh** autentificándonos
mediante claves asimétricas **ed25519**.
## Instalación y uso básico
1. Acceder al servidor:
```bash
ssh -i ~/.ssh/id_ed25519 10.0.2.4
```
> Casi toda la instalación y configuración la debemos hacer con privilegios de administrador
podemos ejecutar **sudo** en todas las instrucciones o cambiar al usuario administrador **sudo su**.

2. Instalar bind 9:
```bash
sudo apt update
sudo apt install bind9 bind9utils
```
> La instalación crea el usuario **bind** que ejecuta el servicio DNS denominado **named**. Puedes
comprobarlo mostrando el contenido del archivo **/etc/passwd**.

3. Comprobar estado del servicio **bind**:
```bash
sudo systemctl status bind9
```
> Muestra advertencias ya que aún no lo hemos configurado.

4. Con los siguientes comandos lo activaremos para que se inicie al arrancar el servidor y lo iniciaremos:
```bash
sudo systemctl enable bind9
sudo systemctl start bind9
```
> Otras comandos del servicio son:
> ```bash
>sudo systemctl enable bind9
>sudo systemctl start bind9
>sudo systemctl stop bind9
>sudo systemctl restart bind9
>sudo systemctl status bind9
>sudo systemctl reload bind9
>sudo systemctl show bind9
>```

5. Reglas firewall:
```bash
sudo ufw enable
sudo ufw allow bind9
sudo ufw status
```
6. Probar desde el cliente qué puertos tiene abiertos el servidor, en nuestro ejemplo desde el equipo
**Casa** ejecutaremos:
```bash
nmap 10.0.2.4 -p 1-1024
```
> Por defecto el servicio DNS utiliza el puerto 53.

> Si no tienes instalada esta utilidad, instalalá con: **sudo apt install nmap**. Esta
comprobación también se puede hacer desde el propio servidor, pero es menos fiable que
desde otro equipo ya que puede conectarse por localhost.

7. Comprobar en el equipo **Servidor** qué conexiones tiene abiertas:
```bash
sudo ss -natp | grep named
sudo ss -naup | grep named
```

## Archivos de configuración
1. El archivo principal de configuración del **bind** es: **/etc/bind/named.conf**. En él vemos que hace
referencia a otros tres archivos de configuración:
    * **/etc/bind/named.conf.options**: hace referencia al archivo de configuración que posee opciones genéricas.
    * **/etc/bind/named.conf.local**: hace referencia al archivo de configuración para opciones particulares.
    * **/etc/bind/named.conf.default-zones**: hace referencia al archivo de configuración de zonas.

Abre el archivo y muesta estas referencias.
```bash
sudo nano /etc/bind/named.conf
```

## Verificar archivos de configuración
1. Puedes realizar una verificación de los ficheros de configuración y de zona por posibles fallos
mediante los comandos **named-checkconf** y **named-checkzone** respectivamente. Estos comandos
suelen ejecutarse con la siguiente sintaxis:
    * **named-checkconf [-p] {filename}**: Comprueba la sintaxis, pero no la semántica de un fichero de configuración named. El fichero se analiza y comprueba por errores de sintaxis, junto con todos los archivos incluidos en él.<br>
    Parámetros:
        * El parámetro **-p**: imprime la salida de named.conf y los ficheros incluidos en forma canónica si no fueron detectados errores.
        * **filename**: es el nombre del archivo de configuración que se desea comprobar. Si no se especifica, por defecto es **/etc/named.conf**.
    * **named-checkzone {zonename} {filename}**: Comprueba la sintaxis y la integridad de un archivo de zona. Realiza las mismas comprobaciones que hace named al cargar una zona. Esto hace que sea útil para comprobar los archivos de zona antes de configurarlos en un servidor de nombres.<br>
    Parámetros:
        * **zonename** es el nombre de dominio de la zona que se comprueba.
        * **filename** es el nombre del archivo de zona.

Prueba a verificar los siguientes archivos:

Verificar el fichero de configuración:
```bash
named-checkconf -p /etc/bind/named.conf
```

Verificar el dominio de zona ejemplo.com en el archivo de zona:
```bash
named-checkzone ejemplo.com /etc/bind//db.local
```

## Configurar servidor DNS
### Configurar Reenviadores (forwarders)
1. Primero indicar que cuando se ejecute **bind** lo haga solo sobre IPv4. Editar el archivo **named** y añadir **-4**.
```bash
sudo nano /etc/default/named
```
```bash
OPTIONS="-u bind -4"
```
2. Añadir al bloque **options** del archivo **named.conf.options** los reenviadores e indicar que no se validen las conexiones seguras DNS con las instrucciones siguientes:
```bash
sudo nano /etc/bind/named.conf.options
```
```bash
forwarders {
1.1.1.1;
8.8.8.8;
};
dnssec-validation no;
```
> En la siguiente dirección tienes unas estadísticas de velocidad de respuesta de distintos DNS: **DNSPerf**

3. Verificar el archivo anterior:
```bash
sudo named-checkconf /etc/bind/named.conf.options
```

Verificar el archivo de configuración principal **named.conf**:
```bash
sudo named-checkconf
```
4. Reiniciar el servicio **bind**:
FINAL DE PAGINA 4 PRINCIPIO DE LA 5 DEL PDF