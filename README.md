# Pipeline operator proposal for PHP 7

This document aims to propose the implementation of the pipeline operator (`|>`) for PHP 7.

### Description

- Associativity: **Left**
- Precedence: **Lowest**
- Token: **T_PIPELINE**

The pipeline operator applies the function on its right side to the value on the left side.
This makes easy to write complex expressions that are evaluated from left-to-right/top-to-bottom.
Many people find the left-to-right order more readable, especially in cases like this where the
filters are higher-order functions that take parameters.

#### Example

##### Old style

```php
function even(int $x): int { return $x % 2 === 0; }
function println($x) { echo $x, PHP_EOL; }

Seq::each("println",
  Seq::map("sqrt",
    Seq::filter("even",
      range(1, 10)
  )
);
```

##### Pipeline operator

```php
function even(int $x): int { return $x % 2 === 0; }
function println($x) { echo $x, PHP_EOL; }

range(1, 10) |> Seq::filter ("even")
             |> Seq::map ("sqrt")
             |> Seq::each ("println");
```

#### Implemented in

- LiveScript
- F#
- Shellscript

### Benefits

- Readability
- No need to nest functions

### Problems

- Parameterization of built-in functions that differ, such as `array_map` and `array_filter`.
