# Multiple Sheet Export

## Package

```json
"maatwebsite/excel": "^3.1",
```

## Controller Function

```php

use App\Exports\MultipleSheetExport;
use Maatwebsite\Excel\Facades\Excel;

$fileNameDownload = 'user-report-' . now()->format('d-m-Y') . '.xlsx';

$fileName = 'export/user-report-' . now()->format('d-m-Y') . '.xlsx';

$data = [
    "users" => [
        ["id" => 1, "name" => "John Doe", "email" => "john@example.com"],
        ["id" => 2, "name" => "Jane Smith", "email" => "jane@example.com"],
        ["id" => 3, "name" => "Bob Johnson", "email" => "bob@example.com"],
    ]
];
Excel::store(
    new MultipleSheetExport($data),
    $fileName,
    'private'
);


return response()->json(['message' => 'File exported successfully', 'file_url' => getFileTemporaryURL($fileName)]);


return Excel::download(new MultipleSheetExport($data), $fileNameDownload);
```

## Main Export File `MultipleSheetExport`

```php
namespace App\Exports;

use App\Exports\Sheet\FirstSheetExport;
use App\Exports\Sheet\SecondSheetExport;
use Maatwebsite\Excel\Concerns\WithMultipleSheets;

class MultipleSheetExport implements WithMultipleSheets
{
    protected array $data;

    public function __construct(array $data)
    {
        $this->data = $data;
    }
    public function sheets(): array
    {

        return [
            new FirstSheetExport($this->data),
            new SecondSheetExport($this->data),
        ];
    }
}
```

## FirstSheetExport

```php
namespace App\Exports\Sheet;

use Maatwebsite\Excel\Concerns\FromArray;
use Maatwebsite\Excel\Concerns\ShouldAutoSize;
use Maatwebsite\Excel\Concerns\WithTitle;
use Maatwebsite\Excel\Concerns\WithHeadings;
use Maatwebsite\Excel\Concerns\WithStyles;
use PhpOffice\PhpSpreadsheet\Worksheet\Worksheet;

class FirstSheetExport implements FromArray, WithTitle, WithHeadings, WithStyles, ShouldAutoSize
{
    protected array $data;

    public function __construct(array $data)
    {
        $this->data = $data;
    }

    public function array(): array
    {
        return $this->data['users'];
    }

    public function headings(): array
    {
        return [
            'ID',
            'Name',
            'Email',
        ];
    }

    public function title(): string
    {
        return 'Users';
    }

    public function styles(Worksheet $sheet)
    {
        return [
            1 => [
                'font' => [
                    'bold' => true,
                    'size' => 14
                ],
                'fill' => [
                    'fillType' => \PhpOffice\PhpSpreadsheet\Style\Fill::FILL_SOLID,
                    'startColor' => [
                        'rgb' => '4F81BD'
                    ]
                ],
                'alignment' => [
                    'horizontal' => \PhpOffice\PhpSpreadsheet\Style\Alignment::HORIZONTAL_CENTER,
                    'vertical' => \PhpOffice\PhpSpreadsheet\Style\Alignment::VERTICAL_CENTER,
                ],
            ]
        ];
    }
}
```

## SecondSheetExport

```php
namespace App\Exports\Sheet;

use Maatwebsite\Excel\Concerns\FromArray;
use Maatwebsite\Excel\Concerns\WithTitle;
use Maatwebsite\Excel\Concerns\WithHeadings;
use Maatwebsite\Excel\Concerns\WithStyles;
use PhpOffice\PhpSpreadsheet\Worksheet\Worksheet;

class SecondSheetExport implements FromArray, WithTitle, WithHeadings, WithStyles
{
    /**
     * @return \Illuminate\Support\Collection
     */
    protected array $data;

    public function __construct(array $data)
    {
        $this->data = $data;
    }

    public function array(): array
    {
        return $this->data['users'];
    }

    public function headings(): array
    {
        return [
            'ID',
            'Name',
            'Email',
        ];
    }

    public function title(): string
    {
        return 'Users';
    }

    public function styles(Worksheet $sheet)
    {
        return [
            1 => [
                'font' => [
                    'bold' => true,
                    'size' => 14
                ]
            ]
        ];
    }
}
```
