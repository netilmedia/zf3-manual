Spis treści:

[TOC]

Kontroler
=

Zgodnie z wzorcem [MVC](https://pl.wikipedia.org/wiki/Model-View-Controller) kontroler *(Controller)* łączy model *(Model)* z widokiem *(View)*. Najprościej mówiąc jeżeli użytkownik wpisuje adres:
```
www.example.com/application/test/akcja
```
 uruchamia w ten sposób kontroler **Test** *(TestController)*, który następnie pobiera potrzebne modele (np. produkt w sklepie) i przekazuje do zwracanego widoku.

Przykład wywołania adresu sklepu:

```
www.example.com/sklep/produkt/widok/id/1
```

Uruchamia kontroler **produkt** *(ProductController)* a następnie akcję kontrolera **widok** *(viewAction)*. Akcja pobiera wartość parametru **id**, następnie pobiera produkt o takim id i przekazuje do widoku.

Jak to wygląda w ZF?

```php
<?php
// Plik kontrolera ProductController.php

// Klasa kontrolera
class ProductController extend AbstractControllerAction {
    
    // akcja viewAction
    public function viewAction() {
        // pobranie id z adresu
        $id = (int)$this->params()->fromRoute('id');

        // pobranie produktu (modelu) np. z tabeli w bazie danych
        $product = $this->productDbTable->getProductById($id);

        // zwrócenie widoku z przekazanym obiektem $product
		return ['product' => $product];
    }
}
```

Dodanie nowego kontrolera
-

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

  

Dodanie pliku kontrolera *TestController.php* w katalogu

```
/module
    /Application
        /src
            /Controller
```

```php
// plik TestController.php
<?php
namespace Application\Controller;

class TestController extends AbstractControllerAction {

    public function indexAction() {
        
    }
}
```

Dodanie pliku fabryki kontrolera *TestControllerFactory.php* w katalogu

```
/module
    /Application
        /src
            /Controller
                /Factory	
```					

Połączenie kontrolera z widokiem
---------------------------------

Domyślnie kontroler zwraca powiązany z nim widok np. 
indexAction zwraca widok *view/Application/index/index.phtml*

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

Korzystanie z modułu Zend\Form
=

Moduł Zend\Form daje możliwość zdefiniowania pól formularza, dodanie do nich filtrów i walidacji.

Definiowanie formularza
-

Tworzymy plik *TestForm.php* w katalogu:

```
/module
    /Application
        /src
            /Form
```

Moduły
=

Aktualizacja Modułu
-------------------
 1. Pobranie katalogu modułu
 2. Pobranie pliku /config/modules.config.php i / modules.acl.roles.php
 3. Pobranie /composer.json
 4. Uruchomienie komendy

```
> composer dump-autoload
```



Dodatek A: Nowości PHP
=====================

**::class** - jest to specjalna stała klasy, której wartością jest nazwa klasy z całą przestrzenią nazw np.  
```php
echo IndexController::class
// wyświetli \Application\Controller\IndexController
```


----



