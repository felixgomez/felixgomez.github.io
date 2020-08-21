---
title: El patrón observer
categories:
- PHP
- Design patterns
tags:
    - php
    - design-patterns
thumbnail: /images/design-patterns/observer/binoculars-1280.png
date: 2019-07-31
---

![](/images/design-patterns/observer/binoculars-1280.png)

Hoy me ha tocado hablar en una charla en [PHPVigo](https://phpvigo.com) sobre el **patrón Observer**.
Por aquí dejo la [presentación](https://docs.google.com/presentation/d/1fSPNsZ9zT_IybxFlZCyi93pw9A9NSEu41SGjWhp8cp8/edit?usp=sharing) (Observer a partir de la página 66) y un pequeño resumen.

<!-- more -->

## Qué hace

Permite a un objeto mantener una lista de objetos a los que notifica cuando su estado cambia.

## Conceptos que se manejan

- **SUBJECT (aka OBSERVABLE)** El objeto que notifica su cambio de estado a los que lo observan.
- **OBSERVER** Lista de objetos que mantiene el objeto observable y que son notificados por él.

![](/images/design-patterns/observer/observer-concepts.png)

## Interfaces

La interfaz para el objeto *Observable* es:
```php
interface Observable {
    public function attach (Observer $observer);
    public function detach (Observer $observer);
    public function notify();
}
```
y para el objeto observer:
```php
interface Observer {
    public function update (Observable $observable);
}
```

## El patrón observer en la SPL (Standard PHP Library)
La librería estándar de PHP dispone de una serie de interfaces para implementar este patrón y una clase (**SplObjectStorage**) que permite al *Observable* almacenar los *Observers* a los que tiene que notificar.

![](/images/design-patterns/observer/observer-spl.png)

*SplObjectStorage* dispone de los métodos `attach` y `detach` para añadir objectos o eliminarlos.

## Ejemplo de implementación

### ExampleSubject (Observable)

```php
class ExampleSubject implements \SplSubject
{
    private $storage;

    public function __construct()
    {
        $this->storage = new \SplObjectStorage();
    }
    
    public function attach(\SplObserver $observer)
    {
        $this->storage->attach($observer);
    }

    public function dettach(\SplObserver $observer)
    {
        $this->storage->dettach($observer);
    }
    
    public function notify()
    {
        foreach ($this->storage as $observer) {
            $observer->update($this);
        }
    }
}
```

En la línea `7` inicializamos un **SplObjectStorage** donde poder almacenar los *Observers* y
en la línea `23` vemos que de forma opcional podemos pasar el propio *Observer* a los observables como parámetro.

### ExampleObserver

```php
class ExampleObserver implements \SplObserver
{
    public function __construct(\SplSubject $subject)
    {
        $subject->attach($this);
    }

    public function update(\SplSubject $subject)
    {
        if ($subject instanceof ExampleSubject) {
            $this->doUpdate($subject);
        }
    }

    public function doUpdate(ExampleSubject $subject) {
        // Something to do
    }
}
```

## Ejemplo de utilización

Podéis ver un ejemplo muy sencillo en [Github](https://github.com/phpvigo/PHPVigo-31-Patrones-de-software-en-la-practica).
