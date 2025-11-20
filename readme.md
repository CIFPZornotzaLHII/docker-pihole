# Práctica: Desplegar y Configurar Pi-hole con Docker

![alt text](image.png)

- [Práctica: Desplegar y Configurar Pi-hole con Docker](#práctica-desplegar-y-configurar-pi-hole-con-docker)
  - [Objetivo](#objetivo)
  - [Requisitos previos](#requisitos-previos)
  - [1. Preparar el entorno](#1-preparar-el-entorno)
    - [Clonar o crear el directorio del proyecto](#clonar-o-crear-el-directorio-del-proyecto)
    - [Crear el fichero `docker-compose.yml`](#crear-el-fichero-docker-composeyml)
    - [Crear carpetas para persistencia](#crear-carpetas-para-persistencia)
  - [2. Levantar el contenedor Pi-hole](#2-levantar-el-contenedor-pi-hole)
  - [3. Configurar firewall / abrir puertos](#3-configurar-firewall--abrir-puertos)
  - [4. Acceder a la interfaz web](#4-acceder-a-la-interfaz-web)
  - [5. Configurar clientes Linux para usar Pi-hole como DNS](#5-configurar-clientes-linux-para-usar-pi-hole-como-dns)
    - [Paso 1: Identificar la IP del servidor Pi-hole](#paso-1-identificar-la-ip-del-servidor-pi-hole)
    - [Paso 2: Cambiar el DNS en la máquina cliente Linux](#paso-2-cambiar-el-dns-en-la-máquina-cliente-linux)
      - [Método persistente (modificar `/etc/resolv.conf`):](#método-persistente-modificar-etcresolvconf)
  - [6. Verificar que Pi-hole filtra publicidad](#6-verificar-que-pi-hole-filtra-publicidad)
    - [Prueba básica desde un cliente con DNS apuntando a Pi-hole](#prueba-básica-desde-un-cliente-con-dns-apuntando-a-pi-hole)
    - [Accede al panel web y revisa estadísticas](#accede-al-panel-web-y-revisa-estadísticas)
  - [7. Personalizaciones y tareas extras](#7-personalizaciones-y-tareas-extras)
  - [8. Apagar el servicio](#8-apagar-el-servicio)
  - [9. Actualizar Pi-hole](#9-actualizar-pi-hole)
  - [Consejos importantes](#consejos-importantes)
  - [Preguntas frecuentes](#preguntas-frecuentes)


---

## Objetivo

Aprender a instalar, configurar y utilizar Pi-hole como filtro DNS para bloquear publicidad y malware, usando Docker en una máquina Linux.
Además, realizar pruebas para comprobar que el filtro funciona correctamente.

---

## Requisitos previos

* Máquina Linux (local o en la nube) con Docker y Docker Compose instalados.
* Permisos para abrir puertos 53 (DNS) y 80 (HTTP) en el firewall.
* Conocimientos básicos de línea de comandos.

---

## 1. Preparar el entorno

### Clonar o crear el directorio del proyecto

```bash
mkdir pi-hole-practica
cd pi-hole-practica
```

### Crear el fichero `docker-compose.yml`

Crea un archivo llamado `docker-compose.yml` con este contenido:

```yaml
version: '3.7'

services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      TZ: 'America/Mendoza'          # Cambia a tu zona horaria
      WEBPASSWORD: 'tu_contraseña'  # Cambia la contraseña por seguridad
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
```

### Crear carpetas para persistencia

```bash
mkdir -p etc-pihole etc-dnsmasq.d
```

---

## 2. Levantar el contenedor Pi-hole

```bash
docker-compose up -d
```

Esto descargará la imagen de Pi-hole y la ejecutará.

---

## 3. Configurar firewall / abrir puertos

Si estás en un entorno cloud o tienes firewall activo, abre los puertos:

* UDP y TCP 53 (para DNS)
* TCP 80 (para la interfaz web)

Ejemplo con `ufw` (Ubuntu):

```bash
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw allow 80/tcp
sudo ufw reload
```

---

## 4. Acceder a la interfaz web

Abre un navegador y visita:

```
http://<IP_DE_TU_MAQUINA>
```

* Usuario: `admin` (por defecto)
* Contraseña: la que pusiste en `WEBPASSWORD` del docker-compose

Desde aquí podrás gestionar listas de bloqueo, dominios permitidos o bloqueados, y ver estadísticas.

---

## 5. Configurar clientes Linux para usar Pi-hole como DNS

### Paso 1: Identificar la IP del servidor Pi-hole

Por ejemplo, supongamos que la IP del servidor Pi-hole es `192.168.1.50`.

### Paso 2: Cambiar el DNS en la máquina cliente Linux

#### Método persistente (modificar `/etc/resolv.conf`):

- Edita `/etc/resolv.conf` con permisos de superusuario:

```bash
sudo nano /etc/resolv.conf
```

- Cambia o añade esta línea al inicio:

```
nameserver 192.168.1.50
```

- Guarda y cierra.

> Nota: Algunos sistemas gestionan `/etc/resolv.conf` automáticamente (NetworkManager, systemd-resolved). En ese caso, es mejor usar `nmcli` para no perder los cambios.


## 6. Verificar que Pi-hole filtra publicidad

### Prueba básica desde un cliente con DNS apuntando a Pi-hole

Abre terminal o consola y ejecuta:

```bash
nslookup doubleclick.net
```

* El resultado debe mostrar que la consulta es bloqueada o responde con `0.0.0.0` o IP localhost.
* También puedes visitar páginas web con anuncios para verificar que se bloquean.

### Accede al panel web y revisa estadísticas

* En la interfaz web verás las consultas DNS, dominios bloqueados, etc.

---

## 7. Personalizaciones y tareas extras

* Añadir listas de bloqueo extra en la interfaz.
* Bloquear dominios específicos manualmente.
* Revisar los logs de DNS para entender qué se está filtrando.
* Probar diferentes configuraciones DNS en `etc-dnsmasq.d`.

---

## 8. Apagar el servicio

```bash
docker-compose down
```

---

## 9. Actualizar Pi-hole

Para actualizar la imagen:

```bash
docker-compose pull
docker-compose up -d
```

---

## Consejos importantes

* Cambia siempre la contraseña del panel (`WEBPASSWORD`).
* No expongas puertos innecesarios al exterior si usas cloud.
* Usa listas de bloqueo confiables para evitar falsos positivos.
* Para resolver problemas, revisa logs:

```bash
docker logs pihole
```

---

## Preguntas frecuentes

**¿Por qué usar volúmenes?**
Para que la configuración y estadísticas se mantengan aunque detengas el contenedor.

**¿Qué pasa si no abro el puerto 53?**
No podrás usar Pi-hole como DNS.

**¿Puedo usar Pi-hole sin Docker?**
Sí, pero Docker facilita instalación y aislamiento.
