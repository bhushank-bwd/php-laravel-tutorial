# Multiple Sheet Import

## Package

```json
"maatwebsite/excel": "^3.1",
```

## Controller Function

```php

use App\Imports\MultipleSheetImport;
use Maatwebsite\Excel\Facades\Excel;

$file = $request->file("file");
$import = new MultipleSheetImport();
Excel::import($import, $file);
```

## Main Import File `MultipleSheetImport`

```php
namespace App\Imports;

use App\Imports\Sheets\FirstSheetImport;
use App\Imports\Sheets\SecondSheetImport;
use Maatwebsite\Excel\Concerns\WithMultipleSheets;

class MultipleSheetImport implements WithMultipleSheets
{
    public array $firstSheetRef = [];
    public function sheets(): array
    {
        return [
            0 => new FirstSheetImport($this),
            1 => new SecondSheetImport($this),
        ];
    }
}
```

## FirstSheetImport

```php
namespace App\Imports\Sheets;

use App\Imports\Support\MultipleSheetContext;
use Illuminate\Support\Collection;
use Maatwebsite\Excel\Concerns\ToCollection;
use Maatwebsite\Excel\Concerns\WithHeadingRow;

class FirstSheetImport implements ToCollection, WithHeadingRow
{
    protected $parent;

    public function __construct($parent)
    {
        $this->parent = $parent;
    }
    public function collection(Collection $rows)
    {
        $header = array_keys($rows->first()->toArray());
        foreach ($rows as $row) {
            if (!$row['user_key']) {
                continue;
            }
            $this->parent->firstSheetRef[$row['user_key']] = [];
        }
    }
}
```

## SecondSheetImport

```php
namespace App\Imports\Sheets;

use Illuminate\Support\Collection;
use Maatwebsite\Excel\Concerns\ToCollection;
use Maatwebsite\Excel\Concerns\WithHeadingRow;

class SecondSheetImport implements ToCollection, WithHeadingRow
{

    protected $parent;

    public function __construct($parent)
    {
        $this->parent = $parent;
    }
    /**
     * @param Collection $collection
     */
    public function collection(Collection $collection)
    {
        $header = array_keys($collection->first()->toArray());
        foreach ($collection as $row) {
            if (array_key_exists($row['user_key'], $this->parent->firstSheetRef)) {
                $this->parent->firstSheetRef[$row['user_key']][] = $row->toArray();
            }
        }
        info(__LINE__ . "", $this->parent->firstSheetRef);
    }
}
```
