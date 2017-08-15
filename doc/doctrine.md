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