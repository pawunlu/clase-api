# Modelo conectado a la base de datos

Vamos a modificar nuestra aplicación ahora para poder recuperar datos de la base de datos. Lo que haremos será lo siguiente:

* Generar un *seeder* para la tabla `movies` y ejecutarlo
* Generar a nivel del modelo métodos de consulta (individual y todos)
* Hacer que el *controller* hable con el modelo
* Generar la ruta adecuada

## Seeder de Movies

Un *seeder* básicamente es una clase que nos permite incorporar a la base de datos información, normalmente de prueba. Vamos a generar uno para la tabla `movies`:

```bash
vendor/bin/phinx seed:create MovieSeeder
```

Esto generará el archivo `db/seeds/MovieSeeder.php`. Editamos su contenido para que quede de la siguiente forma:

```php
use Phinx\Seed\AbstractSeed;

class MovieSeeder extends AbstractSeed
{
    public function run()
    {
        $faker = Faker\Factory::create();
        $data = [];
        for ($i=0; $i < 50; $i++) {
            $data[] = [
                'name' => $faker->words(5, true),
                'year' => $faker->year($max = 'now'),
                'director' => $faker->name(),
                'summary' => $faker->text(350)
            ];
        };
        $this->table('movies')->insert($data)->save();
    }
}
```

Ahora, ejecutamos el *seeder*:

```bash
vendor/bin/phinx seed:run -s MovieSeeder -e development
```

Observemos que en la tabla se han generado datos de prueba.

## Conexión del modelo con la base de datos

Aquí es necesario implementar los métodos de consulta haciendo uso de la instancia de PDO inyectada al modelo `Movie`.

```php
<?php
namespace App\Model;

class Movie
{
    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function find($id){
        $sentencia = $this->db->prepare("SELECT * FROM movies WHERE id = :id");
        $sentencia->execute(compact('id'));
        return $sentencia->fetch();
    }

    public function findAll(){
        $sentencia = $this->db->prepare("SELECT * FROM movies");    
        $sentencia->execute();    
        return $sentencia->fetchAll();        
    }
}
```

## Interacción del controlador con el modelo

El siguiente paso es agregar en el controlador los métodos que manejarán las peticiones tanto para recuperar una película como para el listado completo. Para ello agregamos los siguientes métodos en `MoviesController`

```php
    public function read($request, $response, $args){
        $movie = $this->model->find($args['id']);
        $response->getBody()->write(\json_encode($movie));
        return $response->withHeader('Content-Type', 'application/json');
    }

    public function index($request, $response, $args){
        $movies = $this->model->findAll();
        $response->getBody()->write(\json_encode($movies));
        return $response->withHeader('Content-Type', 'application/json');  
    }
```

## Generación de rutas

Creadas los métodos en el controlador y en el modelo, debemos agregar las rutas para estos dos recursos y conectarlas al controlador. Para ello, agregamos en el archivo `app/routes.php`:

```php
    $app->get('/movies', 'MoviesController:index');
    $app->get('/movies/{id}', 'MoviesController:read');
```

> Observemos que `{id}` indica que la ruta toma un argumento al cual llamamos `id` y accedemos al mismo en el método `MoviesController::read()` mediante el parámetro `$args`



[< Anterior](05-first-endpoint.md) | [Siguiente >](07-more-endpoints.md)