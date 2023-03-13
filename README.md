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
