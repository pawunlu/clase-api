# Instalación del Entorno

Se considera que se dispone con un entorno con las siguientes tecnologías en su versión mas reciente:

* PHP 8
* Composer 2

Se mostraran los comandos de instalación como si se estuviera en un entorno Linux (Debian o Ubuntu). Adapte el lector al SO en el que se encuentre.

## Instalación de Librerías y Dependencias.

Las dependencias obligatorias para instalar son las librerías de XML y Mbstring de PHP.

```bash
sudo apt-get install php-xml php-mbstring php-mysql
```

Luego, creamos el esqueleto de la aplicación y algunas otras dependencias del proyecto.

```bash
composer create-project slim/slim-skeleton ~/workspace/slim-api --ignore-platform-reqs
cd ~/workspace/slim-api
composer require vlucas/phpdotenv robmorgan/phinx fzaninotto/faker --ignore-platform-reqs
```

Finalizado el proceso (que puede tardar un rato), ejecutamos el Servidor Web Stand-alone de PHP

```bash
php -S localhost:8000 -t public
```

Y probamos la instalación vía Browser a la URL http://localhost:8000 o por linea de comandos

```bash
curl http://localhost:8000
```

Deberíamos ver un `Hello word!`.

[< Anterior](01-theory.md) | [Siguiente >](03-db.md)

