---
title: Web scraping rápido con Symfony Panther
categories:
- Symfony
- Panther
tags:
    - PHP
    - Panther
    - scraping
    - BrowserKit
    - DomCrawler
thumbnail: /images/panther/panther_logo.png
date: 2020-05-25
---

{% img /images/panther/web_scraping_panther.png %}

Durante este confinamiento, que en mi caso ha sido con hijos en edad escolar, han ido aflorando muchas carencias tanto en contenidos digitales como en herramientas de comunicación por parte de la enseñanza pública. 

En concreto, durante el inicio del confinamiento muchos alumnos hicieron uso de servicios online ofrecidos por editoriales (al menos de aquellas que dieron acceso), ya que muchos dejaron sus libros físicamente en el colegio.

La mayor parte de las editoriales no ofrecen la posibilidad de descarga directa de los contenidos, y la impresión directa en algunos casos no es tan directa como cabe esperar.

Mi hija más pequeña tiene 8 años y utiliza libros de fichas para completar.

Durante varios días muchos de los contenidos no estuvieron disponibles por lentitud, interrupciones e incidencias varias, mientras las editoriales (*supongo*) trataban de escalar los servicios.

Entendía el tema de la lentitud, al fin y al cabo estábamos en una situación excepcional, pero una vez revisado el código de las webs comprobé que los servicios estaban claramente orientados a impedir, o al menos a dificultar, la reproducción y/o impresión de los contenidos: **Login desde javascript, renderización de contenidos en Canvas, menús gráficos, mala semántica, navegación compleja, etc.**
 
Mi paciencia acabó por consumirse después de varios días y de ahí surgió la idea este post: Recuperar los libros completos desde las páginas de un par de editoriales.

<!-- more -->

# El protocolo WebDriver (W3C) y Symfony Panther

Reconozco que hacía tiempo que no hacía *web scraping*, porque no había tenido la necesidad y el recuerdo no era bueno con herramientas tipo *CasperJS*. Cuando por mi cabeza ya pasaba hacer uso de `pip install scrapy` 😅  decidí darle una oportunidad a [Symfony Panther](https://github.com/symfony/panther) que ya había utilizado para *testing*.

## La especificación WebDriver

 Básicamente se trata de una interfaz de control remoto para introspección y control de navegadores. La especificación está inspirada y es compatible con en el famoso framework de automatización [**Selenium**](https://www.selenium.dev/).

Si no conoces la especificación puedes consultarla [aquí](https://www.w3.org/TR/webdriver/).

Junto con esta especificación también puedes encontrar la definición de las interfaces necesarias para la manipulación del *DOM* y el comportamiento del navegador. Lo interesante es que las especificaciones son independientes del lenguaje de programación de la implementación final.

## Symfony Panther
**¿Qué ofrece Symfony Panther?** *Symfony Panther* implementa este conjunto de especificaciones haciendo uso de [php-webdriver](https://github.com/php-webdriver/php-webdriver) y de ciertos componentes bien conocidos de Symfony como [BrowserKit](https://symfony.com/doc/current/components/browser_kit.html) y [DomCrawler](https://symfony.com/doc/current/components/dom_crawler.html).
 
 También añade lo necesario para facilitar el *testing* de aplicaciones (como una clase `PantherTestCase` y *assertions*, por ejemplo) y localiza automáticamente la instalación local de *Chrome* o *Firefox* mediante el uso de [ChromeDriver](https://chromedriver.chromium.org/) y/o [GeckoDriver](https://github.com/mozilla/geckodriver).

Si lo utilizas para *testing* no necesitas instalar un servidor web, ya que por defecto utiliza el *servidor web integrado de PHP*.

En resumen, con un navegador web local y un PHP con `composer` ya tienes lo necesario para empezar.

La programa final para *web scraping* de los libros resultó en poco más de 100 líneas de código (incluyendo namespaces y con formateado PSR-2), que pienso entra dentro de lo razonable.

# Instalación

Aunque hice uso de [console](https://symfony.com/doc/current/components/console.html) y los conocidos [config](https://symfony.com/doc/current/components/config.html), [yaml](https://symfony.com/doc/current/components/yaml.html), [dependency injection](https://symfony.com/doc/current/components/dependency_injection.html), [dotenv](https://symfony.com/doc/current/components/dotenv.html) y otros, su uso es es totalmente opcional.

A partir de aquí me centraré fundamentalmente en [Symfony Panther](https://github.com/symfony/panther).

Comenzamos con la instalación de Panther con `composer`
```
composer req symfony/panther
```
Si lo utilizas para *testing*, estarás acostumbrado a instalarlo con `--dev`, pero no es el caso.
 
Como primer paso lógico tenía que ser capaz de capturar todo el contenido de una página a un archivo (a poder ser un *pdf* o un *png*).

Como en mi caso utilicé el navegador *Firefox*

```php
use Symfony\Component\Panther\Client;

$this->client = Client::createFirefoxClient();
```

## Primera request y primera captura
Hacer una primera petición (*request*) a una página es directo

```php
use Symfony\Component\Panther\Client;
...
$crawler = $this->client->request('GET', $initURL);
```

Y hacer un pantallazo del contenido mostrado en el navegador es cosa sencilla *a priori*. 

La documentación de *Symfony Panther* explica en uno de los ejemplos como realizar una captura de pantalla a través de

```php
$this->client->takeScreenshot('filename.png');
```
Pero **¿y si necesito capturar un elemento en concreto?**

En la documentación de *WebDriver* puedes encontrar un [*snippet* para esto precisamente](https://github.com/php-webdriver/php-webdriver/wiki/Taking-Full-Screenshot-and-of-an-Element).

El «truco» empleado está claro: hacer la captura de todo el contenido y posteriormente recortar la imagen en función de la ubicación y dimensiones del elemento en el que estamos interesados y que se le pasa a la función.

Modifiqué un poco el código para adecuarlo a lo que necesitaba. Cambios triviales que tuvieron que ver con la organización y renombrado de los archivos para identificar las unidades o capítulos del libro de forma coherente.

```php
public function takeScreenshot($chapter, $page, WebDriverElement $element = null)
{

    $filename = str_pad($chapter, 2, 0, STR_PAD_LEFT) . '-' . str_pad($page, 3, 0, STR_PAD_LEFT);

    $screenshot = $filename. '-orig.png';

    $element_screenshot = $filename.'.png';

    $element_width = $element->getSize()->getWidth();
    $element_height = $element->getSize()->getHeight();

    $element_src_x = $element->getLocation()->getX();
    $element_src_y = $element->getLocation()->getY();

    $this->client->takeScreenshot($screenshot);
    if (!file_exists($screenshot)) {
        throw new Exception('Could not save screenshot');
    }

    $src = imagecreatefrompng($screenshot);
    $dest = imagecreatetruecolor($element_width, $element_height);

    imagecopy($dest, $src, 0, 0, (int)ceil($element_src_x), (int)ceil($element_src_y), (int)ceil($element_width),
        (int)ceil($element_height));

    imagepng($dest, $element_screenshot);

    unlink($screenshot);

    if (!file_exists($element_screenshot)) {
        throw new \Exception('Could not save element screenshot');
    }

    return $element_screenshot;
}
```

## No veo el navegador durante la ejecución
Existen ciertas [variables de entorno](https://github.com/symfony/panther#environment-variables) que puedes tocar para modificar el comportamiento de *Panther* y del navegador. Algunas variables afectan tanto a *Firefox* como a *Chrome* y otras son exclusivas de cada uno.

En concreto, si quieres ver lo que va haciendo el navegador durante la ejecución lo que necesitas es modificar la variable de entorno `PANTHER_NO_HEADLESS` (válida tanto para *Firefox* como para *Chrome*).
 
Puedes hacer uso de [dotenv](https://symfony.com/doc/current/components/dotenv.html) pero un
```
$ export PANTHER_NO_HEADLESS=1
```
antes de la ejecución de tu código es suficiente.

# Primeras preguntas

Llegado a este punto tanto si has hecho *web scraping* con anterioridad como si no las primeras dudas que pueden surgir son

* **¿En qué resolución del navegador se realiza la captura? ¿Se puede cambiar?**

* **¿En qué momento puedo saber si el contenido está ya totalmente renderizado para poder realizar la captura con lo que necesito?**

## La resolución de pantalla

En este caso la calidad final del libro depende directamente de la calidad de la captura y ésta directamente de la resolución de la pantalla cuando se realiza.
 
Si haces *testing end-to-end* este factor también es importante para poder comprobar si el contenido adaptable (*responsive*) se está comportando como debe.

Este método `manage()` del cliente *Panther* permite trabajar con *Cookies* y su método `window()` permite trabajar con tamaños (redimensionar, maximizar y minimizar) y orientaciones de pantalla.

```php
use Symfony\Component\Panther\Client;

...
$crawler = $this->client->request('GET', $initURL);

$this->client
    ->manage()
    ->window()
    ->setSize(new WebDriverDimension(1920, 1080));
```

## *Timeouts* y esperas con `wait` y `waitFor`

Seguro que lo primero que se te ha pasado por la cabeza para parar la ejecución y tener tiempo de ver el contenido del navegador es hacer un `sleep($number_of_seconds)` ¿a qué sí? 😛

Aunque puedes hacerlo, lo aconsejado de forma general es hacer uso de las funciones del cliente *WebDriver* (al que se accede desde el cliente *Panther*) como se indica en la [wiki de php-webdriver](https://github.com/php-webdriver/php-webdriver/wiki/HowTo-Wait).

*WebDriver* define dos tipos de **espera**:

1. **Implícita**: Consiste en establecer un tiempo de espera o *timeout* mientras trata de encontrar elementos (en caso en que éstos no estén disponibles de forma inmediata). Por defecto el valor de ese *timeout* es `0` y una vez que se asigna su valor se mantiene durante el tiempo de vida del cliente. 
```php
$client->manage()->timeouts()->implicitWait(10);
```

2. **Explícita**: Es la recomendada y se utiliza normalmente en combinación con distintas condiciones que puedes consultar en la [wiki de php-webdriver](https://github.com/php-webdriver/php-webdriver/wiki/HowTo-Wait). Para este tipo de espera se utiliza función `wait()` del cliente *Panther* (que llama por debajo a esta función de *WebDriver*): 
3. 
```php
    /**
     * Construct a new WebDriverWait by the current WebDriver instance.
     * Sample usage:
     *
     *   $driver->wait(20, 1000)->until(
     *     WebDriverExpectedCondition::titleIs('WebDriver Page')
     *   );
     *
     * @param int $timeout_in_second
     * @param int $interval_in_millisecond
     * @return WebDriverWait
     */
    public function wait(
        $timeout_in_second = 30,
        $interval_in_millisecond = 250
    );
```
El primer parámetro es el *timeout* (el tiempo máximo en el que se da como no cumplida la condición que sigue) y el segundo indica cada cuánto se ha de comprobar la condición.
 
 Si no se le pasan parámetros a `wait()` los valores del código anterior son los que se toman por defecto: `30s` para el *timeout* y `250ms` para el *intervalo de espera*.

Una consulta típica con `wait` tiene es del tipo 

```php
use Facebook\WebDriver\WebDriverExpectedCondition;
use Facebook\WebDriver\WebDriverBy;

$this->client->navigate()->to($url);
$this->client
    ->wait(30, 250)
    ->until(WebDriverExpectedCondition::elementToBeClickable(
        WebDriverBy::id('elementId'))
    );
```
Donde el método `navigate` hace exactamente lo que crees que hace.

Además de este método, *Panther* nos proporciona `waitFor`

```php
waitFor(
    string $locator,
    int $timeoutInSecond = 30,
    int $intervalInMillisecond = 250)
```
 pensado para ir directamente a la operación más común: **esperar hasta que el elemento especificado en `$locator` se renderice **.
 
 Aquí `$locator` es un *string* puede ser tanto un selector *CSS* como *XPath*.

```php
$this->client->navigate()->to($url);
$this->client->waitFor('.element');
```

## ¿Y qué pasa si utilizo `sleep()` para esperar? 

Pues corres el riesgo de que por algún pequeño corte o por lentitud en el acceso a internet tu script de *scraping* deje de funcionar.

Una de las cosas que hace `waitFor` es crear un nuevo *crawler* (hablo un poco de él en el siguiente punto) asociado al cliente *Panther* porque está a la espera de que se produzca un cambio en el DOM.

Si utilizas `sleep()` estarás obligado llamar a `$client->refreshCrawler()` cada vez que navegues entre páginas o cuando cambie el DOM por otro tipo de interacción.

Dicho esto, si estás en modo *debug* `PANTHER_NO_HEADLESS` y quieres parar la ejecución para poder visualizar un error o verificar que un comportamiento esté funcionando de forma correcta `sleep` es una opción.

# Navegación, interacción y recuperación de datos

En el apartado anterior he hablado del `crawler`. El componente [*DomCrawler*](https://symfony.com/doc/current/components/dom_crawler.html) tiene como misión facilitar la navegación a través del *DOM* en documentos *HTML* y *XML*.

Podemos acceder al `crawler` de diferentes maneras

```php
// Obtaining the $crawler
$crawler = $this->client->request('GET', $initUrl); // From simple request
$this->client->waitFor('.element'); // updates the crawler associated to the client
$crawler = $this->client->getCrawler();
$crawler = $this->client->refreshCrawler(); // Forces crawler refresh
...
// Examples
$numberUnits = $crawler->filter('a.descargas_uni')->count();
$title = $crawler->filter('#titulo_cent')->getText();
$chapter = $crawler->filter('a.descargas_uni')->getElement(0)->getText();
...
}
```

A partir de aquí el cliente de *Panther* nos permite interactuar de forma directa para enviar formularios, hacer clics directamente sobre elementos, movernos hacia delante y hacia atrás en el navegador, etc.

```php
$crawler->filter('a.descargas_uni')->getElement(1)->click();
$this->client->waitFor('.element'); // Creates a new Crawler
$this->client->navigate()->to($otherURL);
$this->client->waitFor('.element2'); // Creates a new Crawler
```
# Ejecutando Javascript. *Total control* 😏 

Todo lo anterior no bastó para llevar a buen término el *web scraping* de los libros de la editorial online, como os podéis imaginar.

Fue necesario hacer *submit* de algún formulario, realizar paginación sobre contenido, y alguna otra interacción **directamente mediante *javascript*.** 

Para ello el cliente dispone de un útil `executeScript`

```php
$this->client
    ->executeScript("document.getElementsByClassName('sig-p')[0].click()");
```

Con todos estos ingredientes ya solo quedaba recostarse en la silla y esperar hasta que todas las imágenes estuviesen listas y organizadas

```php
$pageElements = $this->client->getCrawler()->filter('div.pc');
$numberPages = $pageElements->count();

$io->title(sprintf("Unit: %d (%d pages)", $chapter, $numberPages));
$io->progressStart($numberPages);
while ($page < $numberPages) {
    $element = $pageElements->getElement($page);
    $this->takeScreenshot($chapter, $page + 1, $element);
    $page++;
    $io->progressAdvance();
...
```

# Generando el PDF a partir de las imágenes

Siempre te puedes liar la manta a la cabeza y tirar de los paquetes típicos para la generación de PDF, como [dompdf](https://packagist.org/packages/dompdf/dompdf) o [tcpdf](https://packagist.org/packages/tecnickcom/tcpdf).
 
Pero este no es el caso y con [ImageMagick](https://imagemagick.org/) ya disponemos de lo necesario.

Una vez organizadas las imágenes con nombres correlativos (por ejemplo) *ImageMagick* hace el resto:

```bash
convert *.png -quality 100 filename.pdf
```

Si queréis utilizar [symfony/console](https://symfony.com/doc/current/components/console.html) y [symfony/process](https://symfony.com/doc/current/components/process.html) esto lo podéis poner como:

```php
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;
use Symfony\Component\Process\Process;
...
$io->section('Beginning PDF generation from images');
$process = new Process(['convert', '*.png', '-quality', '100', 'book.pdf']);
$process->run();
// Only if you want to get rid of the images after completion
array_map('unlink', glob("*.png"));
$io->success('Book ready!!!');
```

# Conclusión

En este post solo se han arañado las posibilidades más básicas de *Symfony Panther*. Lo necesario para un primer proyecto. En un *web scraping* un poco más serio lo habitual es que tengamos que hacer uso de *proxies* y otras opciones un poco más avanzadas que también están disponibles.

Decir también que es un software en continuo desarrollo que tiene algunas limitaciones: 
* Solo están soportados documentos *HTML* (no XML)
* No está soportada la actualización de documentos
* El envío de formularios utilizando la sintaxis de *arrays* multidimensionales de PHP no funciona
* No funcionan los métodos que devuelven una instancia de `\DomElement` (porque la librería utiliza `WebDriverElement` internamente).

Pese a estas limitaciones es un proyecto muy útil que bien merece una estrella en su [Github](https://github.com/symfony/panther) y un agradecimiento a [Kévin Dunglas](https://dunglas.fr/) y a [Les-Tilleuls.coop](https://les-tilleuls.coop/) por el apoyo al proyecto.

## Save the panthers

Paso a reproducir aquí también un mensaje que aparece en el Readme del repositorio oficial de *Symfony Panther*

> Many of the wild cat species are highly threatened. If you like this software, help save the (real) panthers by [donating to the Panthera organization](https://www.panthera.org/donate).
