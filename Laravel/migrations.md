# Migrations

## Create migrations using following command

- It creates migrations in public folder

```bash
php artisan make:migration create_roles_table --path=database/migrations/public
```

- add column to Role table

```bash
php artisan make:migration add_description_to_roles_table --path=database/migrations/public
```

## definition or column list

```php
public function up(): void
{
    Schema::create('roles', function (Blueprint $table) {
        $table->id();
        $table->string("name", 255); // 255 by default
        $table->string("key", 255)->unique();
        $table->enum("status", ["1", "2"])->default('')->comment("1=active 2=deactivated");
        $table->json("features")->nullable();
        $table->softDeletes();
        $table->timestamps();
    });
}
```

## run migrations

```bash
php artisan migrate --path=database/migrations/public
```

```bash
php artisan migrate --path=database/migrations/*
```

```bash
php artisan migrate
```

```bash
php artisan migrate:rollback --path=/database/migrations/your_migration_file.php
```

## Foreign key in Primary users added by following

```php
$table->unsignedBigInteger("role_id");
$table->foreign("role_id")->references("id")->on("roles");
```
## Get All Columns of table in database
```php
$tables = DB::select("
                SELECT table_name
                FROM information_schema.tables
                WHERE table_schema = 'public'
            ");

$result = [];
foreach ($tables as $table) {
    $tableName = $table->table_name;
    $columns = DB::table('information_schema.columns')
        ->selectRaw("
                    CONCAT(
                        column_name,
                        ' ',
                        data_type,
                        CASE
                            WHEN character_maximum_length IS NOT NULL
                            THEN '(' || character_maximum_length || ')'
                            ELSE ''
                        END,
                        CASE
                            WHEN numeric_precision IS NOT NULL
                                AND data_type IN ('numeric', 'decimal')
                            THEN '(' || numeric_precision || ',' || COALESCE(numeric_scale,0) || ')'
                            ELSE ''
                        END,
                        CASE
                            WHEN is_nullable = 'NO'
                            THEN ' NOT NULL'
                            ELSE ' NULL'
                        END,
                        CASE
                            WHEN column_default IS NOT NULL
                            THEN ' DEFAULT ' || column_default
                            ELSE ''
                        END
                    ) AS column_definition
                ")
        ->where('table_schema', 'public')
        ->where('table_name', $tableName)
        ->orderBy('ordinal_position')
        ->pluck('column_definition')
        ->toArray();

    $result[$tableName] = $columns;
}

return response()->json($result);
```

