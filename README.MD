# Decorator x Inheritance

although inheritance seems good and works fine, it kind of goes against solid principles of mainly Open-closed principle and a little bit of Single-responsability principle.

EG:

let's imagine the following... A system for a mechanic

```php
class CarService {
    protected $cost = 25;

    public function getCost() : int
    {
        return $this->cost;
    }

    public function setCost(int $cost) : int
    {
        $this->cost = $cost;
    }

    public function withOilChange() : int
    {
        return $this->cost += 29;
    }

    public function withTireRotation() : int
    {
        return $this->cost += 15;
    }
}
```

Even though this works, it doesn't seem very closed + changing these after some period of time would not be the most beautiful thing in life, would it? Imagine having to add new methods here, that are based on a return of another method.

therefore Decorators would be a much cleaner approach:

```php
interface CarService {
    public function getCost();

    public function getDescription();
}
```

```php
class BasicInspection implements CarService {
    public function getCost() : int
    {
        return 25;
    }

    public function getDescription() : string
    {
        return 'Basic inspection';
    }
}
```

```php
class OilChange implements CarService {
    protected $carService;

    function __construct(CarService $carService)
    {
        $this->carService = $carService;
    }

    public function getCost()
    {
        return 29 + $this->carService->getCost();
    }

    public function getDescription()
    {
        return $this->carService->getDescription() . ', and an oil change';
    }
}
```

```php
class TireRotation implements CarService {
    protected $carService;

    function __construct(CarService $carService)
    {
        $this->carService = $carService;
    }

    public function getCost()
    {
        return 35 + $this->carService->getCost();
    }

    public function getDescription()
    {
        return $this->carService->getDescription() . ', and a tire rotation';
    }
}
```

```php
    $service = new TireRotation(new BasicInspection);
    $service2 = new OilChange(new TireRotation(new BasicInspection));

    echo $service->getDescription();
    echo $service->getCost();
    echo "-----------------";
    echo $service2->getDescription();
    echo $service2->getCost();
```

Having our core service "CarService"
and having the decorators implement the core service. Together with the constructor of the decorators, forcing an implementation of the core... and there we go! objects built at Runtime!

So the question is: Do I really need the entirely of the functionality? or just adjust the behaviour of one or two methods?

# Adapter Design Pattern

The name says it all, adapt/translate one interface to another.

```php
class Person {
    public function read(BookInterface $book)
    {
        $book->open();
        $book->turnPage();
    }
}

```

```php
class Book implements BookInterface {
    public function open()
    {
        var_dump('opening the paper book.');
    }

    public function turnPage()
    {
        var_dump('turning the page of the paper book.');
    }
} 
```

```php
interface BookInterface {
    public function open();
    
    public function turnPage();
}
```

```php
interface eReaderInterface {
    public function turnOn();

    public function pressNextButton();
}
```

```php
class Kindle implements eReaderInterface {
    public function turnOn()
    {
        var_dump('turn the Kindle on');
    }
    
    public function pressNextButton()
    {
        var_dump('press the next button on the Kindle');
    }
} 
```

```php
class eReaderAdapter implements BookInterface {
    private $reader;

    public function __construct(eReaderInterface $reader)
    {
        $this->reader = $reader;
    }

    public function open()
    {
        return $this->reader->turnOn();
    }

    public function turnPage()
    {
        return $this->reader->pressNextButton();
    }
}
```

```php
$person = new Person;

$person->read(new Book);
$person->read(new KindleAdapter(new Kindle));
```

So in this case example, I originally had a person read a book only, but by receiving the BookInterface, anyone that implements the bookinterface and I receive will be used...
thus the ereaderadapter will implement the original bookinterface methods and overwrite its functionality by using the reader interface initiated in the constructor...