A collection of useful macros and lua functions for [d0s.boots' editor](https://d0sboots.github.io/perfect-tower/).

## Import code

```
{"workspaces":{"TPT2.lib":[["TPT2.lib.general","; Used to parse multiline expressions as a single AI instruction\n#multiline(expr) {lua(return ([[{expr}]]):gsub(\"\\n\", \"\"))}\n\n; A generalization of pos.relative, allows setting position in a rectangular stretchable area\n; Is equal to pos.relative(left - x * (left + right - 1.), bottom - x * (bottom + top - 1.), x, y)\n; ┌──────┬────────────┬───────┐\n; │      │ ↕ top      │       │\n; │      ├────────────┤       │\n; │ left │←───→╳      │ right │\n; │←────→│  x  ↕ y    │←─────→│\n; │      ├────────────┤       │\n; │      │ ↕ bottom   │       │\n; └──────┴────────────┴───────┘\n#pos.stretchable(x, y, left, right, bottom, top)={{lua(\n  return stretchable(\"{x}\", \"{y}\", \"{left}\", \"{right}\", \"{bottom}\", \"{top}\")\n)}}\n\n#click.stretchable(x, y, left, right, bottom, top)={\n  click({pos.stretchable({x}, {y}, {left}, {right}, {bottom}, {top})})\n}\n\n{lua(\n\n  function stretchable(x, y, left, right, bottom, top)\n\n    local function op(operand1, operation, operand2)\n\n      operand1 = tonumber(operand1) or operand1\n      operand2 = tonumber(operand2) or operand2\n\n      local expr = string.format(\"(%s) %s (%s)\", operand1, operation, operand2)\n      local status, res = pcall(load(\"return \" .. expr))\n      if status then\n        return res\n      end\n\n      if operation == \"+\" then\n        return operand1 == 0.0 and operand2 or operand2 == 0.0 and operand1 or expr\n      elseif operation == \"*\" then\n        return (operand1 == 0.0 or operand2 == 0.0) and 0.0\n          or operand1 == 1.0 and operand2 or operand2 == 1.0 and operand1 or expr\n      end\n      return expr\n\n    end\n\n    local vec = {}\n  \n    vec.x = op(left, \"-\", op(x, \"*\", op(left, \"+\", right)))\n    vec.x = op(\"min(width.d(), (16./9.) * height.d()) * ui.size()\", \"*\", vec.x)\n    vec.x = op(vec.x, \"+\", op(\"width.d()\", \"*\", x))\n  \n    vec.y = op(bottom, \"-\", op(y, \"*\", op(bottom, \"+\", top)))\n    vec.y = op(\"min(height.d(), (9./16.) * width.d()) * ui.size()\", \"*\", vec.y)\n    vec.y = op(vec.y, \"+\", op(\"height.d()\", \"*\", y))\n  \n    return string.format(\"vec(%s, %s)\", vec.x, vec.y)\n\n  end\n\n  -- Iteratively substitutes %1 with res and %2 with the next argument in the pattern\n  -- Reads arguments from the end to the beginning\n  function string.nested(pattern, args)\n    local argt = string.args(args)\n    local res = argt[#argt]\n    for i = #argt - 1, 1, -1 do\n      res = string.substitute(pattern, res, argt[i])\n    end\n    return res\n  end\n\n  function string.substitute(pattern, ...)\n\n    if select(\"#\", ...) > 9 then\n      error(\"The function supports only up 9 replacement parameters.\")\n    end\n\n    local res, count = pattern\n    for i = 1, select(\"#\", ...) do\n      local search1 = string.format(\"^((%%%%*)%%2)(%%%%%u)(.*)\", i)\n      local search2 = string.format(\"(.-[^%%%%](%%%%*)%%2)(%%%%%u)(.*)\", i)\n      local repl = string.format(\"%%1%s%%4\", select(i, ...))\n      res, count = res:gsub(search1, repl, 1):gsub(search2, repl, 1)\n      while count > 0 do\n        res, count = res:gsub(search2, repl, 1)\n      end\n    end\n\n    return res\n\n  end\n\n  function string.args(args)\n\n    local res = {}\n    local first, last = 1, args:find([[([,'\"])]], 1)\n\n    while last do\n      local symbol = args:sub(last, last)\n      if symbol ~= \",\" then\n        last = args:find(symbol, last + 1)\n      else\n        table.insert(res, args:sub(first, last - 1):match(\"^%s*(.-)%s*$\"))\n        first = last + 1\n      end\n      last = args:find([[([,'\"])]], last + 1)\n    end\n    table.insert(res, args:sub(first):match(\"^%s*(.-)%s*$\"))\n\n    return res\n\n  end\n\n)}\n"],["TPT2.lib.math",":import TPT2.lib.general\n\n; Returns the sum/product of numeric values (respectively)\n#sum($...) {lua(return string.nested(\"%2 + (%1)\", [[{...}]]))}\n#prod($...) {lua(return string.nested(\"%2 * (%1)\", [[{...}]]))}\n\n; Returns the absolute value of an integer/double number (respectively)\n#i.abs(number) if({number} < 0 , 0  - {number}, {number})\n#d.abs(number) if({number} < 0., 0. - {number}, {number})\n\n; Returns a minimum/maximum (respectively) of multiple values\n#min($...) {lua(return string.nested(\"min(%2, %1)\", [[{...}]]))}\n#max($...) {lua(return string.nested(\"max(%2, %1)\", [[{...}]]))}\n"],["TPT2.lib.tower",":import TPT2.lib.general\n\n; The string containing names of all \"permanent\" active modules\n#spells.permanent \"crate.rex,sphere.energy,google.influence,spell.immolation,knight.power,boost.shoreline,guns.space\"\n\n; Returns the total cost of listed era disables\n#disable.cost.total($...) {lua(return string.nested(\"disable.cost(%2)+(%1)\", [[{...},]]):gsub(\"%+%(%)\", \"\"))}\n\n; Returns true if any of listed era disables is free\n#disable.exists.free($...) {lua(return string.nested(\"disable.cost(%2)*(%1)\", [[{...},]]):gsub(\"%*%(%)\", \"\"))} == 0.\n\n; Disables next element in the elements list\n#disable.era.next($...) disable.era({lua(return string.nested(\"if(disable.cost(%2) >= 0., %2, %1)\", [[{...}]]))})\n"]]}}
```

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
