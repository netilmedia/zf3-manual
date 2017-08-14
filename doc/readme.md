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

Kontroler
===

Dodanie nowego kontrolera
--------------------------

Dodanie fabryki i aliasu w pliku /module/*NazwaModułu*/config/module.config.php

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
//...
]
```

  

Dodanie pliku kontrolera w katalogu

	/module
	    /<NazwaModułu>
	        /src
	            /<NazwaKontrolera>
	                /NowyController.php	
					
				

Dodanie fabryki kontrolera w katalogu

	 /module
	    /<NazwaModułu>
	        /src
	            /<NazwaKontrolera>
	                /Factory
	                    /NowyControllerFactory.php	
					

Połączenie kontrolera z widokiem
---------------------------------

Domyślnie kontroler zwraca powiązany z nim widok np. 
indexAction zwraca widok view/*NazwaModułu*/*NazwaKontrolera*/index.phtml

W celu przekazania do niego parametru w akcji zwracamy tablicę:


```php
//IndexController.php
namespace Application\Controller;
	
class IndexController extends AbstractControllerAction {

    public function indexAction() {
        $wartosc = 'test';
        // tak zwrócona tablica powoduje zamianę jej
        // przez klasę AbstractControllerAction na
        // obiekt ViewModel(['parametr' => $wartosc']
        return ['parametr' => $wartosc];
    }
}
```

Ten sam efekt uzyskamy poprzez zwrócenie obiektu tak jak robi to klasa AbstractControllerAction:

```php
namespace Application\Controller;
//potrzebne jedynie w przypadku użycia new ViewModel() w akcji
use Zend\View\Model\ViewModel;

class IndexController extends AbstractControllerAction {

    public function indexAction() {
        return new ViewModel([
            'parametr' => $wartosc
        ]);
    }
}
```

Posiadając obiekt ViewModel możemy przekazać do niego parametry na kilka sposobów. Jednym z nich jest przekazanie tablicy do konstruktora obiektu tak jak w przykładzie powyżej. Kolejnym sposobem jest użycie metod:

```php
public function indexAction() {

    $viewModel = new ViewModel();
    // przekazanie jednego parametru
    $viewModel->setVariable('parametr', 'wartosc');
    
    // lub przekazanie wielu parametrów w tablicy
    $viewModel->setVariables([
        'parametr1' => 'wartosc1',
        'parametr2' => 'wartosc2'
    ]);

    return $viewModel;
}
```

Przekazanie widoku jednej akcji do widoku drugiej:

```php
public function indexAction() {
    $testView = $this->forward()->dispatch(IndexController::class, [
        'action => 'test'
    ]);
    
    $viewModel = new ViewModel();
    $viewModel->addChild($testView, 'testView');
    
    return $viewModel;
}

public function testAction() {
    // pobieranie wartości parametru /?parametr=wartosc
    $wartosc = $this->params()->fromQuery('parametr');
    return ['parametr' => $wartosc];
}
```

W widoku możemy się wtedy odwołać w następujący sposób:

```php
// plik view/application/index/index.phtml
echo $this->testView;
```

Dodatek A: Nowości PHP
=====================

**::class** - jest to specjalna stała klasy, której wartością jest nazwa klasy z całą przestrzenią nazw np.  
```php
echo IndexController::class
// wyświetli \Application\Controller\IndexController
```


----



