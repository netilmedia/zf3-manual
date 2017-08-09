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

Dodanie fabryki i aliasu w pliku <M>/config/module.config.php

```php
return [
//...
['controllers' => [
	'factories' => [
		//...
		Controller\NazwaController::class => Controller\Factory\NazwaControllerFactory::class,
	],
	'aliases' => [
		//...
		// nazwa aliasu (example.com/module/nazwa-aliasu
		'nazwa-aliasu' => Controller\NazwaController::class,
		]
	]
]
```

  

Dodanie pliku Controllera w katalogu

	module/
		NazwaModułu/
			src/
				Controller/
					NowyController.php	
					
				

Dodanie fabryki Controllera w katalogu

	 module/
		 <NazwaModułu>/
			 src/			 
				 Controller/
					Factory/
						NowyControllerFactory.php	
					



Dodatek A Nowości PHP
=====================

**::class** - jest to specjalna stała klasy, której wartością jest nazwa klasy z całą przestrzenią nazw np. IndexController::class zwraca: \Application\Controller\IndexController


----



