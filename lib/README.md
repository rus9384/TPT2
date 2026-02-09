A collection of useful macros and lua functions for [d0s.boots' editor](https://d0sboots.github.io/perfect-tower/).

## TPT2.lib.general

Contains general purpose macros and functions.

### `#multiline(expr)`

Use this macro inside new format (`={}`) macros to parse multiple lines as a single AI line.

### `#pos.stretchable(x, y, left, right, bottom, top)`

Returns `vec(x, y)` inside a stretchable rectangle where `x` is the distance from the left side of the rectangle in \[0-1\] range and `y` is the distance from the bottom of the rectangle in \[0-1\] range. The other parameters specify non-stretchable bounds of the rectangle.

### `#click.stretchable(x, y, left, right, bottom, top)`

Clicks on a position inside a stretchable rectangle as defined in `#pos.stretchable`.

### `string.nested(pattern, args)`

Iteratively substitutes a substring `%1` with `res` and a substring `%2` with the next argument in the `pattern`. Reads arguments from the end to the beginning.

For example, `string.nested("%1: %2", "a, b, c")` returns `"c: b: a"`.

## TPT2.lib.math

Contains mathematical macros.

For descriptions of macro functionality see comments.

## TPT2.lib.tower

Contains macros that are useful for Tower Testing scripts.

For descriptions of macro functionality see comments.
