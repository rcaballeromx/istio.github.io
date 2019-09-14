---
title: Ejecuta un service localmente
overview: Aprende a ejecutar un servicio localmente.
weight: 10
---


Antes de la arquitectura de micro-servicios, equipos de desarrollo compilaban,
desplegaban, y ejecutaban la aplicación completa como un pedazo enorme de
software. Para probar un cambio pequeño en su módulo que iba más allá de las
pruebas unitarias, los desarrolladores tenían que compilar la aplicación
completa. Por lo tanto, el construir una aplicación tomaba mucho tiempo. Luego
de compilar, los desarrolladores desplegaban su versión en un servidor de
pruebas. Ese servidor or corría en una máquina remota or en el sistema local
del desarrollador. En el último caso, desarrolladores se veían forzados a
mantener y operar un ambiente complejo en sus sistemas locales.

En la era de la arquitectura de micro-servicios, desarrolladores escriben,
compilan, prueban y ejecutan pequeños servicios de software. Compilaciones son
rápidas. En frameworks modernos como [Node.js](https://nodejs.org/en/) no hay
necesidad de instalar y operar servidores con ambientes complejos para probar
un servicio. Tú no tienes que desplegar tu servicio en algún ambiente solo para
probarlo, y por lo tanto puedes compilar y ejecutar el servicio inmediatamente
en tu sistema local.

Este módulo cubre los diferentes aspectos involucrados en el desarrollo de un servicio en un sistema local.

¡No necesitas saber escribir código! En su lugar, compilarás, ejecutarás y probarás un servicio existente:
`ratings`.

El servicio `ratings` es una aplicación web pequeña escrita en
[Node.js](https://nodejs.org/en/) que puede ejecutar por sí sola. Esta
aplicación realiza acciones similares a otras aplicaciones web:

- Escucha a un puerto que recibe como parámetro.
- Espera llamadas `HTTP GET`
  en la ruta `/ratings/{productID}` y regresa las calificaciones del producto tenga
  el valor que especifiques para `productID`.

- Espera llamadas `HTTP POST` en la ruta `/ratings/{productID}` y actualiza las
  calificaciones del producto que tenga el valor que especifiques para
  `productID`.

Sigue estos pasos para descargar el código de la aplicación, instalar sus
dependencias y ejecutarla localmente:

1. Descarga el
    [código del servicio]({{< github_blob >}}/samples/bookinfo/src/ratings/ratings.js)
    y
    [el archivo de paqueete]({{< github_blob >}}/samples/bookinfo/src/ratings/package.json)
    a un directorio aparte:

    {{< text bash >}}
    $ mkdir ratings
    $ cd ratings
    $ curl -s {{< github_file >}}/samples/bookinfo/src/ratings/ratings.js -o ratings.js
    $ curl -s {{< github_file >}}/samples/bookinfo/src/ratings/package.json -o package.json
    {{< /text >}}

1. Ve el código del servicio e identifica los siguientes elementos:

    - Los features del servidor web:
        - Cómo escucha al puerto
        - Qué hace con las llamadas y las respuestas
    - Los aspectos relacionados con HTTP:
        - headers
        - rutas
        - códigos de estado

    {{< tip >}}
    En Node.js, la funcionalidad del servidor web está dentro del código de la aplicación. Una web app en Node.js
    se ejecuta como un proceso independiente.
    {{< /tip >}}

1. Aplicaciones en Node.js están hechas con JavaScript. Por esto, no hay un paso de
   compilación explícito. En su lugar, se usa la [compilación
   justo-a-tiempo](https://en.wikipedia.org/wiki/Just-in-time_compilation).
   Compilar una aplicación Node.js, significa instalar sus dependencias. Instala
   las dependencias del servico `ratings` en la misma carpeta donde pusiste el
   código del servicio y el archivo de paquete:

    {{< text bash >}}
    $ npm install
    npm notice created a lockfile as package-lock.json. You should commit this file.
    npm WARN ratings No description
    npm WARN ratings No repository field.
    npm WARN ratings No license field.

    added 24 packages in 2.094s
    {{< /text >}}

1. Ejecuta el servicio y mándale el puerto `9080` como parámetro y el servicio
   entonce escucha en el puerto 9080.

    {{< text bash >}}
    $ npm start 9080
    > @ start /tmp/ratings
    > node ratings.js "9080"
    Server listening on: http://0.0.0.0:9080
    {{< /text >}}

    {{< tip >}}
    El servicio `ratings` es una web app y puedes comunicarte con ella como con cualquier otra. Puedes usar el navegador o la linea de comandos con un cliente web como [cURL](https://curl.haxx.se) o [wget](https://www.gnu.org/software/wget/). Como estas ejecutando el servicio `ratings` localmente, puedes accederlo con el hostname `localhost`.
    {{< /tip >}}

1. Abre [http://localhost:9080/ratings/7](http://localhost:9080/ratings/7) en to navegador o usa `curl`:

    {{< text bash >}}
    $ curl localhost:9080/ratings/7
    {"id":7,"ratings":{"Reviewer1":5,"Reviewer2":4}}
    {{< /text >}}

1. Usa el método `POST` de `curl` para cambiar la calificación del producto a `1`:

    {{< text bash >}}
    $ curl -X POST localhost:9080/ratings/7 -d '{"Reviewer1":1,"Reviewer2":1}'
    {"id":7,"ratings":{"Reviewer1":1,"Reviewer2":1}}
    {{< /text >}}

1. Verifica la calificación cambiada:

    {{< text bash >}}
    $ curl localhost:9080/ratings/7
    {"id":7,"ratings":{"Reviewer1":1,"Reviewer2":1}}
    {{< /text >}}

1. Usa `Ctrl-C` para detener el servicio.

Éxito!

Ahora puedes compilar, ejecutar y probar un servicio en tu sistema local.
