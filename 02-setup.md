# Instalación del Entorno

Se considera que se dispone con un entorno con las siguientes tecnologías en su versión mas reciente:

* PHP
* Composer

Se mostraran los comandos de instalación como si se estuviera en un entorno Linux (Debian o Ubuntu). Adapte el lector al SO en el que se encuentre.

## Instalación de Librerías y Dependencias.

La única dependencia obligatoria para instalar es la librería de XML de PHP.

```bash
sudo apt-get install php-xml
```

Luego, creamos el esqueleto de la aplicación y algunas otras dependencias del proyecto.

```bash
composer create-project slim/slim-skeleton ~/workspace/slim-api
composer require vlucas/phpdotenv robmorgan/phinx fzaninotto/faker
```

Finalizado el proceso, ejecutamos el Servidor Web Stand-alone de PHP

```bash
cd ~/workspace/slim-api
php -S localhost:8000 -t public
```

Y probamos la instalación

```bash
curl http://localhost:8000
```

Deberíamos ver un `Hello word!`.

[< Anterior](01-theory.md) | [Siguiente >](03-db.md)

