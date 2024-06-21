# Some Design Patterns in PHP

## 1. Singleton Pattern

**Purpose**: Ensures a class has only one instance and provides a global point of access to it.

**Features**:

- Private constructor to prevent direct creation of objects.
- Static method to get the instance.
- Prevents cloning and unserializing.

**Usage**:

- Database connections.
- Logging.
- Configuration settings.

**Example**:

```php
<?php
class Singleton {
    private static $instance = null;

    private function __construct() {
        // Private constructor to prevent instantiation.
    }

    public static function getInstance(): Singleton {
        if (self::$instance === null) {
            self::$instance = new Singleton();
        }
        return self::$instance;
    }

    private function __clone() {
        // Prevent cloning.
    }

    public function __wakeup() {
        throw new \Exception("Cannot unserialize singleton");
    }
}

// Usage
$instance = Singleton::getInstance();
?>
```

## 2. Factory Pattern

**Purpose**: Provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.

**Features**:

- Centralizes object creation logic.
- Makes code more flexible and reusable.

**Usage**:

- Object creation based on conditions or configurations.

**Example**:

```php
<?php
interface Shape {
    public function draw();
}

class Circle implements Shape {
    public function draw() {
        echo "Inside Circle::draw() method.";
    }
}

class Rectangle implements Shape {
    public function draw() {
        echo "Inside Rectangle::draw() method.";
    }
}

class ShapeFactory {
    public function getShape(string $shapeType): Shape {
        if ($shapeType === 'circle') {
            return new Circle();
        } elseif ($shapeType === 'rectangle') {
            return new Rectangle();
        } else {
            throw new \InvalidArgumentException("Unsupported shape type");
        }
    }
}

// Usage
$factory = new ShapeFactory();
$circle = $factory->getShape('circle');
$circle->draw();

$rectangle = $factory->getShape('rectangle');
$rectangle->draw();
?>
```

## 3. Observer Pattern

**Purpose**: Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Features**:

- **Subject Interface**: Provides methods to attach, detach, and notify observers.
- **Observer Interface**: Specifies an update method that concrete observers must implement.
- **Loose Coupling**: Enables subject and observers to interact without knowing each other's details.

**Usage**:

- Event-driven systems.
- UI components reacting to data changes.
- Monitoring and logging applications.

**Example**:

```php
<?php
// Subject (Publisher) interface
interface Subject {
    public function attach(Observer $observer);
    public function detach(Observer $observer);
    public function notify();
}

// Concrete Subject (Concrete Publisher)
class WeatherStation implements Subject {
    private $temperature;
    private $observers = [];

    public function setTemperature(float $temperature) {
        $this->temperature = $temperature;
        $this->notify(); // Notify observers on state change
    }

    public function attach(Observer $observer) {
        $this->observers[] = $observer;
    }

    public function detach(Observer $observer) {
        $key = array_search($observer, $this->observers);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }

    public function notify() {
        foreach ($this->observers as $observer) {
            $observer->update($this->temperature);
        }
    }
}

// Observer interface
interface Observer {
    public function update(float $temperature);
}

// Concrete Observer
class TemperatureDisplay implements Observer {
    public function update(float $temperature) {
        echo "Temperature Display: " . $temperature . " degrees Celsius<br>";
    }
}

// Concrete Observer
class LogObserver implements Observer {
    public function update(float $temperature) {
        echo "Logging Temperature: " . $temperature . " degrees Celsius<br>";
    }
}

// Usage
$weatherStation = new WeatherStation();
$display = new TemperatureDisplay();
$log = new LogObserver();

$weatherStation->attach($display);
$weatherStation->attach($log);

$weatherStation->setTemperature(25.5);
$weatherStation->setTemperature(28.0);

$weatherStation->detach($log);

$weatherStation->setTemperature(30.2);
?>
```

## 4. Strategy Pattern

**Purpose**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from clients that use it.

**Features**:

- **Context**: Maintains a reference to a Strategy object.
- **Strategy Interface**: Declares an interface common to all supported algorithms.
- **Concrete Strategies**: Implement different algorithms defined by the Strategy interface.

**Usage**:

- Selecting sorting algorithms.
- Payment processing with different providers.
- Text processing with various formatting options.

**Example**:

```php
<?php
// Strategy Interface
interface PaymentStrategy {
    public function pay(float $amount);
}

// Concrete Strategies
class CreditCardPayment implements PaymentStrategy {
    public function pay(float $amount) {
        echo "Paying with Credit Card: $" . $amount . "<br>";
    }
}

class PayPalPayment implements PaymentStrategy {
    public function pay(float $amount) {
        echo "Paying with PayPal: $" . $amount . "<br>";
    }
}

// Context
class ShoppingCart {
    private $paymentStrategy;

    public function setPaymentStrategy(PaymentStrategy $paymentStrategy) {
        $this->paymentStrategy = $paymentStrategy;
    }

    public function checkout(float $amount) {
        $this->paymentStrategy->pay($amount);
    }
}

// Usage
$cart = new ShoppingCart();

// Pay with Credit Card
$cart->setPaymentStrategy(new CreditCardPayment());
$cart->checkout(100.0);

// Pay with PayPal
$cart->setPaymentStrategy(new PayPalPayment());
$cart->checkout(50.0);
?>
```

## 5. Decorator Pattern

**Purpose**: Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Features**:

- **Component Interface**: Defines the interface for objects that can have responsibilities added to them dynamically.
- **Concrete Component**: Defines an object to which additional responsibilities can be attached.
- **Decorator**: Maintains a reference to a Component object and conforms to the Component interface. It adds its own functionality before and/or after delegating to the Component.

**Usage**:

- Extending functionalities of classes without creating subclasses.
- Adding features to objects dynamically at runtime.
- Maintaining the single responsibility principle.

**Example**:

```php
<?php
// Component Interface
interface Coffee {
    public function getCost(): float;
    public function getDescription(): string;
}

// Concrete Component
class SimpleCoffee implements Coffee {
    public function getCost(): float {
        return 2.0; // Base cost of simple coffee
    }

    public function getDescription(): string {
        return "Simple Coffee";
    }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected $decoratedCoffee;

    public function __construct(Coffee $coffee) {
        $this->decoratedCoffee = $coffee;
    }

    public function getCost(): float {
        return $this->decoratedCoffee->getCost();
    }

    public function getDescription(): string {
        return $this->decoratedCoffee->getDescription();
    }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
    public function getCost(): float {
        return $this->decoratedCoffee->getCost() + 1.0; // Add cost of milk
    }

    public function getDescription(): string {
        return $this->decoratedCoffee->getDescription() . ", Milk";
    }
}

class WhipDecorator extends CoffeeDecorator {
    public function getCost(): float {
        return $this->decoratedCoffee->getCost() + 0.5; // Add cost of whipped cream
    }

    public function getDescription(): string {
        return $this->decoratedCoffee->getDescription() . ", Whip";
    }
}

// Usage
$simpleCoffee = new SimpleCoffee();
echo $simpleCoffee->getDescription() . ": $" . $simpleCoffee->getCost() . "<br>";

$milkCoffee = new MilkDecorator($simpleCoffee);
echo $milkCoffee->getDescription() . ": $" . $milkCoffee->getCost() . "<br>";

$whipMilkCoffee = new WhipDecorator($milkCoffee);
echo $whipMilkCoffee->getDescription() . ": $" . $whipMilkCoffee->getCost() . "<br>";
?>
```

## 6. Adapter Pattern

**Purpose**: Allows objects with incompatible interfaces to collaborate. It converts the interface of a class into another interface clients expect, enabling classes to work together that couldn't otherwise due to incompatible interfaces.

**Features**:

- **Target Interface**: Defines the domain-specific interface that Client uses.
- **Adapter**: Adapts the interface Adaptee to the Target interface.
- **Adaptee**: Defines an existing interface that needs adapting.
- **Client**: Collaborates with objects using the Target interface.

**Usage**:

- Integrating legacy systems with new systems.
- Reusing existing classes with incompatible interfaces.
- Implementing third-party libraries or APIs with different interfaces.

**Example**:

```php
<?php
// Target Interface
interface MediaPlayer {
    public function play(string $audioType, string $fileName);
}

// Adaptee
class AudioPlayer {
    public function playAudio(string $fileName) {
        echo "Playing audio file: " . $fileName . "<br>";
    }
}

// Adapter
class MediaAdapter implements MediaPlayer {
    private $audioPlayer;

    public function __construct(AudioPlayer $audioPlayer) {
        $this->audioPlayer = $audioPlayer;
    }

    public function play(string $audioType, string $fileName) {
        if ($audioType === "mp3") {
            $this->audioPlayer->playAudio($fileName);
        } elseif ($audioType === "mp4" || $audioType === "vlc") {
            // Simulating adaptation for other audio formats
            echo "Playing " . $audioType . " file: " . $fileName . "<br>";
        } else {
            echo "Unsupported audio type: " . $audioType . "<br>";
        }
    }
}

// Client
class AudioPlayerClient {
    public function playAudio(string $audioType, string $fileName) {
        $mediaAdapter = null;

        if ($audioType === "mp3") {
            echo "Playing mp3 file: " . $fileName . "<br>";
        } elseif ($audioType === "mp4" || $audioType === "vlc") {
            $mediaAdapter = new MediaAdapter(new AudioPlayer());
            $mediaAdapter->play($audioType, $fileName);
        } else {
            echo "Unsupported audio type: " . $audioType . "<br>";
        }
    }
}

// Usage
$client = new AudioPlayerClient();
$client->playAudio("mp3", "song.mp3");
$client->playAudio("mp4", "movie.mp4");
$client->playAudio("vlc", "video.vlc");
$client->playAudio("avi", "video.avi"); // Unsupported audio type
?>
```

## 7. Facade Pattern

**Purpose**: Provides a unified interface to a set of interfaces in a subsystem. It defines a higher-level interface that makes the subsystem easier to use, hiding its complexities.

**Features**:

- **Facade**: Provides a simplified interface to a larger body of code, such as a library or framework.
- **Subsystem Classes**: Implements functionality but is not directly accessible by clients.
- **Client**: Uses the Facade to access the subsystem without needing to interact directly with its components.

**Usage**:

- Simplifying complex APIs.
- Providing a unified interface to a set of related interfaces.
- Hiding implementation details and promoting loose coupling.

**Example**:

```php
<?php
// Subsystem Classes
class CPU {
    public function freeze() {
        echo "CPU: Freezing processor.<br>";
    }

    public function jump(string $position) {
        echo "CPU: Jumping to position {$position}.<br>";
    }

    public function execute() {
        echo "CPU: Executing command.<br>";
    }
}

class Memory {
    public function load(string $data) {
        echo "Memory: Loading data '{$data}' into memory.<br>";
    }
}

class HardDrive {
    public function read(string $sector, int $size): string {
        echo "Hard Drive: Reading from sector {$sector} with size {$size}.<br>";
        return "Data from sector {$sector}.";
    }
}

// Facade
class ComputerFacade {
    private $cpu;
    private $memory;
    private $hardDrive;

    public function __construct() {
        $this->cpu = new CPU();
        $this->memory = new Memory();
        $this->hardDrive = new HardDrive();
    }

    public function start() {
        $this->cpu->freeze();
        $this->memory->load("BOOT_ADDRESS");
        $this->cpu->jump("BOOT_ADDRESS");
        echo "ComputerFacade: System started and ready to use.<br>";
    }

    public function readData(int $sector, int $size): string {
        return $this->hardDrive->read($sector, $size);
    }
}

// Client
class Client {
    public function run() {
        $computer = new ComputerFacade();
        $computer->start();
        $data = $computer->readData(100, 1024);
        echo "Client: Data read from hard drive: '{$data}'<br>";
    }
}

// Usage
$client = new Client();
$client->run();
?>
```

## 8. Abstract Factory Pattern

**Purpose**: Provides an interface for creating families of related or dependent objects without specifying their concrete classes. It allows the creation of objects to be independent of the main system.

**Features**:

- **Abstract Factory**: Declares interfaces for creating abstract products.
- **Concrete Factory**: Implements the Abstract Factory to create concrete products.
- **Abstract Product**: Declares interfaces for a set of distinct products.
- **Concrete Product**: Implements specific products.

**Usage**:

- Creating families of related products.
- Hiding product creation details from clients.
- Supporting multiple platforms or product variants.

**Example**:

```php
<?php
// Abstract Products
interface Chair {
    public function create(): void;
}

interface Sofa {
    public function create(): void;
}

// Concrete Products
class VictorianChair implements Chair {
    public function create(): void {
        echo "Victorian Chair created.<br>";
    }
}

class VictorianSofa implements Sofa {
    public function create(): void {
        echo "Victorian Sofa created.<br>";
    }
}

class ModernChair implements Chair {
    public function create(): void {
        echo "Modern Chair created.<br>";
    }
}

class ModernSofa implements Sofa {
    public function create(): void {
        echo "Modern Sofa created.<br>";
    }
}

// Abstract Factory
interface FurnitureFactory {
    public function createChair(): Chair;
    public function createSofa(): Sofa;
}

// Concrete Factories
class VictorianFurnitureFactory implements FurnitureFactory {
    public function createChair(): Chair {
        return new VictorianChair();
    }

    public function createSofa(): Sofa {
        return new VictorianSofa();
    }
}

class ModernFurnitureFactory implements FurnitureFactory {
    public function createChair(): Chair {
        return new ModernChair();
    }

    public function createSofa(): Sofa {
        return new ModernSofa();
    }
}

// Client
class Client {
    public function createFurniture(FurnitureFactory $factory) {
        $chair = $factory->createChair();
        $sofa = $factory->createSofa();

        $chair->create();
        $sofa->create();
    }
}

// Usage
$client = new Client();
$victorianFactory = new VictorianFurnitureFactory();
$modernFactory = new ModernFurnitureFactory();

echo "Client: Creating Victorian Furniture<br>";
$client->createFurniture($victorianFactory);

echo "<br>Client: Creating Modern Furniture<br>";
$client->createFurniture($modernFactory);
?>
```
