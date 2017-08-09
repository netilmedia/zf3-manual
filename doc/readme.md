Spis treści:

[TOC]

QuickStart
==========

Aktualizacja Modułu
-------------------
 1. Pobranie katalogu modułu
 2. Pobranie pliku /config/modules.config.php i / modules.acl.roles.php
 3. Pobranie /composer.json
 4. Uruchomienie komendy

```
> composer dump-autoload
```

Dodanie nowego Controllera
--------------------------

 1. Dodanie fabryki i aliasu w pliku <M>/config/module.config.php
	 Klucz tablicy

```php
return [
//...
	['controllers' => [
        'factories' => [
        //...
            Controller\NazwaController::class => Controller\Factory\NazwaControllerFactory::class,
        ],
        'aliases' => [
            'ticket-listing' => Controller\TicketListingController::class,
            'ticket-create' => Controller\TicketCreateController::class,
            'ticket-details' => Controller\TicketDetailsController::class,
        ]
	]
```

	 Klucz tablicy controllers/aliases
    Dodanie pliku Controllera w katalogu <M>/Controller Dodanie fabryki
    Controllera w katalogu <M>/Controlller/Factory Dodanie katalogu
    widoków do katalogu <M>/view/<M>/nazwa_controllera

Dodatek A Nowości PHP
=====================

**::class** - jest to specjalna stała klasy, której wartością jest nazwa klasy z całą przestrzenią nazw np. IndexController::class zwraca: \Application\Controller\IndexController


----



