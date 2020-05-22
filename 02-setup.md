# Setup de Slim

Lo primero que necesitamos es instalar la extensión de `php-xml`.

```bash
sudo apt-get install php-xml
```

Luego, creamos el esqueleto de la aplicación y algunas otras dependencias del proyecto.

```bash
composer create-project slim/slim-skeleton ~/workspace/slim-api
composer require vlucas/phpdotenv robmorgan/phinx fzaninotto/faker
```

Finalizado el proceso, ejecutamos el webserver de PHP

```bash
cd ~/workspace/slim-api
php -S localhost:8000 -t public
```

Y probamos la instalación

```bash
curl http://localhost:8000
```

Deberíamos ver un `Hello word!`.