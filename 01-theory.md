# Introducción al diseño e implementación de APIs REST

REST (Representational State Transfer) o Transferencia de Estado Representacional es un enfoque arquitectónico para el diseño de servicios web y, más en general, sistemas distribuidos. Se origina en una [tesis doctoral](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) escrita por Roy Thomas Fielding en el año 2000.

Partiendo de que, cualquier API debe cumplir con al menos los siguientes requisitos:

* ser independiente de un plataforma
* debe ser capaz de evolucionar en el tiempo, sin por ello, perder compatibilidad con sus consumidores.

una API REST resuelve esteb problema a través de una serie de principios de diseño:

* existe una series de **recursos** que pueden ser accedidos por el **cliente**;
* un recurso tiene un **identificador** que identifica unívocamente a un recurso;
* los clientes interactúan con el serivicio mediante el intercambio de **representaciones** del recurso;
* se utiliza una **interfaz uniforme** que desacopla las implementaciones consumidoras de las que implementan el servicio;
* se utiliza un modelo **sin estado** de peticiones independientes y atómicas;
* deben estar basadas en **hipermedia**

Si observamos los requisitos, podemos inferir que sobre HTTP podemos implementar una API que cumpla con los principios enunciados anteriormente dado que:

* HTTP permite el acceso a recursos;
* los identifica mediante una URI;
* nuestros recursos pueden ser documentos JSON, XML, imágenes, etc.
* la interfaz uniforme va a estar dada por los métodos HTTP, los diferentes códigos de respuesta, encabezados, etc.
* HTTP es un protocolo sin estado;
* nuestros recursos pueden contener links a otros recursos asociados.

Supongamos que tenemos un sistema que gestiona una base de películas y queremos que el mismo pueda ser consumido por terceros. Algunos recursos posibles del mismo serían:

* películas
* directores
* actores

Así una película podría estar identificada mediante la siguiente URL: `https://mi-app.com/peliculas/1`. Así, dicha película podría estar representada por el siguiente documento JSON:

```json
{
    "name": "Pulp Fiction",    
    "year": 1994,
    "summary": "La vida de un boxeador, dos sicarios, la esposa de un gánster y dos bandidos se entrelaza en una historia de violencia y redención.",
    "links": [
        { "rel":"director", "href":"https://mi-app.com/directores/3", "action":"GET" },
        { "rel":"actor", "href":"https://mi-app.com/actores/102", "action":"GET" },
    ]
}
```

En consecuencia, si quisiéramos recuperar dicha película desde otra aplicación, la misma deberá implementar un cliente HTTP, que realice una petición `GET` sobre `https://mi-app.com/peliculas/1`. En cambio, si quisiéramos eliminar dicha película podríamos hacer una petición usando el método `DELETE`. El resultado de nuestra operación estaría dado por la respuesta HTTP. Por ejemplo, si obtenemos el código de respuesta `200`, en el cuerpo de la petición tendremos el recurso solicitado. Obsérvese además, que nuestra respuesta contiene un link a la información del director, con lo cual nuestra aplicación podría obtener la información del mismo, simplemente haciendo una petición `GET` a dicha URL.

## Lineamientos para modelar una API REST

### 1 - Definición de recursos

Si quisiéramos implementar una API REST lo primero que deberíamos hacer es identificar los recursos de nuestro negocio que queremos que sean accesibles a través de la misma. Es importante destacar que estamos pensando en recursos y no en operaciones. Además, que dichos recursos no necesariamente se corresponden con las tablas de la base de datos de nuestra aplicación, si la misma tuviese. Por ejemplo, si nuestra aplicación tiene una tabla de clientes y otra con teléfonos de clientes, desde el punto de vista de la API podemos modelar un único recurso "cliente" que contenga no solo la información del cliente, sino también, los teléfonos correspondientes al mismo.

Normalmente, nuestra aplicación tiene colecciones de recursos las cuales desde el punto de vista RESTful son un recurso. Siguiendo con el ejemplo de la aplicación de películas, una colección se corresponde a los directores. Así la misma podría esta identificada mediante la URL `https://mi-app.com/directores` mientras que un director en particular por `https://mi-app.com/directores/1`. De lo anterior, se deduce los siguiente:

* normalmente usamos sustantivos plurales para identificar la collección;
* aprovechamos la jerarquía que nos permiten expresar las URLs para identificar un recurso particular perteneciente a la colección.

Respecto al último punto, y siguiendo con el ejemplo de las películas, si quisiéramos modelar todos los actores pertenecientes a una película en particular, podríamos identificar dicho recurso con la siguiente URL:

```
https://mi-app.com/peliculas/1/actores
```

Es importante destacar que no conviene abusar tampoco de esta cuestión, dado que puede complicar bastante la implementación tanto del cliente como del servicio. Una alternativa a esto, es el uso de links dentro del recurso a los recursos asociados, como se ve en el JSON de ejemplo de la película "Pulp Fiction".

### 2 - Definición de operaciones

Queda claro que al implementar REST sobre HTTP nuestras operaciones estarán dadas por los métodos HTTP. En general, podemos usar la siguiente tabla para definir qué método se adecua mejor a una operación determinada de nuestra aplicación:

| Método   | Uso                                                                                          | Idempotente |
|:--------:|----------------------------------------------------------------------------------------------|:-----------:|
| `GET`    | Para obtener un recurso                                                                      | Si          |
| `POST`   | Crear un recurso                                                                             | No          |
| `PUT`    | Actualizar un recurso especificando su estado completo                                       | Si (hay implementaciones que permiten crear recursos con este método, en tal caso, se pierda esta propiedad) |
| `DELETE` | Eliminar un recurso                                                                          | No          |
| `PATCH`  | Actualizar un recurso especificando solo una parte de su estado mediante un documento patch. | No          |

> Un documento patch es básicamente un documento que especifican qué debe actualizaciones y dónde en un documento destino sobre el cal será aplicado

Cabe aclarar que no necesariamente necesitamos soportar todos los métodos. Por ejemplo, si no queremos dar la posibilidad de eliminar películas, no permitiremos peticiones `DELETE` sobre `https://mi-app.com/peliculas/{id}`.

Algunos ejemplos de aplicación sobre nuestro caso

| Recurso               | `GET`                                         | `POST`                                         | `PUT`                                                                              | `PATCH`                                                                                      | `DELETE`                                            |
|-----------------------|-----------------------------------------------|------------------------------------------------|------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|-----------------------------------------------------|
| `peliculas`           | Obtiene el listado de películas disponibles   | Crea una nueva película                        | Actualización en masa de todas las películas                                       | Actualización parcial y en masa de todas las películas                                       | Elimina todas las películas                         |
| `peliculas/1`         | Obtiene la película con ID = 1                | No soportado                                   | Actualización de la película con ID = 1                                            | Actualización parcial de la película con ID = 1                                              | Elimina la película con ID = 1                      |
| `peliculas/1/actores` | Obtiene los actores de la película con ID = 1 | Agrega un nuevo actor a la película con ID = 1 | Actualización en masa de todos los actores pertenecientes a la película con ID = 1 | Actualización parcial y en masa de todos los actores pertenecientes a la película con ID = 1 | Elimina todos los actores de la película con ID = 1 |

Respecto a los códigos de estado a utilizar en las respuesta, cabe destacar lo siguiente:

* con respecto a `GET`:
  * si el recurso fue encontrado, se devuelve el código `200`;
  * si el recurso no fue encontrado, se devuelve `404`
* con respecto a `POST`:
  * si el recurso fue creado se devuelve código `201` y se utiliza el header `Location` con la URL al recurso generdo
  * si el cliente cometió un error (por ejemplo, no especificó el título para una película, siendo este requerido), se retorna el código `400`
* con respecto a `PUT`:
  * si el recurso fue creado, se devuelve el código `201`
  * si se actualiza un recurso existente `200` o `204`
  * si por algún motivo el recurso no puede ser actualizado `409`.
* con respecto a `PATCH`:
  * `204` si el patch fue aplicado exitosamente
  * `400` si el patch está mal formado
* con respecto a `DELETE`:
  * `204` si el recurso fue exitosamente eliminado
  * `404`  si el recurso a eliminar no fue ncontrado

Hasta el momento no se ha mencionado, pero en la URL podemos especificar paráemteros y modificar el comportamiento de nuestra API en función de ellos. Por ejemplo, para el caso de las colecciones de recursos, podemos usarlos para implementar paginación de forma tal de evitar recuperar gran cantidad de datos. Siguiendo con el ejemplo de las películas:

```
https://mi-app.com/peliculas?limit=10&offset=2
```

En este caso, recuperamos de a 10 películas por petición, obteniendo la segunda página. Otras aplicaciones podrían ser el ordenamiento o la recuperación de registros que cumplan con ciertos criterios. Por ejemplo:

```
https://mi-app.com/peliculas?year=1980
```

Lo cual recuperaría todas las películas del año 1980.

## Bibliografía

* Fielding, R. T., & Taylor, R. N. (2000). Architectural styles and the design of network-based software architectures (Vol. 7). Irvine: University of California, Irvine.
* Web API Design. Microsoft Docs - Última consulta: 22/05/2020. https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design.
* Webber, J., Parastatidis, S., & Robinson, I. (2010). REST in practice: Hypermedia and systems architecture. " O'Reilly Media, Inc.".