Spis treści:

[TOC]

Kontroler
=

Wprowadzenie
-

Zgodnie z wzorcem [MVC](https://pl.wikipedia.org/wiki/Model-View-Controller) kontroler *(Controller)* obsługuje żądanie *(request)* łącząc przy tym model *(Model)* z widokiem *(View)*. Najprościej mówiąc jeżeli użytkownik wpisuje adres:
```
www.example.com/application/test/akcja
```
 uruchamia w ten sposób kontroler **Test** *(TestController)*, który następnie pobiera potrzebne modele (np. produkt w sklepie) i przekazuje do zwracanego widoku (np. generuje kod HTML, XML czy JSON).

Przykład wywołania adresu sklepu:

```
www.example.com/sklep/produkt/widok/id/1
```

Uruchamia kontroler **produkt** *(ProductController)* a następnie akcję kontrolera **widok** *(viewAction)*. Akcja pobiera wartość parametru **id**, następnie pobiera produkt o takim id i przekazuje do widoku.

**Jak to wygląda w ZF?**

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

Plik widoku obsługujący wyświetlenie danych produktu mógłby wyglądać następująco:

```php
<html>
    <title><?php $this->product->getName() ?> - sklep example.com</title>
    <body>
        <h1><?php echo $this->product->getName(); ?></h1>
        <p><?php echo $this->product->getDescription(); ?></p>
        <dl>
            <dt>Cena</dt>
            <dd><?php echo number_format($this->product->getPrice(), 2, ',' ' '); ?>
        </dl>
        <button>dodaj do koszyka</button>
    </body>
</html>
```

Jak widać na powyższym przykładzie w pliku widoku możemy odwołać się do zmiennej zawierającej obiekt *$product* przekazanej w kontrolerze.

ZF w całości obsługuje wzorzec MVC, po stronie programisty pozostaje jedynie obsługa wygenerowania odpowiedniej odpowiedzi.

Dodanie nowego kontrolera
-

Poniższe przykłady pokazują jak zbudować kontroler użytkownika *(UserController)*

Dodanie fabryki i aliasu w pliku */module/Application/config/module.config.php*

```php
return [
//...
    ['controllers' => [
        'factories' => [
            //...
            Controller\UserController::class => Controller\Factory\UserControllerFactory::class,
        ],
        'aliases' => [
            //...
            // nazwa aliasu (example.com/application/user
            'user' => Controller\UserController::class,
        ]
    ]
//...
]
```

  

Dodanie pliku kontrolera *UserController.php* w katalogu

```
/module
    /Application
        /src
            /Controller
```

```php
// plik UserController.php
<?php
namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class UserController extends AbstractActionController {

    public function indexAction() {
        
    }
}
```

Dodanie pliku fabryki kontrolera *UserControllerFactory.php* w katalogu

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
indexAction zwraca widok */view/Application/user/index.phtml*

W celu przekazania do niego parametru w akcji zwracamy tablicę:


```php
//UserController.php
<?php
// tutaj nagłówek z poprzedniego przykładu

class UserController extends AbstractActionController {

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
<?php
// UserController.php

// ...

// importujemy ViewModel
use Zend\View\Model\ViewModel;

class UserController extends AbstractActionController {

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
    $testView = $this->forward()->dispatch(UserController::class, [
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
// plik view/application/user/index.phtml
echo $this->testView;
```

Korzystanie z modułu Zend\Form
======

Moduł Zend\Form daje możliwość zdefiniowania pól formularza, dodanie do nich filtrów i walidacji.

Definiowanie formularza
------

W poniższych przykładach stworzymy formularz rejestracji użytkownika.

Tworzymy plik */Application/src/Form/UserForm.php* w katalogu:

```php
<?php
// UserForm.php

namespace Application\Form;

use Zend\Form\Form;

class UserForm extends Form { // extends Zend\Form\Form
    
    public function __construct() {
        
        parent::__construct('user');

        $this->init();
    }

    public function init() {
        
        // dodajemy pole tekstowe "nazwa użytkownika"
        $this->add([
            'name' => 'username',
            'type' => 'text',
            'options' => [
                'label' => 'Nazwa użytkownika'
            ],
            'attributes' => [
                'placeholder' => 'Nazwa użytkownika'
            ]
        ]);

        // dodajemy pole tekstowe "hasło"
        $this->add([
            'name' => 'password',
            'type' => 'password',
            'options' => [
                'label' => 'Hasło'
            ]
        ]);
        
        // dodajemy guzik
        $this->add([
            'name' => 'submit',
            'type' => 'button',
            'attributes' => [
                'type' => 'submit',
                'value' => 'Zarejestruj się'
            ]
        ]);
    }
}
```

Tak utworzony formularz możemy przekazać do widoku w naszym kontrolerze:
```php
<?php
// UserController.php
namespace Application/Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Application\Form\UserForm;

class UserController extends AbstractActionController {    

    public function formAction() {
        
        $form = new UserForm();
        // przekazujemy obiekt formularza do widoku
        return [
            'form' => $form
        ];
    }
}
```

Tworzymy plik widoku w katalogu */module/Application/view/application/user/form.phtml*

```php
<?php
// przekazujemy do lokalnej zmiennej $form referencje do
// obiektu $this->form 
$form = &$this->form;
// ustawiamy atrybut action na /application/user/create
// i wywołujemy metodę prepare() przed użyciem formularza
// w widoku (wymagane)
$form->setAttribute('action', $this->url('application', [
        'controller' => 'user',
        'action' => 'create'
    ]))->prepare();

// pobieramy obiekt pola "username"
// i dodajemy do niego atrybut class="form-control"
$username = $form->get('username')
            ->setAttribute('class', 'form-control');

// pobieramy obiekt pola "password"
$password = $form->get('password')
            ->setAttribute('class', 'form-control');

// pobieramy obiekt guzika "submit"
$submit = $form->get('submit')
            ->setAttribute('class', 'btn btn-primary');
?>
<div class="container">
    <?php echo $this->form()->openTag($form); ?>
    
    <div class="form-group">
        <?php echo $this->formLabel($username); // wyświetlamy etykietę pola "username" ?>
        <?php echo $this->formElement($username); // wyświetlamy pole "username" ?>
    </div>
    <div class="form-group">
        <?php echo $this->formRow($password); // wyświetlamy etykietę i pole "password" korzystając z formRow ?>
    </div> 
    <?php echo $this->formElement($submit); ?>
    
    <?php echo $this->form()->closeTag($form); ?>
</div><!-- end .container -->
```
Powyższy przykład wygeneruje kod HTML:

```html
<div class="container">
    <form>
        <div class="form-group">
            <label>Nazwa użytkownika</label>
            <input type="text" class="form-control" placeholder="nazwa użytkownika">
        </div>
        <div class="form-group">
            <label>hasło</label>
            <input type="password" class="form-control">
        </div>        
        <button type="submit">Zarejestruj się</button>
    </form>
</div><!-- end .container -->
```

Obsługa danych z formularza
------

W celu obsłużenia danych wysłanych przez nasz formularz wracamy do kontrolera i dodajemy akcję *createAction()*.

```php
// UserController.php

public function createAction() {

    $form = new UserForm();

    if ($this->getRequest()->isPost()) {
        //
        $data = $this->params()->fromPost();
        
        $form->setData($data);
        
        if ($form->isValid()) {
			
			// dane są prawidłowe...            
            
        } else {
            // w przypadku błędnie wypełnionego formularza
            // wyświetlamy wynik na ekran dla celów przykładu
            var_dump($form->getMessages());
        }
    }    

    $viewModel = new ViewModel();
    $viewModel->addChild($userForm, 'form');
    return $viewModel;
}

```

Obsługa bazy danych korzystając z Doctrine
======

Pobieranie danych
------

Klasa EntityManager pozwala na łatwe pobranie encji jeżeli szukamy korzystając z  identyfikatora zadeklarowanego w klasie encji:

```php
// pobieranie encji User o id = 1
$user = $this->em->find(User::class, 1);
// SELECT * FROM user WHERE user_id = 1;
```

Jeżeli chcemy pobrać różną ilość rekordów lub skorzystać z kryteriów wyszukiwania musimy skorzystać z metod udostępnianych przez repozytorium encji:
```php
// pobiera dane encji tak samo jak powyższy przykład
$user = $this->em->getRepository(User::class)->find(1);
``` 
Pobieranie wszystkich rekordów:

```php
$users = $this->em->getRepository(User::class)->findAll();
// SELECT * FROM user
```
Metoda *findAll()* nie przyjmuje żadnych argumentów, więc nie możemy sortować lub wyszukiwać danych. Aby to zrobić musimy użyć metody *findBy()*:
```
$where = [
    'first_name' => 'Jan'
];

$orderBy = [
    'id' => 'DESC'
];

$limit = 10;

$offset = 0;

$users = $this->em->getRepository(User::class)
            ->findBy($where, $orderBy, $limit, $offset);

// SELECT * FROM user
// WHERE first_name = 'Jan'
// ORDER BY id DESC LIMIT 0, 10
```

Dodawanie nowych rekordów
------

```php
<?php
// UserManager.php
namespace Applcation\Service;

use Application\Entity\User;

class UserManager {

    protected $entityManager;    

    public function __construct($entityManager) {
        $this->entityManager = $entityManager;
    }

    /**
     * @return User
     */
    public function create(array $data) {
    
        $user = new User();
        $hydrator->hydrate($data, $user);

        $this->entityManager->persist($user)
                ->flush();

        return $user;
    }
}
```


Model
======

Wracając do wzorca MVC mamy już nasz kontroler ( **C** ontroller) i widok ( **V** iew). Nadszedł więc czas na nasz model ( **M** odel).

Korzystając z biblioteki Doctrine tworzymy w tym celu encję **User**.

Dodanie pliku */Application/src/Entity/User.php*

```php
// User.php
namespace Application\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="user")
 */
class User {

    /**
     * @ORM\Id;
     * @ORM\GeneratedValue
     * @ORM\Column(name="user_id", type="integer")
     */
    protected $id;

    /**
     * @ORM\Column(name="username")
     */
    protected $username;

    /**
     * @ORM\Column(name="password")
     */
    protected $password;

    public function getId() {
        return $this->id;
    }

    public function setId($id) {
        $this->id = $id;
        return $this;
    }

    public function getUsername() {
        return $this->username;
    }

    public function setUsername($username) {
        $this->username;
        return $this;
    }

    public function getPassword($password) {
        return $this->password;
    }

    public function setPassword($password) {
        $this->password = $password;
        return $this;
    }
}
```

Doctrine używa adnotacji (annotations), czyli specjalnych komentarzy w celu mapowania klasy encji z danymi tabeli.

W naszej encji użyliśmy następujących adnotacji:

```php
/**
 * @ORM\Entity
 * @ORM\Table(name="user")
 */
```

**@ORM\Entity** - identyfikuje klasę User jako encję.

**@ORM\Table(name="user")** - wskazuje nazwę tabeli, w której przechowywane są dane encji.

Następnie definiujemy atrybuty encji, w tym przypadku wszystkie odpowiadają kolumnom tabeli *user*.

```php
/**
 * @ORM\Id
 * @ORM\GeneratedValue
 * @ORM\Column(name="user_id", type="integer")
 */
``` 
**@ORM\Id** - identyfikuje pole jako identyfikator encji.

**@ORM\GeneratedValue** - informuje Doctrine o ustawionym parametrze *auto_increment* dla danej kolumny.

**@ORM\Column(name="user_id", type="integer")** - odpowiada za wskazanie kolumny w tabeli user dla atrybutu $id czyli user_id i dodatkowo informuje, że kolumna jest typu int (Doctrine automatycznie wykona rzutowanie dla tej kolumny).
Domyślnie wszystkie atrybuty są typu tekstowego *type="string"*. Deklarowanie typów pól po stronie encji nie jest wymagane, jednak zachęcam do tego, ponieważ ułatwia to identyfikowanie błędów już po stronie PHP.

Moduły
======

Aktualizacja Modułu
------
 1. Pobranie katalogu modułu
 2. Pobranie pliku /config/modules.config.php i / modules.acl.roles.php
 3. Pobranie /composer.json
 4. Uruchomienie komendy

```
> composer dump-autoload
```



Dodatek A: Nowości PHP
======
**::class** - jest to specjalna stała klasy, której wartością jest nazwa klasy z całą przestrzenią nazw np.  
```php
echo IndexController::class
// wyświetli \Application\Controller\IndexController
```


----



