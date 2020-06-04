# Endpoints adicionales

En esta sección vamos a agregar algunos endpoints más a nuesta API.

## Alta de película

Como hemos visto en la [primer sección](01-theory.md) el alta de un recurso debe realizarse mediante el método HTTP `POST`. Dado que nuestra API utilizará el formato JSON para el intercambio de datos, esperamos que en el cuerpo de la petición los datos vengan codificados utilizando este formato. Esto implica que, para poder manipular los datos con mayor facilidad, debamos decodificar el cuerpo de la petición haciendo uso de la función `json_decode`. Para evitar tener que hacer esto nosotros, vamos a utilizar un [middleware](http://www.slimframework.com/docs/v4/middleware/body-parsing.html) (`BodyParsingMiddleware`) que nos resuelva esta cuestión. Un middleware es una porción de código que se ejecuta antes o después que nuestra aplicación procese la petición y genere una respuesta, teniendo la capacidad de manipular a ambas. Una aplicación puede tener más de un middleware, lo cual termina definiendo una estructura de capas tipo cebolla, donde en el centro se encuenta nuestra aplicación en sí. Tienen por objetivo resolver _cross cuttings concerns_, es decir, funcionalidades que son "transversales" a nuestra aplicación. Por ejemplo, la autenticación. Es común que las APIs requieran autenticación para poder consumirse. Por ende, esta es una funcionalidad que debe implementar nuestra aplicación, que a su vez, estará presente en una gran cantidad de endpoints. En nuestro caso, sucede algo similar con el parseo de los datos que vienen en la petición.

Entonces, para que nuestra API permite generar una nueva película debemos hacer lo siguiente:

1. Habilitar el middleware `BodyParsingMiddleware`. Para ello, agregamos en el archivo `index.php` la siguiente línea (de forma tal que quede antes de la llamada al método `addRoutingMiddleware();`):

  ```php
  $app->addBodyParsingMiddleware();
  ```

2. Generar un método en el modelo que permite almacenar una nueva película en la base de datos:

  ```php
    public function insert($movie){
        $sql = "INSERT INTO
                    movies
                    (name, year, director, summary)
                VALUES
                    (:name, :year, :director, :summary)";
        $sentencia = $this->db->prepare($sql);
        $sentencia->execute($movie);
        return $this->db->lastInsertId();        
    }
  ```

3. Agregar un método en el controlador que maneje las peticiones de insercción:

  ```php
    public function store($request, $response, $args)
    {
        $params = $request->getParsedBody();
        $id = $this->model->insert($params);
        $response->getBody()->write(\json_encode($params));
        return $response->withHeader('Content-Type', 'application/json');  
    }      
  ```

4. Generar la ruta y conectarla con el controlador. Notemos que aquí debemos utilizar el método HTTP `POST` sobre el recurso `/movies`:

  ```php
  $app->post('/movies', 'MoviesController:store');
  ```  

Estamos en condiciones ahora de probar nuestro nuevo endpoint. Para ello, utilizaremos el comando `curl`:

```bash
curl -X POST -H 'Content-Type: application/json' -i http://localhost:8000/movies --data '{
"name": "Hello world!",
"year": "2020",
"director": "Alan Turing",
"summary": "Hello world!"
}'
```

> Nótese que cuando generamos la peticón estamos utilizando el header `Content-Type: application/json`. Este permite al middleware `BodyParsingMiddleware` determinar la codificación de los datos y aplicar el mecanismo de decodificación correspondiente al mismo dado que soporta otros formatos de intercambio como, por ejemplo, `XML`.

## Baja de película

Te proponemos como actividad que extiendas la aplicación de forma tal que pueda darse de baja una película específica. Ten en cuenta las siguientes cuestiones:

* ¿qué método HTTP debe usarse?
* ¿cuál es la URL que identifica al recurso a elimnar?
* ¿qué cambios y en qué capas deben hacerse?

## Modificación de película

Adicionalmente, puedes modificar la aplicación de forma tal que una película sea modificable tanto garantizando como no la idempotencia de la acción.

[< Anterior](06-db-connect.md) | [Siguiente >](08-final.md)