# Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 (completando el LAMP)

| | |
|---|---|
| **Alumno** | Nabil Aisatin |
| **Ciclo** | Sistemas Microinformáticos y Redes (SMR) |
| **Módulo** | Aplicaciones Web |
| **Curso** | 2026/2027 |
| **Fecha** | Octubre de 2026 |
| **Servidor** | Ubuntu Server 24.04 (máquina virtual, usuario `smr-web`, IP `192.168.56.17`) |

---

## Índice

0. [Introducción: qué es una base de datos y qué es el LAMP](#0-introducción-qué-es-una-base-de-datos-y-qué-es-el-lamp)
1. [Preparación del sistema](#1-preparación-del-sistema)
2. [Instalación de MariaDB](#2-instalación-de-mariadb)
3. [Comprobación del funcionamiento](#3-comprobación-del-funcionamiento)
4. [Comandos principales](#4-comandos-principales)
5. [¿Qué pasa si MariaDB está parado?](#5-qué-pasa-si-mariadb-está-parado)
6. [Instalación de PHP](#6-instalación-de-php)
7. [Comprobación final](#7-comprobación-final)
8. [Resumen de comandos (chuleta)](#8-resumen-de-comandos-chuleta)
9. [Conclusiones](#9-conclusiones)

---

## 0. Introducción: qué es una base de datos y qué es el LAMP

**Base de datos:** es como un archivador muy ordenado donde un programa guarda su información para encontrarla rápido. Se organiza en **tablas** (como hojas de cálculo), con **columnas** (qué datos se guardan) y **filas** (cada elemento guardado).

**¿Para qué la necesita WordPress?** WordPress guarda en la base de datos todo lo que no son ficheros: entradas, páginas, comentarios, usuarios y la configuración del sitio. Cada vez que alguien visita la web, WordPress le pide los datos a la base de datos.

**MariaDB:** es el programa (el *gestor de base de datos*) que guarda los datos, los protege y responde a las peticiones. Viene en los repositorios de Ubuntu y funciona muy bien con WordPress.

**LAMP:** es el conjunto de cuatro piezas necesarias para alojar WordPress:

| Letra | Pieza | Qué hace | Estado antes de esta práctica |
|---|---|---|---|
| **L** | Linux (Ubuntu 24.04) | Sistema operativo | ✅ Instalado |
| **A** | Apache | Sirve la web al navegador | ✅ Instalado (práctica anterior) |
| **P** | PHP | Lenguaje en el que está hecho WordPress; lo ejecuta | ❌ Se instala hoy |
| **M** | MariaDB | Guarda los datos | ❌ Se instala hoy |

Flujo de una visita: **Navegador → Apache → PHP → MariaDB**, y la respuesta vuelve por el mismo camino.

---

## 1. Preparación del sistema

La práctica anterior dejó dos sitios virtuales activos (`sitio1` y `sitio2`) y el sitio por defecto desactivado. Para esta práctica quiero volver a tener **un solo sitio**, el de por defecto (`000-default.conf`).

> ⚠️ **Nota importante:** el enunciado solo pone `a2ensite 000-default.conf` (activar el de por defecto). Para que de verdad queden desactivados `sitio1` y `sitio2`, hay que añadir también `a2dissite`.

```bash
sudo a2dissite sitio1.conf sitio2.conf
sudo a2ensite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

**Explicación:**

- `a2dissite` quita los enlaces simbólicos de `sitio1` y `sitio2` en `sites-enabled` (los ficheros siguen existiendo en `sites-available`).
- `a2ensite 000-default.conf` vuelve a crear el enlace del sitio por defecto.
- `configtest` comprueba que no hay errores de sintaxis antes de aplicar.
- `reload` aplica la configuración **sin cortar** las conexiones activas.

Ahora actualizo el sistema y compruebo que Apache sigue funcionando:

```bash
sudo apt update
sudo apt upgrade -y
systemctl is-active apache2
```

**Explicación:** `apt update` actualiza la lista de paquetes disponibles, `apt upgrade -y` instala las actualizaciones (`-y` responde «sí» automáticamente) y `systemctl is-active apache2` responde en una sola palabra si el servicio está funcionando (`active`) o no (`inactive`).

Salida de la última línea:

```text
active
```

📸 **Captura: respuesta de `systemctl is-active apache2`**

![Apache activo](img/mariadb-1.png)

**Interpretación:** Apache está activo, así que la primera pieza del LAMP sigue funcionando antes de instalar las demás.

---

## 2. Instalación de MariaDB

```bash
sudo apt install mariadb-server -y
```

**Explicación:** instala el servidor de base de datos MariaDB. Ubuntu lo deja ya configurado, arrancado y habilitado para arrancar solo con el sistema.

Compruebo la versión instalada:

```bash
mariadb --version
```

Salida:

```text
mariadb  Ver 15.1 Distrib 10.11.14-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper
```

📸 **Captura: versión de MariaDB**

![Versión de MariaDB](img/mariadb-2.png)

**Explicación:** la versión instalada es **MariaDB 10.11.14**. El «15.1» es la versión del cliente de línea de comandos (`mariadb`), no del servidor; la del servidor es la que aparece tras `Distrib`. `EditLine wrapper` es la librería que permite editar la línea de comandos con las flechas.

---

## 3. Comprobación del funcionamiento

### 3.1. Estado del servicio

```bash
sudo systemctl status mariadb
systemctl is-enabled mariadb
```

**Explicación:**

- `systemctl status mariadb` muestra si el servicio está funcionando **ahora mismo**. Debe aparecer `Active: active (running)` en verde. Para salir de la pantalla de estado se pulsa **`q`**.
- `systemctl is-enabled mariadb` dice si arrancará solo al encender el equipo. Debe responder `enabled`.

Salida esperada:

```text
● mariadb.service - MariaDB 10.11.14 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since ...
enabled
```

> 📸 **PENDIENTE: captura de `sudo systemctl status mariadb`** (se ve `active (running)`) y de `systemctl is-enabled mariadb` (`enabled`). Guárdala como `img/mariadb-3.1.png` y descomenta la línea de abajo:
>
> `![Estado de MariaDB](img/mariadb-3.1.png)`

### 3.2. Entrar en la base de datos

```bash
sudo mariadb
```

📸 **Captura: mensaje de bienvenida del monitor de MariaDB**

![Monitor de MariaDB](img/mariadb-3.2.png)

**Explicación:** al entrar con `sudo mariadb` aparece el mensaje *«Welcome to the MariaDB monitor»* y el prompt cambia a `MariaDB [(none)]>`. `(none)` significa que todavía no he seleccionado ninguna base de datos. Entro como administrador (root de MariaDB) gracias a `sudo`, sin contraseña, porque MariaDB en Ubuntu autentica por *socket*: reconoce que soy root del sistema. Esto confirma que el servidor **funciona y responde**. Para salir:

```sql
exit
```

---

## 4. Comandos principales

| Comando | Función | Explicación con mis palabras |
|---|---|---|
| `sudo systemctl start mariadb` | Arranca el servicio | Lo enciende si estaba apagado |
| `sudo systemctl stop mariadb` | Detiene el servicio | Lo apaga; nadie podrá conectarse a la base de datos |
| `sudo systemctl restart mariadb` | Reinicia el servicio | Lo apaga y lo vuelve a encender, cortando las conexiones abiertas |
| `sudo systemctl status mariadb` | Muestra su estado | Dice si está encendido, apagado o con errores, y enseña las últimas líneas del log |
| `sudo systemctl enable mariadb` | Arranque automático | Hace que se encienda solo cada vez que arranca Ubuntu |

Comandos probados:

```bash
sudo systemctl start mariadb
sudo systemctl stop mariadb
sudo systemctl restart mariadb
sudo systemctl status mariadb
sudo systemctl enable mariadb
```

> **Diferencia clave que suele preguntarse:** `start`/`stop` actúan **ahora**; `enable`/`disable` actúan en el **próximo arranque** del equipo. Un servicio puede estar `active` pero `disabled` (funciona ahora, pero no volverá a arrancar solo al reiniciar).

---

## 5. ¿Qué pasa si MariaDB está parado?

Cuando WordPress muestre el error *«Error al establecer una conexión con la base de datos»*, lo primero que hay que mirar es si MariaDB está funcionando. Para ver cómo se nota, lo detengo a propósito:

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
```

Salida:

```text
inactive
ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)
```

**Explicación:** `is-active` responde `inactive` porque el servicio está parado. Al intentar entrar aparece el **ERROR 2002**, que significa *«no puedo conectar con MariaDB porque no está en marcha»*. El *socket* `/run/mysqld/mysqld.sock` es el «enchufe» por el que se comunican el cliente y el servidor; si el servidor está apagado, ese fichero no existe y no hay conexión.

Lo vuelvo a arrancar y compruebo que todo vuelve a la normalidad:

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

📸 **Capturas: error 2002 y recuperación (`active`)**

![Error 2002 y recuperación](img/mariadb-5.png)

**Interpretación:** en la captura se ve primero el ERROR 2002 al intentar entrar con el servicio parado, y después, tras `start`, que `is-active` vuelve a responder `active`.

> 📸 **PENDIENTE (recomendado):** la parte de arriba, el momento en que `systemctl is-active mariadb` responde `inactive` tras el `stop`.

### ❓ Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos de este apartado usarías para saber si el problema es que MariaDB está parado?

**Respuesta:**

1. `systemctl is-active mariadb`: responde `active` o `inactive` en una sola palabra. Es la forma más rápida de saberlo.
2. `sudo systemctl status mariadb`: da más detalle (estado, desde cuándo, y las últimas líneas del log con posibles errores).

Si salen `inactive`/`dead`, el problema es que MariaDB está parado y se soluciona con `sudo systemctl start mariadb`. Como comprobación adicional, `sudo mariadb`: si da el ERROR 2002, el servicio no está funcionando.

---

## 6. Instalación de PHP

WordPress está hecho en PHP, así que hay que instalarlo junto con las extensiones que necesita:

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
php -v
```

### Para qué sirve cada paquete

| Paquete | Función |
|---|---|
| `php` | El lenguaje PHP |
| `libapache2-mod-php` | Permite a **Apache ejecutar PHP** |
| `php-mysql` | Permite a PHP **hablar con MariaDB** (imprescindible) |
| `php-curl` | Conexiones a otros servidores (actualizaciones, plugins) |
| `php-gd` | Tratamiento de imágenes |
| `php-mbstring` | Manejo de textos con tildes y caracteres especiales |
| `php-xml` | Lectura y escritura de ficheros XML |
| `php-zip` | Instalación y actualización de plugins y temas |
| `php-intl` | Idiomas y formatos regionales |

📸 **Captura: `php -v`**

![Versión de PHP](img/mariadb-6.1.png)

Salida:

```text
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.6, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.6, Copyright (c), by Zend Technologies
```

**Explicación:** la versión instalada es **PHP 8.3.6**. `cli` significa que es la versión para la línea de comandos; `NTS` (*Non Thread Safe*) es la compilación normal usada con Apache en modo `prefork`. `OPcache` es una caché que acelera la ejecución de PHP.

### Reiniciar Apache y comprobar que carga PHP

```bash
sudo systemctl restart apache2
sudo apache2ctl -M | grep -E "mpm|php"
```

Salida esperada:

```text
 mpm_prefork_module (shared)
 php_module (shared)
```

**Explicación:** hay que **reiniciar** (no basta `reload`) para que Apache cargue el módulo nuevo. `apache2ctl -M` lista los módulos cargados y `grep` filtra los de `mpm` y `php`. Al instalar `libapache2-mod-php`, Ubuntu cambia Apache al modo `mpm_prefork`, que es el que necesita `mod_php`.

> 📸 **PENDIENTE: captura de la salida de `apache2ctl -M | grep -E "mpm|php"`** (módulos cargados). Guárdala como `img/mariadb-6.3.png`.

### Comprobar que PHP puede hablar con MariaDB

```bash
php -m | grep -i mysql
```

Salida esperada:

```text
mysqli
mysqlnd
pdo_mysql
```

**Explicación:** `php -m` lista los módulos de PHP instalados. `mysqli` y `pdo_mysql` son las dos formas que tiene PHP de conectarse a MySQL/MariaDB y `mysqlnd` es el driver nativo que usan. WordPress se conecta a su base de datos con estas piezas.

> 📸 **PENDIENTE: captura de `php -m | grep -i mysql`.** Guárdala como `img/mariadb-6.4.png`.

### Comprobar que Apache ejecuta PHP (`info.php`)

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

Abro en el navegador `http://192.168.56.17/info.php`:

📸 **Captura: página `info.php` en el navegador**

![Página info.php](img/mariadb-6.2.png)

**Explicación:** la función `phpinfo()` genera una página con toda la configuración de PHP. Que aparezca esa página (y no el código fuente) demuestra que **Apache le pasa los ficheros `.php` a PHP y este los ejecuta**. En la captura se ve `Apache/2.4.58 (Ubuntu)`, `User/Group www-data(33)/33` y, entre los módulos cargados, `mod_php` y `prefork`, que confirman que Apache ejecuta PHP.

### Borrar el fichero de prueba (obligatorio)

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```

Salida esperada: solo aparecen `index.html` (y `404.html` si lo creé en la práctica anterior); ya no está `info.php`.

> 📸 **PENDIENTE (opcional): captura de `ls /var/www/html`** sin `info.php`.

### ❓ ¿Por qué es peligroso dejar `info.php` en un servidor real?

**Respuesta:** porque la página `phpinfo()` revela información interna del servidor que un atacante puede aprovechar:

- La **versión exacta de PHP y de Apache** (puede buscar vulnerabilidades conocidas de esa versión).
- El sistema operativo, las **rutas** de los ficheros y la configuración.
- Los **módulos** y extensiones cargados.
- Variables de entorno y datos de configuración que pueden contener información sensible.

Por eso es un fichero que solo se usa para probar y se **borra siempre** después.

### ❓ ¿Para qué sirve el paquete `php-mysql`?

**Respuesta:** es la extensión que permite que **PHP se conecte y hable con MariaDB/MySQL** (instala `mysqli`, `mysqlnd` y `pdo_mysql`). Sin él, PHP funcionaría, pero WordPress no podría leer ni guardar datos en la base de datos y daría el error «Error al establecer una conexión con la base de datos». Por eso es el paquete imprescindible del conjunto.

---

## 7. Comprobación final

```bash
systemctl is-active apache2 mariadb
php -v
```

Salida esperada:

```text
active
active
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)
...
```

**Explicación:** `is-active` con dos servicios responde `active` una vez por cada uno (primero Apache, luego MariaDB). `php -v` confirma que PHP responde.

> 📸 **PENDIENTE: captura de `systemctl is-active apache2 mariadb` y `php -v`.** Guárdala como `img/mariadb-7.png`.

### Estado final del servidor

| Pieza del LAMP | Programa | Estado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado |
| Apache | Apache2 (2.4.58) | Funcionando |
| MariaDB | MariaDB 10.11.14 | Funcionando |
| PHP | PHP 8.3.6 | Funcionando y preparado para MariaDB |

### Snapshot de la máquina virtual

Con el servidor listo, hago una **instantánea (snapshot)** de la máquina virtual. Será mi punto de partida seguro para la práctica de WordPress: si algo sale mal, puedo volver a este estado en un minuto.

> 📸 **PENDIENTE (si lo pide el profesor): captura del snapshot** en el programa de virtualización.

---

## 8. Resumen de comandos (chuleta)

| Para qué | Comando |
|---|---|
| Dejar solo el sitio por defecto | `sudo a2dissite sitio1.conf sitio2.conf` y `sudo a2ensite 000-default.conf` |
| Recargar Apache | `sudo systemctl reload apache2` |
| Actualizar sistema | `sudo apt update` y `sudo apt upgrade -y` |
| Comprobar si Apache funciona | `systemctl is-active apache2` |
| Instalar MariaDB | `sudo apt install mariadb-server -y` |
| Versión de MariaDB | `mariadb --version` |
| Estado de MariaDB | `sudo systemctl status mariadb` |
| ¿Arranca solo? | `systemctl is-enabled mariadb` |
| Entrar en MariaDB | `sudo mariadb` (salir con `exit`) |
| Arrancar / parar / reiniciar | `sudo systemctl start` / `stop` / `restart mariadb` |
| Arranque automático | `sudo systemctl enable mariadb` |
| Instalar PHP y extensiones | `sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y` |
| Versión de PHP | `php -v` |
| Módulos de Apache (mpm/php) | `sudo apache2ctl -M \| grep -E "mpm\|php"` |
| Módulos de PHP para MySQL | `php -m \| grep -i mysql` |
| Crear / borrar fichero de prueba | `echo '<?php phpinfo();' \| sudo tee /var/www/html/info.php` / `sudo rm /var/www/html/info.php` |
| Comprobación final | `systemctl is-active apache2 mariadb` |

---

## 9. Conclusiones

Con esta práctica he completado el **LAMP** de mi servidor: ya tengo Linux (Ubuntu 24.04), Apache, MariaDB 10.11 y PHP 8.3 funcionando, todo listo para instalar WordPress. He aprendido para qué sirve cada pieza: Apache sirve la web, PHP ejecuta el código de WordPress y MariaDB guarda los datos.

También he aprendido a **diagnosticar** una caída de la base de datos: si MariaDB está parado, `systemctl is-active mariadb` responde `inactive` y al intentar entrar aparece el **ERROR 2002** (no hay socket porque el servicio no está en marcha). Se soluciona con `systemctl start mariadb`. Esto será lo primero que mire cuando WordPress muestre el error de conexión a la base de datos.

**Problemas encontrados y cómo los resolví:**

| Problema | Causa | Solución |
|---|---|---|
| El enunciado solo indicaba `a2ensite 000-default.conf` para «deshabilitar» los sitios virtuales | Activar el sitio por defecto no desactiva `sitio1` y `sitio2` | Añadí `a2dissite sitio1.conf sitio2.conf` antes de `a2ensite` |
| ERROR 2002 al ejecutar `sudo mariadb` | El servicio estaba parado (lo hice a propósito) | `sudo systemctl start mariadb` y comprobar con `is-active` |
| PHP no se ejecuta tras instalarlo (se vería el código o se descargaría el fichero) | Apache no ha cargado todavía el módulo nuevo | `sudo systemctl restart apache2` (hace falta reiniciar, no solo recargar) |
| `info.php` expone información del servidor | Fichero de prueba que muestra la configuración | Lo borré con `sudo rm /var/www/html/info.php` |

> ✏️ **Edita esta tabla** y deja solo los problemas que de verdad tuviste. Si no tuviste ninguno, dilo con tus palabras.
