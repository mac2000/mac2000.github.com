---
layout: post
title: PowerShell ConvertTo-Markdown
tags: [powershell, convertto-markdown, function]
---

[Found](https://gist.github.com/GuruAnt/4c837213d0f313715a93) good idea of how to produce markdown tables right from powershell.

Just need some more nice output with aligned columns like this:

```
Region        | Category   | Count
------------- | ---------- | -----
UK            | java       | 35
UK            | javascript | 34
CA            | javascript | 27
CA            | java       | 27
NY            | java       | 23
allows remote | javascript | 22
Deutschland   | java       | 21
UK            | c#         | 21
CA            | python     | 21
UK            | php        | 20
```

Here is what I have:

```powershell
<#
.Synopsis
    Converts a PowerShell object to a Markdown table.
.EXAMPLE
    $data | ConvertTo-Markdown
.EXAMPLE
    ConvertTo-Markdown($data)
#>
Function ConvertTo-Markdown {
    [CmdletBinding()]
    [OutputType([string])]
    Param (
        [Parameter(
            Mandatory = $true,
            Position = 0,
            ValueFromPipeline = $true
        )]
        [PSObject[]]$collection
    )

    Begin {
        $items = @()
        $columns = @{}
    }

    Process {
        ForEach($item in $collection) {
            $items += $item

            $item.PSObject.Properties | %{
                if(-not $columns.ContainsKey($_.Name) -or $columns[$_.Name] -lt $_.Value.ToString().Length) {
                    $columns[$_.Name] = $_.Value.ToString().Length
                }
            }
        }
    }

    End {
        ForEach($key in $($columns.Keys)) {
            $columns[$key] = [Math]::Max($columns[$key], $key.Length)
        }

        $header = @()
        ForEach($key in $columns.Keys) {
            $header += ('{0,-' + $columns[$key] + '}') -f $key
        }
        $header -join ' | '

        $separator = @()
        ForEach($key in $columns.Keys) {
            $separator += '-' * $columns[$key]
        }
        $separator -join ' | '

        ForEach($item in $items) {
            $values = @()
            ForEach($key in $columns.Keys) {
                $values += ('{0,-' + $columns[$key] + '}') -f $item.($key)
            }
            $values -join ' | '
        }
    }
}
```

As usual save this to `%USERPROFILE%\Documents\WindowsPowerShell\Modules\ConvertTo-Markdown\ConvertTo-Markdown.psm1`

Note that folder containing psm1 file should have same name otherwise module will not be autoloaded

Note to get modules autoload you should run PowerShell as Administrator

Otherwise you must load module manually by typing `Import-Module $Home\Documents\WindowsPowerShell\Modules\ConvertTo-Markdown\ConvertTo-Markdown.psm1`
