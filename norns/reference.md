---
layout: default
parent: scripting
grand_parent: norns
has_children: false
title: reference
nav_order: 1
has_toc: false
---

# norns script reference

Scripts are located in `~/dust/code/`, and are what make norns do things. A script consists of at least a Lua file but can additionally also contain supporting Lua libraries, SuperCollider engines and data.

## directory structure

```
myscript/
  myscript.lua -- main version, shows up as MYSCRIPT
  mod.lua -- alt version, shows up as MYSCRIPT/MOD
  README.md -- main docs/readme
  data/
    myscript-01.pset -- pset, loaded via params:read(1) or via menu
  lib/
    somelib.lua -- arbitrary lib, imported via require 'lib/somelib'
    some-engine.sc -- engine file
    some-engine.lua -- engine lib, require lib/some-engine'
  docs/ -- more documentation, won't be shown in SELECT
    more-docs.md
```

## basic script

```lua
-- scriptname: short script description
-- v1.0.0 @author
-- llllllll.co/t/22222

engine.name = 'PolySub'

function init()
  -- initialization
end

function key(n,z)
  -- key actions: n = number, z = state
end

function enc(n,d)
  -- encoder actions: n = number, d = delta
end

function redraw()
  -- screen redraw
end

function cleanup()
  -- deinitialization
end
```

## libraries

Lua libraries can be used by using `include("path/to/library")`, remember not to include `.lua` in the library name.

### local libraries

`include()` will first look in the directory of the current script. This allows using relative paths to use libraries local to the script. For example, with the following structure:

```
myscript/
  myscript.lua
  lib/
    somelib.lua
```

`myscript.lua` can include `somelib.lua` using:

```lua
include("lib/somelib")
```

### third-party libraries

Third party libraries can be included using their full path starting from the `~/dust/code/` directory. For example, with the following structure in `~/dust/code/`:

```
myscript/
  myscript.lua
  lib/
    somelib.lua
otherscript/
  otherscript.lua
  lib/
    otherlib.lua
```

`myscript.lua` can include `otherlib.lua` using:

```lua
include("otherscript/lib/otherlib")
```

### engine

Specify an engine at the top of your script, see the [engine docs](https://monome.org/norns/classes/engine.html) for more details.

```lua
engine.name = 'PolySub'
```

If you want to use an engine from another project make sure to install that project first.
If the engine comes with an accompanying Lua file make sure to import it:

```lua
engine.name = 'R'

local R = require 'r/lib/r'
```

- `engine.list_commands()` shows the commands.

For example to set the command `cutoff` to 500:

```lua
engine.cutoff(500)
```

To see a list of all locally installed engines:

```lua
tab.print(engine.names)
```

### screen

The screen API handles drawing on the norns screen, see the [screen docs](https://monome.org/norns/classes/screen.html) for more details.

```lua
function redraw()
  screen.clear()
  screen.move(10,10)
  screen.text("hello world")
  screen.update()
end
```

### metro

The metro API allows for high-resolution scheduling, see the [metro docs](https://monome.org/norns/classes/metro.html) for more details.

```lua
re = metro.init()
re.time = 1.0 / 15
re.event = function()
  redraw()
end
```

- `re:start()`, starts metro.
- `re:stop()`, stops metro.

### paramset

The paramset API allows to read and write temporary data and files, see the [paramset docs](https://monome.org/norns/classes/paramset.html) for more details.

A parameter can be installed with the following:

```lua
params:add{type = "number", id = "someparam", name = "Some Param", min = 1, max = 48, default = 4}
```

- `params:set(index, value)`, writes a parameter.
- `params:get(index)`, reads a parameter.

### system globals

Do not overwrite them. Doing so may break things.

System globals:

```lua
_G, _VERSION, assert, bit32, collectgarbage, dofile , error, getmetatable, ipairs, io, load,
loadfile, next, math, os, pairs, pcall, print, rawequal, rawget, rawlen, rawset, require,
select, setmetatable, tonumber, tostring, table, type, utf8, xpcall
```

Norns globals:

```lua
_menu, _norns, _path, _startup, arc, audio, cleanup, clock, controlspec, coroutine, crow,
debug, enc, engine, grid, hid, include, inf, key, metro, midi, mix, norns, osc, package,
params, paramset, paths, poll, redraw, screen, softcut, string, tab, util, wifi
```

## devices

### grid

`grid.connect(n)` to create device, returns object with handler, see the [grid docs](https://monome.org/norns/classes/grid.html) for more details.

```lua
g = grid.connect()
```

- `g:led(x, y, val)`, sets state of single LED on this grid device.
- `g:all(val)`, sets state of all LEDs on this grid device.
- `g:refresh()`, update any dirty quads on this grid device.
- `g.key(x, y, state)`, key event handler function.

### arc

`arc.connect(n)` to create device, returns object with handler, see the [arc docs](https://monome.org/norns/classes/arc.html) for more details.

```lua
a = arc.connect()
```

- `a:led(ring, x, val)`, sets state of single LED on this arc device.
- `a:all(val)`, sets state of all LEDs on this arc device.
- `a:segment(ring, from, to, level)`, creates an anti-aliased point to point arc - segment/range on a specific LED ring.
- `a:refresh()`, updates any dirty quads on this arc device.
- `a.delta(n, delta)`, encoder event handler function.

### midi

`midi.connect(n)` to create device, returns object with handler, see the [midi docs](https://monome.org/norns/classes/midi.html) for more details.

```lua
m = midi.connect()
```

- `m:note_on(value,velocity,channel)`, sends `note_on` message.
- `m:note_off(value,velocity,channel)`, sends `note_off` message.
- `m.event`, midi event handler function.

### hid

`hid.connect(n)` to create device, returns object with handler, see the [hid docs](https://monome.org/norns/classes/hid.html) for more details.

```lua
h = hid.connect()
```

### osc

- `osc.send(to, path, args)`, sends osc event.
- `osc.event(path, args, from)` handler function called when an osc event is received.

## crone

![](../image/crone-process-routing.png)

[pdf version](../crone-process-routing.pdf)


# FAQ

## what is the difference between `require` and `include`?

- `require()` is a built in lua function, `include()` is a norns function

- `require()` looks in more places - see `norns/lua/core/config.lua`

- most importantly, `require()` caches its results, and will not re-run its arguments (unless you use a hack[^1]) the state of a `require`'d module is persisted, and this is significant. consider the following set of files.

`baz.lua`, a module:
```lua
local Baz = {}
Baz.value = 100
return Baz
```

'foo1' and 'foo2' scripts will use the `baz` module twice via `require`, incrementing the module state by 1 and 2 respectively.

`foo1.lua`:
```lua
local baz = require('baz')
print('foo1:baz initial value = ' .. baz.value)
baz.value = baz.value + 1
print('foo1:baz new value = ' .. baz.value)
```

`foo2.lua`:
```lua
local baz = require('baz')
print('foo1:baz initial value = ' .. baz.value)
baz.value = baz.value + 2
print('foo1:baz new value = ' .. baz.value)
```

'bar1' and 'bar2' do the same, but with `dofile`.

`bar1.lua`:
```lua
local baz = dofile('baz.lua')
print('bar1:baz initial value = ' .. baz.value)
baz.value = baz.value + 1
print('bar1:baz new value = ' .. baz.value)
```

`bar2.lua`:
```lua
local baz = dofile('baz.lua')
print('bar2:baz initial value = ' .. baz.value)
baz.value = baz.value + 2


# further

- [Norns Lua API](https://monome.org/docs/norns/api)
- [https://github.com/monome/norns](https://github.com/monome/norns)
