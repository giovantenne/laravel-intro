# ITIS A. Meucci
## Introduzione a Laravel
### Indirizzo server: 172.16.102.3
Day 1
------------
1. INTRODUZIONE A LARAVEL
2. INSTALLAZIONE DI LARAVEL TRAMITE COMPOSER
3. DEFINIZIONE DELLE ROTTE
4. PASSARE DATI ALLE VISTE
5. DELEGARE AI CONTROLLER
6. FILE DI LAYOUT

------------
* Creare un nuovo progetto
```bash
composer create-project --prefer-dist laravel/laravel prova
```
* Avviare il server locale
```bash
php artisan serve
```
* Avviare il server locale sulla porta 3000 consentendo connessioni remote 
```bash
php artisan serve --host 0.0.0.0 --port 3000
```
* Visualizzare il funzionamento dell'installazione aprendo il browser sull'indirizzo del server. (es: http://172.16.102.3:3000)

------------
* Editare il file app/Http/routes.php
* Ritornare una semplice stringa
```php
// app/Http/routes.php
Route::get('/', function () { 
  return "FooBar"; 
});
```
* Aggiungere una route per la vista 'about' e creare il file about.blade.php
```php
// app/Http/routes.php
Route::get('about', function () { 
  return view('about'); 
});
```
```php
//resources/views/about.blade.php
<h1>About page</h1>
```

* Spostare la vista nella sottocartella 'pages' e aggiornare il file route.php
```php
// app/Http/routes.php
return view('pages.about'); //per chiamare la vista in resources/views/pages/about.blade.php
```
------------
* Passare dati alle viste dal file route.php (tutti gli esempi sotto sono equivalenti):
```php
// app/Http/routes.php
Route::get('/', function(){
  $people = ['pippo', 'ciccio', 'nino'];
  // return view('welcome', ['people' => $people]);
  return view('welcome', compact('people'));
  // return view('welcome')->with('people', $people);
  // return view('welcome')->withPeople($people);  
});

```
in questo modo viene passata alla vista una variabile "people" contentente il contenuto della variabile locale $people, ovvero un array

* Mostrare i dati nella vista
```php
// resources/views/welcome.blade.php
<?php foreach ($people as $person) : ?>
  <li><?= $person; ?></li>
<?php endforeach; ?>
```
* Utilizzare la sintassi blade;
```php
// resources/views/welcome.blade.php
@foreach ($people as $person)
  <li>{{ $person }}</li>
@endforeach

@if (empty($people))
  There are not people.
@endif
```
------------

* Modificare il file route.php per utilizzare un controller
```php
// app/Http/routes.php
Route::get('/', 'PagesController@home');
```
* Aggiornare il browser e visualizzare l'errore di controller mancante
* Visualizzare i controller esistenti dentro app/Http/Controllers

* Utilizzare il comando "php artisan" per visualizzare l'help generale e per generare un controller
```bash
php artisan help make:controller
php artisan make:controller PagesController
```

* Nel controller appena creato definire il metodo 'home' con la logica che stava nelle route
```php
// app/Http/Controllers/PagesController.php
public function home()
{
  $people = ['pippo', 'ciccio', 'nino'];
  return view('welcome', ['people' => $people]);
}
```
* Aggiungere anche la route 'about' con il relativo metodo nel controller ritornando una stringa o una vista

------------

* Creare un layout file in resources/views/layout.blade.php ed aggiungere lo yield
```php
// resources/views/layout.blade.php
<html>
  <head></head>
  <body>
    @yield('content')
  </body>
</html>
```

* Modificare la vista welcome:
```php
// resources/views/welcome.blade.php
@extends('layout')
@section('content')
    The welcome page goes here
@stop
```
* Fare la stessa cosa con la vista About
* Aggiungere la sezione header e footer


Day 2
------------
1. INTRODUZIONE A ROTTE REST
2. MIGRAZIONI
3. ACCESSO AI DATI SENZA ELOQUENT
4. MODELLI CON L'ORM ELOQUENT



------------

* Di seguito sono elencate le rotte per una risorsa REST
```php
Route::get('cards', 'CardsController@index'); //per mostrare tutte le Cards
Route::get('cards/create', 'CardsController@create'); //per mostrare il form di creazione
Route::post('cards', 'CardsController@store'); //per salvare la Card
Route::get('cards/1', 'CardsController@show'); //per mostrare la Card 1
Route::post('cards/1/edit', 'CardsController@edit'); //per mostrare il form di modifica per la Card 1
Route::put('cards/1', 'CardsController@update'); //per salvare la modifica alla Card 1
Route::delete('cards/1', 'CardsController@destroy'); //per eliminare la Card 1
```
* Creare un controller per la risorsa:
```bash
php artisan make:controller CardsController
```
* In app/Http/Controllers/CardsController.php inserire il seguente codice:
```php
// app/Http/Controllers/CardsController.php
public function index(){
  return view('cards.index'); // creare la sottocartella cards per avere tutte le viste in resources/views/cards/index.blade.php
}
```
------------

* In config/database.php modificare il default in 'sqlite'
```php
// config/database.php
'default' => env('DB_CONNECTION', 'sqlite'),
```
* Creare il file vuoto per il database: 
```bash
touch database/database.sqlite
```

* Eliminare tutte le migrazioni presenti in database/migrations
```bash
rm database/migrations/*
```
* Creare una nuova migrazione
```bash
php artisan help make:migration
php artisan make:migration create_cards_table --create=cards
```
* Nella corpo della migrazione appena creata aggiungere la seguente direttiva:
```php
// database/migrations/xxxx_xx_xx_xxxxxx_create_cards_table.php
$table->string('title');
```
* Migrare il database con:
```bash
php artisan migrate
```
* Utilizzare il tinker
```bash
php artisan tinker
```
* Impartire qualche comando:
```
2+2
echo 'foobar'
```
* Popolare e interrogare il database:
```
DB::table('cards')->insert(['title' => 'My New Card', 'created_at' => new DateTime, 'updated_at' => new DateTime]);
DB::table('cards')->get();
DB::table('cards')->where('title', 'My New Card')->first();
DB::table('cards')->where('title', 'My New Card')->delete();
DB::table('cards')->insert(['title' => 'My New Card', 'created_at' => new DateTime, 'updated_at' => new DateTime]);
DB::table('cards')->insert(['title' => 'My Second New Card', 'created_at' => new DateTime, 'updated_at' => new DateTime]);
```

------------

* In CardsController.php nel metodo index() inserire
```php
// app/Http/Controllers/CardsController.php
$cards = \DB::table('cards')->get();
return view('cards.index', ['cards' => $cards]);
```
oppure l'equivalente più compatto
```php
// app/Http/Controllers/CardsController.php
$cards = \DB::table('cards')->get();
return view('cards.index', compact('cards'));
```
* Per rimuovere lo slash da \DB  inserire il modulo DB con la seguente direttiva
```php
// app/Http/Controllers/CardsController.php
use DB;
```
* Creare la vista
```php
// resources/views/cards/index.blade.php
@extends('layout')
@section('content')
    <h1>All Cards</h1> 
    @foreach ($cards as $card)
        <div>
            {{ $card->title }}
        </div>
    @endforeach
@stop
```

------------

* Utilizzare l'ORM Eloquent e creare il modello Card
```bash
php artisan make:model Card
```
viene creato il file app/Card.php

* Tornare nel controller e sostituire
```php
// app/Http/Controllers/CardsController.php
$cards = \DB::table('cards')->get();
```
con 
```php
$cards = \App\Card::all();
```

* Semplificare importando il modulo
```php
// app/Http/Controllers/CardsController.php
use App\Card;
...
$cards = Card::all();
```

* Aggiungere la route per la show
```php
// app/Http/routes.php
Route::get('cards/1', 'CardsController@show'); //per mostrare la Card 1
```
* parametrizzare con
```php
Route::get('cards/{card}', 'CardsController@show');
```
* modificare il controller
```php
// app/Http/Controllers/CardsController.php
public function show($card)
{
    return $card;
}
```

* riaprire la console tinker per fare qualche esperimento
```bash
php artisan tinker 
```

```bash
DB::table('cards')->get();
```
oppure 
```bash
App\Card::all()
```

* accedere a http://localhost:8000/cards/2
* modificare la show del controller con 
```php
// app/Http/Controllers/CardsController.php
public function show($id)
{
  $card = Card::find($id);
  return $card:
}
```

* aggiornare il browser e verificare che se il controller ritorna un record e non una view Laraver costruisce il JSON

* ritornare una vista:
```php
// app/Http/Controllers/CardsController.php
return view('cards.show', compact('card'));
```
creare la vista show.blade.php
```php
// resources/views/cards/show.blade.php
@extends('layout')
@section('content')
    <h1>{{ $card->title }}</h1>
@stop
```
* visualizzare la pagina con il browser

* modificare il controller per utilizzare l'auto fetch 
```php
// app/Http/Controllers/CardsController.php
public function show(Card $card)
{
    return $card;
    // return view('cards.show', compact('card'));
}
```
per far funzionare la "magia" il paramtero nelle rotte si deve chiamar con lo stesso nome del parametro passato all'azione del controller
