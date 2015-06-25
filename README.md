# Pipeline operator proposal for PHP 7

This document aims to propose the implementation of the pipeline operator (`|>`) for PHP 7.

### Description

- Associativity: **Left**
- Precedence: **Lowest**
- Token: **T_PIPELINE**

The pipeline operator applies the function on its right side to the value on the left side.
This makes easy to write complex expressions that are evaluated from left-to-right/top-to-bottom.
Many people find the left-to-right order more readable, especially in cases like this where the
filters are higher-order functions that take parameters. When called with functions that receive
more than 2 parameters, the left operand is applied as the **last** argument.

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

- 1) Parameterization of built-in functions that differ, such as `array_map` and `array_filter`.

### Solutions

- 1) Use a wildcard `$` for parameter allocation:

```php
range(1, 10) |> array_filter($, "even")
             |> array_map("sqrt", $)
             |> array_walk($, "println");
```

### Equivalence

This feature can actually be emulated by using objects and self-return, with fluent-interfaces:

#### Implementation
```php
class Seq
{
  private $value;
  
  function __construct(array $xs)
  {
    $this->value = $xs;
  }
  
  function map(callable $fn)
  {
    $this->value = array_map($fn, $this->value);
    return $this;
  }
  
  function filter(callable $fn)
  {
    $this->value = array_filter($this->value, $fn);
    return $this;
  }
  
  function each(callable $fn)
  {
    array_walk($this->value, $fn);
    return $this;
  }
}
```

#### Usage
```php
(new Seq(range(1, 10))) -> filter("even")
                        -> map("sqrt")
                        -> each("println");
```
