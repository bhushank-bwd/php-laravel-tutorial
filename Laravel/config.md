# Laravel 11 Configuration

## Database Configuration

1. in .env file set following

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_11
DB_USERNAME=root
DB_PASSWORD=
```

- Note: Laravel 11 has by default sqlite connection

2. In `config\database.php` add another mysql_2 database connection

```php
'mysql_2' => [
    'driver' => 'mysql',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => 'laravel_11_2',
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

- Change value as per your connection

- Usage

```php
$mysql2Posts = DB::connection("mysql_2")->table("posts")->get();
$mysql1Posts = DB::connection("mysql")->table("posts")->get();
$mysql1Posts = POST::on("mysql")->get();
$mysql1Post = new POST;
$mysql1Post->setConnection("mysql");
// other properties
$mysql1Post->save();
```

3. Dynamically Set Connection

```php
Config::set('database.connections.branch.database', $dbRow->db_name);
Config::set('database.connections.branch.username', $dbRow->db_username);
Config::set('database.connections.branch.password', decrypt($dbRow->db_password));

DB::purge('branch');
DB::reconnect('branch');
DB::setDefaultConnection("branch");
DB::connection("branch")->beginTransaction();
DB::connection("branch")->commit();
DB::connection("branch")->rollback();
```
- `branch` is key from `config\database.php` array

## MongoDB connection

### install laravel package for mongodb

```bash
composer require mongodb/laravel-mongodb
```

### Mongo DB for php

1.  Download Mongo DB from
    [Github](https://github.com/mongodb/mongo-php-driver/releases)
    `php_mongodb-1.19.3-8.2-ts-x64.zip`

2.  In `E:\xampp\php\php.ini` add extension `extension=php_mongodb.dll`

- copy php_mongodb.pdb and php_mongodb.dll in `E:\xampp\php\ext`
- php_mongodb.dll is file name in above zip
- check in phpmyinfo

3. Mongo DB in Laravel

- `config\database.php`

```php
'mongodb' => [
    'driver' => 'mongodb',
    'dsn' => env('DB_URI', 'mongodb://localhost:27017'),
    'database' => 'marketplace',
],
```

4. create model store

```bash
php artisan make:model Store
```

- use MongoDB model class

  ```php
  use MongoDB\Laravel\Eloquent\Model;
  class Store extends Model
    {

    }
  ```

5. CRUD

```php
Route::post("create", function (Request $request) {
    $store = new Store();
    $store->name = $request->name;
    $store->description = $request->description;
    $store->owner = $request->owner;
    $store->save();
    return response()->json($store->id);
});
Route::get("/delete/{id}", function ($id) {
    $store = Store::find($id);
    print_r($store->delete());
});
Route::post("/edit/{id}", function (Request $request, $id) {
    $store = Store::find($id);
    $store->description = $request->description;
    print_r($store->save());
});
Route::get("/get/{id}", function (Request $request, $id) {
    $store = Store::find($id);
    print_r($store->toArray());
});
Route::get("/list", function () {
    $store = Store::get();
    return response()->json($store->toArray());
});
```

## Env variable

1. .env file

```bash
CUSTOM_NAME="New Custom Name"
```

2. in `config\app.php`

```php
'custom_name' => env('CUSTOM_NAME',"default_name"),
```

3. Usage

```php
$custom_name = config("app.custom_name");
```
