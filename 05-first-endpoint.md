# Primer recurso

Creamos el archivo `app/controllers/MovieController.php` donde pondremos el controlador del primer recurso. En este caso modelaremos `Movie` (tanto controlador como modelo).

```php
<?php

namespace Api\Controller;

use Api\Model\Movie;

class MovieController {
    function __construct(Movie $model, \Slim\Container $container) {
        $this->model = $model;
        $this->container = $container;
    }

    function test($request, $response, $args) {
        $movies = [
            '1' => [
                'name' => 'Pulp Fiction',
                'director' => 'Quentin Tarantino'
            ],
            '2' => [
                'name' => 'Rear Window',
                'director' => 'Alfred Hitchcock'
            ]
        ];
        return $response->withJson($movies);
    }
}
```

Ademas creamos `app/models/Movie.php`

```php
<?php

namespace App\Model;

/**
 *
 */
class Movie
{
    function __construct(\PDO $db)
    {
        $this->db = $db;
    }
}

```

Y agregamos el controlador al inyector de dependencias en `src/dependencies.php`:

```php
$container['MovieController'] = function ($c) {
    $model = new \Api\Model\Movie($c['db']);
    return new \Api\Controller\MovieController($model, $c);
};
```

Y agregamos la carga de las clases al autoload en composer.json. Autoload es una característica de los proyectos donde todos los archivos son cargados de forma automática al inicio y evita la necesidad de tener dispersos las llamadas a los archivos (nos ahorra el tener que hacer `include` o `require`). Se puede hacer de forma manual, o mejor, dejar que composer maneje esta propiedad mediante el siguiente código en el archivo `composer.json`.

```json
"autoload": {
    "psr-4": {
        "App\\Controller\\": "app/controllers",
        "App\\Model\\": "app/models"
    }
},
```

Luego ejecutamos `composer dump-autoload `. Si quieres saber mas sobre **autoload**, dejamos algunas referencias[^1][^2].

Ademas hay que crear la ruta (path) para acceder al recurso. Esto se hace en en `src/routes.php`:

```php
$app->get('/movies/test', 'MovieController:test');
```

A continuación corremos el server:

```bash
php -S localhost:8000 -t public
```

Si nos dirigimos a la ruta, vemos que Slim nos muestra la salida del metodo test en http://localhost:8000/movies/test

## Referencias

[^1]: https://www.php.net/autoload
[^2]: https://www.php-fig.org/psr/psr-4/



[< Anterior](04-git.md) | [Siguiente >](06-XXX.md)