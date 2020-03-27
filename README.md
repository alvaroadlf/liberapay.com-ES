# Traducción al castellano del repositorio de Liberapay

> Esto es solo una traducíon del repositorio de Liberpay y no forma
> parte de su código, se trata solamente de ayudar a entender el
> funcionamiento de Liberapay a personas que no hablen ingles o
> prefieran por comodidad leerlo en español.
> 
> Fuente original > [README.MD de
> Liberapay](https://github.com/liberapay/liberapay.com/blob/master/README.md)



# Liberapay

[![Build Status](https://travis-ci.org/liberapay/liberapay.com.svg?branch=master)](https://travis-ci.org/liberapay/liberapay.com)
[![Weblate](https://hosted.weblate.org/widgets/liberapay/-/shields-badge.svg)](https://hosted.weblate.org/engage/liberapay/?utm_source=widget)
[![Open Source Helpers](https://www.codetriage.com/liberapay/liberapay.com/badges/users.svg)](https://www.codetriage.com/liberapay/liberapay.com)
[![Gitter](https://badges.gitter.im/liberapay/salon.svg)](https://gitter.im/liberapay/salon?utm_source=badge)
[![Income](https://img.shields.io/liberapay/receives/Liberapay.svg)](https://liberapay.com/Liberapay)
[![Donate](https://liberapay.com/assets/widgets/donate.svg)](https://liberapay.com/liberapay/donate)

[Liberapay](http://liberapay.com) es una plataforma de donaciones recurrentes. Ayudamos a financiar a los creadores y a los proyectos que aprecias.

Nota: Esta webapp no se puede ejecutar por cuenta.

## Tabla de Contenidos

- [Contacto](#contacto)
- [Contribuyendo a las traducción](#contribuyendo-a-las-traducción)
- [Contribuyendo al código](#contribuyendo-al-código)
  - [Introducción](#introducción)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Ejecutando](#ejecutando)
    - [Día de pago](#día-de-pago)
  - [SQL](#sql)
  - [CSS y JavaScript](#css-y-javascript)
  - [Probando](#probando)
    - [Updating test fixtures](#updating-test-fixtures)
    - [Speeding up the tests](#speeding-up-the-tests)
  - [Tinkering with payments](#tinkering-with-payments)
  - [Modificando las dependencias de python](#modificando-las-dependencias-de-python)
  - [Procesando datos personales](#procesando-datos-personales)
  - [Desplegando la app](#desplegando-la-app)
  - [Configurar un entorno de desarrollo utilizando Docker](#configurar-un-entorno-de-desarrollo-utilizando-docker)
- [Licencia](#licencia)
- [Créditos de la traducción](#créditos-de-la-traducción)

## Contacto

¿Quieres hablar? [Únete a nosotros en Gitter](https://gitter.im/liberapay/salon). (Si usas IRC, [Gitter tiene una gateway](https://irc.gitter.im/), y también estamos en el canal #liberapay en Freenode.)

Alternativamente, puedes publicar un mensaje en [nuestro salón en GitHub](https://github.com/liberapay/salon).


## Contribuyendo a la traducción

Puedes ayudar a traducir Liberapay [vía Weblate](https://hosted.weblate.org/engage/liberapay/). Estado actual:

[![global translation status](https://hosted.weblate.org/widgets/liberapay/-/287x66-white.png)](https://hosted.weblate.org/engage/liberapay/?utm_source=widget)

[![translation status by language](https://hosted.weblate.org/widgets/liberapay/-/multi-auto.svg)](https://hosted.weblate.org/projects/liberapay/core/?utm_source=widget)

Si tiene preguntas sobre la traducción de Liberapay, puedes preguntar [en el salón](https://github.com/liberapay/salon/labels/i18n).


## Contribuyendo al código

### Introducción

Liberapay originalmente fue un fork de [Gratipay](https://github.com/gratipay/gratipay.com) y heredó su micro-framework web [Pando](https://github.com/AspenWeb/pando.py) (*né* Aspen), que se basa en el enrutamiento del sistema de archivos y [simplates](http://simplates.org/). No te preocupes, es bastante simple. Por ejemplo, para hacer que Liberapay devuelva un `Hola $user, tu id es $userid` para las peticiones desde la URL `/$user/hello`, solo tienes que crear el archivo `www/%username/hello.spt` con el siguiente contenido:

```
from liberapay.utils import get_participant
[---]
participant = get_participant(state)
[---] text/html
{{ _("Hello {0}, your id is {1}", request.path['username'], participant.id) }}
```

Como muestra la última línea, nuestro motor de plantillas predeterminado es [Jinja](http://jinja.pocoo.org/).

La función `_` intenta traducir el mensaje al idioma del usuario y escapa de las variables correctamente (sabe que está generando un mensaje para una página HTML).

El código de puthon dentro de simplates es sólo para la lógica de petición-específica, el código común del backend se encuentra en el directorio `liberapay/`.

### Instalación

Asegúrate de tener las siguientes dependencias instaladas primero:

- python ≥ 3.6
   - incluidos los headers C de python y libffi, que van por separado en muchas distribuciones de Linux
- postgresql 9.6 (ver [los documentos oficiales de descarga e instalación](https://www.postgresql.org/download/))
- make

A continuación, ejecuta:

    make env

Ahora debes otorgar poderes de superusuario de postgres (si aún no se ha hecho) y crear dos bases de datos:

    su postgres -c "createuser --superuser $(whoami)"

    createdb liberapay
    createdb liberapay_tests

Si necesitas entenderlo mejor, echa un vistazo a las secciones [Roles de la Base de Datos](https://www.postgresql.org/docs/9.4/static/user-manag.html) y a [Gestionando Bases de Datos](https://www.postgresql.org/docs/9.4/static/managing-databases.html) de la documentación de PostgreSQL's..

Entonces puedes configurar la DB:

    make schema

### Configuración

Las variables de entorno se utilizan para la configuración, los valores por defecto están en
`defaults.env` y en `tests/test.env`. Puedes sobreescribirlos en
`local.env` y en `tests/local.env` respectivamente.

### Ejecutando

Una vez que hayas instalado todo y configurado la base de datos, puedes ejecutar la aplicación:

    make run

Debería ser accesible desde [http://localhost:8339/](http://localhost:8339/).

No hay usuarios proporcionados por defecto. Puedes crear cuentas como lo harías en el la web real, y si lo deseas, también puedes crear un grupo de usuarios falsos (pero no son geniales):

    make data

Para otorgar permisos de administrador a una cuenta, modifica la base de datos de la siguiente manera:

    psql liberapay -c "update participants set privileges = 1 where username = 'account-username'"

#### Día de pago

Para ejecutar un día de pago local abre [http://localhost:8339/admin/payday](http://localhost:8339/admin/payday) y haz clic en el botón "Run payday". Puedes añadir `OVERRIDE_PAYDAY_CHECKS=yes` en el archivo `local.env` para desactivar las comprobaciones de seguridad que impiden que se ejecute el día de pago en el momento equivocado.

### SQL

El código de Python interactúa con la base de datos enviando consultas SQL sin procesar a través de la librería [postgres.py](https://postgres-py.readthedocs.org/en/latest/).

La [documentación oficial de PostgreSQL](https://www.postgresql.org/docs/9.6/static/index.html) es tu amiga cuando se trata de SQL, especialmente las secciones "[El Idioma SQL](https://www.postgresql.org/docs/9.6/static/sql.html)" y "[Comandos SQL](https://www.postgresql.org/docs/9.6/static/sql-commands.html)".

La estructura de la DB se encuentra en `sql/schema.sql`, pero no modifiques directamente el archivo,
en su lugar haz los cambios en `sql/branch.sql`. Durante el despliegue, ese script se ejecutará en la base de datos de producción y los cambios se fusionarán en el archivo `sql/schema.sql`.
Ese proceso está semiautomatizado por `release.sh`.

### CSS y JavaScript

For our styles we use [SASS](http://sass-lang.com/) and [Bootstrap 3](https://getbootstrap.com/). Stylesheets are in the `style/` directory and our JavaScript code is in `js/`. Our policy for both is to include as little as possible of them: the website should be almost entirely usable without JS, and our CSS should leverage Bootstrap as much as possible instead of containing lots of custom rules that would become a burden to maintain.

We compile Bootstrap ourselves from the SASS source in the `style/bootstrap/`
directory. We do that to be able to easily customize it by changing values in
`style/variables.scss`. Modifying the files in `style/bootstrap/` is probably
not a good idea.

### Probando

The easiest way to run the test suite is:

    make test

This recreates the test DB's schema and runs all the tests. To speed things up
you can also use the following commands:

- `make pytest` only runs the python tests without recreating the test DB
- `make pytest-re` only runs the tests that failed previously

#### Updating test fixtures

Some of our tests include interactions with external services. In order to speed up those tests we record the requests and responses automatically using [vcr](https://pypi.python.org/pypi/vcrpy). The records are in the `tests/py/fixtures` directory, one per test class.

If you add or modify interactions with external services, then the tests will fail, because VCR will not find the new or modified request in the records, and will refuse to record the new request by default (see [Record Modes](https://vcrpy.readthedocs.io/en/latest/usage.html#record-modes) for more information). When that happens you can either switch the record mode from `once` to `new_episodes` (in `liberapay/testing/vcr.py`) or delete the obsolete fixture files.

If the new interactions are with MangoPay you have to delete the file `tests/py/fixtures/MangopayOAuth.yml`, otherwise you'll be using an expired authentication token and the requests will be rejected.

#### Speeding up the tests

PostgreSQL is designed to prevent data loss, so it does a lot of synchronous disk writes by default. To reduce the number of those blocking writes, our `recreate-schema.sh` script automatically switches the `synchronous_commit` option to `off` for the test database, however this doesn't completely disable syncing. If your PostgreSQL instance only contains data that you can afford to lose, then you can speed things up further by setting `fsync` to `off` in the server's configuration file (`postgresql.conf`).

### Tinkering with payments

Liberapay was built on top of [MangoPay](https://www.mangopay.com/) for payments, however they [kicked us out](https://medium.com/liberapay-blog/liberapay-is-in-trouble-b58b40714d82) so we've shifted to integrating with multiple payment processors. We currently support [Stripe](https://stripe.com/docs) and [PayPal](https://developer.paypal.com/docs/). However, support for Mangopay hasn't been completely removed yet.

### Modificando las dependencias de python
All new dependencies need to be audited to check that they don't contain malicious code or security vulnerabilities.

We use [pip's Hash-Checking Mode](https://pip.pypa.io/en/stable/reference/pip_install/#hash-checking-mode) to protect ourselves from dependency tampering. Thus, when adding or upgrading a dependency the new hashes need to be computed and put in the requirements file. For that you can use [hashin](https://github.com/peterbe/hashin):

    pip install hashin
    hashin package==x.y -r requirements_base.txt -p 3.6 -p 3.7
    # note: we have several requirements files, use the right one

If for some reason you need to rehash all requirements, run `make rehash-requirements`.

To upgrade all the dependencies in a requirements file, run `hashin -u -r requirements_XXX.txt -p 3.6 -p 3.7`. You may have to run extra `hashin` commands if new subdependencies are missing.

### Procesando datos personales

When writing code that handles personal information, keep in mind the principles enshrined in the [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation).

### Desplegando la app

Note: Liberapay cannot be self-hosted, this section is only meant to document how we deploy new versions.

Liberapay is currently hosted on [AWS](https://aws.amazon.com/) (Ireland).

To deploy the app simply run `release.sh`, it'll guide you through it. Of course you need to be given access first.

### Configurar un entorno de desarrollo utilizando Docker

If you don't want to install the dependencies directly on your machine, you can spin up a development environment easily, assuming you have [Docker](https://docs.docker.com/engine/installation/) and [docker-compose](https://docs.docker.com/compose/install/) installed:

    # build the local container
    docker-compose build

    # initialize the database
    docker-compose run web bash recreate-schema.sh

    # populate the database with fake data
    docker-compose run web python -m liberapay.utils.fake_data

    # launch the database and the web server
    # the application should be available on http://localhost:8339
    docker-compose up

You can also run tests in the Docker environment:

    docker-compose -f docker/tests.yml run tests

All arguments are passed to the underlying `py.test` command, so you can use `-x` for failing fast or `--ff` to retry failed tests first:

    docker-compose -f docker/tests.yml run tests -x --ff

## Licencia

[CC0 Dedicación de Dominio Público](https://creativecommons.org/publicdomain/zero/1.0/) (Haz [clic aquí](https://github.com/liberapay/liberapay.com/issues/564) para más detalles.)


## Créditos de la taducción

Traducid por [Álvaro Araoz](https://imalvaro.com).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEyMDExNTAwOCwtODE1MzYwNzY2XX0=
-->