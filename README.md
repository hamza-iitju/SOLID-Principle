# Solid Principle

## What is SOLID design principles
SOLID principles are the set of software design principles used by software engineers in object-oriented software development to follow, scale and maintain a proper structure to the codes & programming. SOLID is a popular set of SOLID is an acronym that stands for five key design principles: single responsibility principle, open-closed principle, Liskov substitution principle, interface segregation principle, and dependency inversion principle.

## Why need SOLID Principles
SOLID principles aim at reducing the dependencies of software engineers. As if they know and understand the basic architecture & design principles, it'll be easier for them to change or edit codes and alter a few changes if required.

Additionally, SOLID principles enable engineers to make designs easier to understand, maintain, and extend. Ultimately, using these design principles makes it easier for software engineers to avoid issues and to build adaptive, effective, and agile software.

### The SOLID Principles are

>S — Single Responsibility
>O — Open-Closed
>L — Liskov Substitution
>I — Interface Segregation
>D — Dependency Inversion

## S — Single Responsibility Principle
It states that a class should do one thing and therefore it should have only a single reason to change. If a Class has many responsibilities, it increases the possibility of bugs, because making changes to one of its responsibilities, could affect the other ones without you knowing. One potential change around database logic, logging logic, or attribute can affect the objects of a specific class.



```php
<?php
namespace App\Solid;
use Illuminate\Support\Facades\DB
class  SaleReports  extends  Controller
{
	public function export($startDate, $endDate, $format = 'csv')
	{
		DB::table('sales')
			->whereBetween('created_at', [$startDate, $endDate])
			->latest()
			->get();
		if($format == 'pdf'){
			return 'PDF format';
		}
		if($format == 'json'){
			return 'JSON format';
		}
		return 'CSV format';
	}
}
```

In this example, the  `export` method has two responsibilities, one to fetch data from database and another to create export format. If export format is multiple, then we need to check which format is being generated that is confusing for another developer.

We can separate the method for two responsibilities, which one is fetching data from database and another is export that data into different format.


```php
<?php
namespace App\Solid;
use Illuminate\Support\Facades\DB
class  SaleReports  extends  Controller
{
	public function between($startDate, $endDate)
	{
		return DB::table('sales')
			->whereBetween('created_at', [$startDate, $endDate])
			->latest()
			->get();
	}
	public function pdfExport($data)
	{
		return 'PDF format';
	}
	public function jsonExport($data)
	{
		return 'JSON format';
	}
	public function export($format = 'csv')
	{
		$data = $this->between($startDate, $endDate);
		if($format == 'pdf'){
			return $this->pdfExport($data);
		}
		if($format == 'json'){
			return $this->jsonExport($data);
		}
		return 'CSV format';
	}
}
```
Still `SaleReports` class has multiple job and also violate the priciples. We can resolve this with a dedicated class. Here we can see `pdfExport, jsonExport` methods that shouldn't be in `SaleReports` class. If we add another format like `xlsExport` then we have to add another responsibility in that class.

So we can extract the `pdfExport, jsonExport` method into the separate class like

```php
<?php
namespace App\Solid;
class PdfExport
{
	public function export($data)
	{
		return 'PDF export';
	}
}
```

```php
<?php
namespace App\Solid;
class JsonExport
{
	public function export($data)
	{
		return 'json export';
	}
}
```
```php
<?php
namespace App\Solid;
class CsvExport
{
	public function export($data)
	{
		return 'CSV export';
	}
}
```

Finally the `SaleReports` class
```php
<?php
namespace App\Solid;
use Illuminate\Support\Facades\DB
class  SaleReports  extends  Controller
{
	public function between($startDate, $endDate)
	{
		return DB::table('sales')
			->whereBetween('created_at', [$startDate, $endDate])
			->latest()
			->get();
	}
}
```

Now we can see the `SaleReports` class has single responsibility to fetch data from database and the other class like `PdfExport, JsonExport, CsvExport` also has a single responsibility to export data into a specific format.

We can add more than one method that is related to `SaleReports` class like

```php
<?php
namespace App\Solid;
use Illuminate\Support\Facades\DB
class  SaleReports  extends  Controller
{
	public function between($startDate, $endDate)
	{
		return DB::table('sales')
			->whereBetween('created_at', [$startDate, $endDate])
			->latest()
			->get();
	}
	public function monthReport($month)
	{
		return DB::table('sales')
			->whereMonth('created_at', $month)
			->latest()
			->get();
	}
	public function yearReport($year)
	{
		return DB::table('sales')
			->whereYear('created_at', $year)
			->latest()
			->get();
	}
}
```

### For laravel example
We can apply this priciple into laravel controller for validating some data. If we validate some data before store or update we can create a dedicated class for that like validation class and keep all validation rules there.

Assume that we have a controller like `PostController` and it has a method like store that have too many responsibilities to handle. Then we can separate the responsibilities into different class. For example:

```php
use App\Models\Post;
use Illuminate\Http\Request;
class PostController extend Controller
{
	public function store(Request $request)
	{
		$request->validate([
			'title' => 'required|max:255',
			'body' => 'required'
		])
		Post::create($request->only('title', 'body'));
		// Image upload
		// Email Sending notification
		return view('app/post/index')->withSuccess('Created Successfully');
	}
}
```
In this above store method, there are 4 responsibilities [validating, storing, image uploading, notification sending] that are violating the principle. So we need to separate the responsibilities from that class into a different class.

#### For validation
The validation should be into the dedicated post request class, for that we can create a class with command `php artisan make:request StorePostRequest`. Now we can move the validation into the `StorePostRequest` class.

```php
namespace App\Http\Request;
use Illuminate\Foundation\Http\FormRequest;
class StorePostRequest extend FormRequest
{
	public function authorize()
	{
		return false;
	}
	public function rule()
	{
		return [
			'title' => 'required|max:255',
			'body' => 'required'
		];
	}
}
```

For the other responsibilities, we can create a service for `PostService` into the service directory.

```php
namespace App\Services;
use App\Models\Post;
use App\Http\Request\StorePostRequest;
class PostService
{
	public static function create(StorePostRequest $request)
	{
		Post::create($request->only('title', 'body');
		// Image upload
		// Email Sending notification
	}
}
```
In the above `PostService` class, we can add other similar responsibilities like image upload, email notification send etc.
Then the controller is now in this format like:

```php
use App\Models\Post;
use App\Http\Request\StorePostRequest;
use Illuminate\Http\Request;
class PostController extend Controller
{
	public function store(StorePostRequest $request)
	{
		PostService::create($request);
		return view('app/post/index')->withSuccess('Created Successfully');
	}
}
```

Now the controller is clean and very handy to extend or readable. 

In this principle, it is easy to understand the code cause every class has a single responsibility, so it can easy to debug or understand for other developers.





## O — Open-Closed Principle
The Open-Closed Principle suggests that classes should be open to extension and closed to modification. Sometimes, we need to add certain functions to the existing class to perform additional tasks. So, according to the Open-Closed Principle, We should add new functionality without touching the existing code for the class. This is because whenever we modify the existing code, we risk creating potential bugs. So we should avoid touching the tested and reliable (mostly) production code if possible. 

For this reason, it is called open for extension (Add functionality) but closed for modification (Couldn't update any existing function) In simple words, this principle aims to extend a Class’s behavior without changing the existing behavior of that Class.

For Example:
If we have a project like calculate area then we can write the code like:


```php
<?php
namespace App\Solid;
class Rectengle
{
	public $width;
	public $height;
	public function __construct($width, $height)
	{
		$this->width = $width;
		$this->height = $height;
	}
}
```
In the 1st principle, we learned that a class has a single responsibility, so we can extract another class for area calculator:

```php
<?php
namespace App\Solid;
class AreaCalculator
{
	public $width;
	public $height;
	public function totalArea(array $rectengles)
	{
		$area = 0;
		foreach($rectengles as $rectengle) {
			$area += $rectengle->width * $rectengle->height;
		}
		return $area;
	}
}
```
`web.php` for route
```php
Route::get('/', function(){
	return (new AreaCalculator)->totalArea(new Rectengle(10,20));
})
```

If we calculate the radius of a circle in that project:
```php
<?php
namespace App\Solid;
class Circle
{
	public $radius;
	public function __construct($radius)
	{
		$this->radius = $radius;
	}
}
```
Now if we pass the Circle 
```php
Route::get('/', function(){
	return (new AreaCalculator)->totalArea(new Circle(10));
})
```
It is not going to work because the totalArea method is written only for rectangle. So we need to modify the method that is violating the 2nd principle.

Instead of passing rectangle into the totalArea method of AreaCalculator class, we can change name shapes like;

```php
<?php
namespace App\Solid;
class AreaCalculator
{
	public $width;
	public $height;
	public function totalArea(array $shapes)
	{
		$area = 0;
		foreach($shapes as $shape) {
			if($shape instanceof Rectangle){
				$area += $shape->width * $shape->height;
			}
			$area += $shape->radius * $shape->radius * pi();
		}
		return $area;
	}
}
```

We see that if we add another area calculator then we have to add another if condition that is violating the principle.

#### So we can update the code like this:

```php
<?php
namespace App\Solid;
class Rectengle
{
	public $width;
	public $height;
	public function __construct($width, $height)
	{
		$this->width = $width;
		$this->height = $height;
	}
	public function area()
	{
		return $this->width * $this->height;
	}
}
```
```php
<?php
namespace App\Solid;
class Circle
{
	public $radius;
	public function __construct($radius)
	{
		$this->radius = $radius;
	}
	public function area()
	{
		return $this->radius * $this->radius * pi();
	}
}
```

```php
<?php
namespace App\Solid;
class AreaCalculator
{
	public $width;
	public $height;
	public function totalArea(array $shapes)
	{
		$area = 0;
		foreach($shapes as $shape) {
			$area += $shape->area()
		}
		return $area;
	}
}
```
It is not done yet. The open closed principle suggests that we should separate the instantiate behavior behind an interface.

### Why we need interface
if we add an another area calculator for triangle then we have to add another class like:
```php
<?php
namespace App\Solid;
class Triangle
{
	public $base;
	public $height;
	public function __construct($base, $height)
	{
		$this->base = $base;
		$this->height = $height;
	}
	public function area()
	{
		return ($this->base * $this->height)/2;
	}
}
```
Let's say in the above class is added by other developer, and he doesn't know the code structure. He changes the method name like `getArea()` then the code runs error. In the `AreaCalculator` class `getArea()` method doesn't exist.

That kind of problem is solved by the interface. Let's see.
### Final code for open closed principle
```php
<?php
namespace App\Solid;
class ShapeInterface
{
	public function area();
}
```
```php
<?php
namespace App\Solid;
class Rectengle implements ShapeInterface
{
	public $width;
	public $height;
	public function __construct($width, $height)
	{
		$this->width = $width;
		$this->height = $height;
	}
	public function area()
	{
		return $this->width * $this->height;
	}
}
```
```php
<?php
namespace App\Solid;
class Circle implements ShapeInterface
{
	public $radius;
	public function __construct($radius)
	{
		$this->radius = $radius;
	}
	public function area()
	{
		return $this->radius * $this->radius * pi();
	}
}
```
```php
<?php
namespace App\Solid;
class Triangle implements ShapeInterface
{
	public $base;
	public $height;
	public function __construct($base, $height)
	{
		$this->base = $base;
		$this->height = $height;
	}
	public function area()
	{
		return ($this->base * $this->height)/2;
	}
}
```

```php
<?php
namespace App\Solid;
class AreaCalculator
{
	public $width;
	public $height;
	public function totalArea(ShapeInterface $shapes)
	{
		$area = 0;
		foreach($shapes as $shape) {
			$area += $shape->area()
		}
		return $area;
	}
}
```
```php
Route::get('/', function(){
	return (new AreaCalculator)->totalArea(
		new Circle(10),
		new Triangle(10, 20),
		new Rectangle(10, 20)
	);
})
```


This is an example of using open closed principle.

Another example we can implement open closed principle is `PaymentService`. We have different types of payment service like `paypal`, `stripe` etc. Here we can make an interface like `PaymentMethodInterface` and declare a method like `makePayment()`. Then create different class for each payment method like `paypal`, `stripe` that implements `PaymentMethodInterface`. For this reason we have to implement method `makePayment()` in every payment class. Then we can pass the dependency injection into the `PaymentService` class and call `makePayment()` method.

## Liskov substitution principle (LSP)
The Liskov Substitution Principle (LSP) states that **objects of a superclass should be replaceable with objects of its subclasses without breaking the application**. 

### Liskov substitution principle (LSP) rules.
1. Parameter types must match in both child and parent classes
2. Method return type must match
3. Preconditions cannot be strengthened by the child's class
4. Postconditions cannot be weakened in the child's class
5. Exceptions thrown by the child method must be the same as from an exception thrown by the parent method.
6. Invariants of a superclass must be preserved

### Violation of Liskov substitution principle (LSP)
There is a checklist to determine whether you are violating Liskov.

Check list:
- **Parameter types must match in both child and parent classes:**  When overriding a method, child class & parent class method must same parameter.
- **Method return type must match:** When overriding a method, child class & parent class method must return same type.
>For Example
```php
<?php
interface Database 
{
    public function selectQuery(string $sql): array;
}
class SQLiteDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // sqlite specific code
        return $result;
    }
}

class MySQLDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // mysql specific code
        return ['result' => $result]; // This violates LSP !
    }
}
```
-   **Preconditions cannot be strengthened by the child's class**: Assume your base class works with a member int. Now, your subtype requires that int to be positive. This is strengthened pre-conditions, and now any code that worked perfectly fine before with negative int is broken.
>For Example

**Violation:**
```php
<?php
class AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// email configuration to send email
	}
}
class XyzSendEmailService extends AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// precondition
		if(!$to){ return; } // This is not allowed cause this condition is not present in the parent class
		
		// email configuration to send email
	}
}
```
**Validate:**
```php
<?php
class AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// precondition
		if(!$to){ return; }
		// email configuration to send email
	}
}
class XyzSendEmailService extends AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// precondition
		if(!$to){ return; } // This is allowed cause this condition is present in the parent class
		
		// email configuration to send email
	}
}
```

-   **Post-conditions cannot be weakened**: Assume your base class required all connections to the database should be closed before the method returned. In your subclass, you overrode that method and left the connection open for further reuse. You have weakened the post-conditions of that method.

>For Example

**Violation:**
```php
<?php
class AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// email configuration to send email
		
		// postcondition
		if(!$isEmailNotSent){ return; } // This is not allowed cause this condition is not present in the child class
	}
}
class XyzSendEmailService extends AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{	
		// email configuration to send email
	}
}
```
**Validate:**
```php
<?php
class AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{
		// email configuration to send email
		
		// postcondition
		if(!$isEmailNotSent){ return; } // This is allowed cause this condition is present in the child class
	}
}
class XyzSendEmailService extends AbcSendEmailService
{
	public function sendEmail($to, $subject, $message)
	{	
		// email configuration to send email
		
		// postcondition
		if(!$isEmailNotSent){ return; }
	}
}
```

-   **No new exceptions should be thrown in derived class**: If your base class threw ArgumentNullException then your subclasses were only allowed to throw exceptions of type ArgumentNullException or any exceptions derived from ArgumentNullException. Throwing IndexOutOfRangeException is a violation of Liskov.
    
-   **Invariants must be preserved**: The most difficult and painful constraint to fulfill. Invariants are sometimes hidden in the base class and the only way to reveal them is to read the code of the base class. Basically you have to be sure when you override a method anything unchangeable must remain unchanged after your overridden method is executed. The best thing I can think of is to enforce these invariant constraints in the base class but that would not be easy.
>For Example

```php
<?php
class AbcSendEmailService
{
	public $secretKey;
	public function __construct($secretKey)
	{
		$this->secretKey = $secretKey; // This secret key must not change from child class or other methods
	}
	public function sendEmail($to, $subject, $message)
	{
		// Email config
	}
}
```
    
-   **History Constraint**: When overriding a method you are not allowed to modify an unmodifiable property in the base class. Take a look at these code and you can see Name is defined to be unmodifiable (private set) but SubType introduces new method that allows modifying it (through reflection):
    
```php
class SuperType
{
	public $name; 
	public function getName()
	{
		return $this->name
	}
	private function setName($name){
		return $this->name = $name;
	}
}
class SubType extends SuperType
{
	public ChangeName($newName)
	{
		var propertyType = $this->setName($newName); 
		// This is the violation of this rules. 
		//We can't change the value of name from child class of a private set.
	}
} 
```


## Interface Segregation

Segregation means keeping things separated, and the Interface Segregation Principle is about separating the interfaces. The principle emphasizes having one general-purpose interface rather than many client-specific interfaces. Clients should not be forced to depend on methods that they do not use. Because, when a Class is required to perform actions that are not useful, it is wasteful and may produce unexpected bugs if the Class cannot perform those actions. It is advisable for software engineers to start by building a new interface and then let the class implement multiple interfaces as needed, rather than using an existing interface and adding new methods.

>For Example
```php
<?php
interface Bird
{
	public function speak();
	public function fly();
}

class Duck implements Bird
{
	public function speak(){
		return "Duck speaks";
	}
	public function fly(){
		return "Duck flies";
	}
}
class Ostrich implements Bird
{
	public function speak(){
		return "Duck speaks";
	}
	public function fly(){
		// no fly
	}
}
```

Here we can see that ostrich can't fly, and `fly()` method is not needed. But for implementing interface we have to add this method. But this is not usable and that is the violation of this principle.

We can rearrange the **interface** like this:
```php
<?php
interface Speakable
{
	public function speak();
}

interface Flyable
{
	public function fly();
}

class Duck implements Speakable, Flyable
{
	public function speak(){
		return "Duck speaks";
	}
	public function fly(){
		return "Duck flies";
	}
}
class Ostrich implements Speakable
{
	public function speak(){
		return "Duck speaks";
	}
}
```

We can segregate the interface into two interface like `Speakable & Flyable` so that we can use it where it is needed, and no unusable methods are there. We can add more related method into the interface instead of one method.

In real life example, Laravel use multiple interface into the `Model` class that is only for used specific purpose. 
