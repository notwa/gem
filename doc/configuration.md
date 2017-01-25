# Configuration

A configuration file is nothing more than a valid Lua file that returns a
`table` with the format specified below. It should be noted that this file
must reside in the project's root directory, and relative paths are specified
with this fact in mind.

The only valid types within the configuration file are `boolean`, `string`,
and `table`.  When listing files, please keep in mind that their order is
respected. All paths are specified using a `string`, and expect the use of
forwardslashes (i.e. `/`) when indicating directory separators. And be sure to
include the file extension when specifying files.

## Debugging

The `debug` keyword can be used within JASS scripts to prefix certain
statements. This is standard JASS notation. Warcraft 3 simply ignores these
lines (although, it is not known if performance is affected). It can safely be
used in conjunction with the following JASS statements:

- `set`
- `if ...`
- `call`
- `loop ...`

For example:

```
function Debug_Example takes nothing returns nothing
    local index = 0

    // This line is ignored.
    debug set index = 1

    // This entire if-statement is ignored.
    debug if index == 1 then
    elseif index == 0 then
    else
        return
    endif

    // This call is ignored.
    debug call BJDisplayMsg ("Hello world!")

    // This infinite loop is ignored.
    debug loop
    endloop

    call BJDisplayMsg (I2S (index)) // 0
endfunction
```

The 'build' command extends upon this behavior without altering the standard
JASS language syntax, making the `debug` keyword more useful. When the
`options.debug` flag is set to `false`, debugging is essentially disabled.
Functionally, this does not really change anything, given that `debug` is
ignored by default. However, in an attempt to avoid potential side effects
(such as code bloat after optimization), statements prefixed with `debug` are
removed. With debugging disabled through the `options.debug` flag, the above
becomes:

```
function Debug_Example takes nothing returns nothing
    local index = 0

    // This line is ignored.

    // This entire if-statement is ignored.

    // This call is ignored.

    // This infinite loop is ignored.

    call BJDisplayMsg (I2S (index)) // 0
endfunction
```

Now, if debugging is enabled by setting `options.debug` flag to `true`, then
all lines modified by the `debug` keyword are kept. The keyword itself is the
only thing removed, and the rest of the code is left intact:

```
function Debug_Example takes nothing returns nothing
    local index = 0

    // This line is executed.
    set index = 1

    // This if-statement is not ignored.
    if index == 1 then
    elseif index == 0 then
    else
        return
    endif

    // This function is called.
    call BJDisplayMsg ("Hello world!") // Hello world!

    // This infinite loop is run, for better or worse.
    loop
    endloop

    // Never reached.
    call BJDisplayMsg (I2S (index))
endfunction
```

As such, care should be taken when using the `debug` keyword. However, it does
enable useful debugging functionality.

There is a known issue with the included version of pjass. It does not
complain about `debug` being used in conjunction with `exitwhen` or `return`.
However, a map with such code will bug and not allow the player to enter the
game lobby. Please avoid such uses of `debug`.

## Globals

All globals meeting specific criteria will be available within Lua scripts
used during the 'constants' and 'objects' commands. They will be exposed
through a global `table` named 'globals'. This includes those present within
both the 'common.j' and 'blizzard.j', as well as any within user provided
scripts.

The criteria for globals to considered are as follows:

1. The global must be declared as constant.
2. The global must be one of the following types:
    - Boolean
    - String
    - Real
    - Integer
3. The global must be assigned a constant value.

Examples of valid globals:

```
constant boolean BOOLEAN_A = true
constant boolean BOOLEAN_B = false

constant string STRING_A = ""
constant string STRING_B = "Hello, world!"

constant real REAL_A = 3.1415926
constant real REAL_B = .50
constant real REAL_C = 0.

constant integer INTEGER_A = 1234
constant integer INTEGER_B = 07
constant integer INTEGER_C = 0xAFF
constant integer INTEGER_D = $ABC1
constant integer INTEGER_E = 'A'
constant integer INTEGER_F = 'A000'
```

Examples of invalid globals:

```
boolean BOOLEAN_C = true
constant boolean BOOLEAN_D = BOOLEAN_C
constant boolean BOOLEAN_E = true and false

constant string STRING_C = "Multiple" + " " + "Parts"
constant string STRING_D = STRING_B + " And you too!"

constant real_C = REAL_A * 5.00
constant real_D = globals.bj_PI

integer INTEGER_G = 1
constant integer_H = 5 + 5 + INTEGER_A
```

In order to faciliate easier understanding of the contents of said globals,
both the JASS `type` and `value` are exposed within the `table` that
represents the global. Do realize that all values wil be of the Lua type
`string`, and it is left up to the user to decide if they wish to transform
these values further. For example:

```
globals.FALSE.type                --> 'boolean'
globals.FALSE.value               --> 'false'

globals.JASS_MAX_ARRAY_SIZE.type  --> 'integer.decimal'
globals.JASS_MAX_ARRAY_SIZE.value --> '8192'

globals.bj_PI.type                --> 'real'
globals.bj_PI.value               --> '3.14159'

globals.INTEGER_F.type            --> 'integer.code'
globals.INTEGER_F.value           --> 'A000'
```

As for the JASS types exposed to Lua, the follow names are used:

- `'boolean'`
- `'string'`
- `'real'`
- `'integer.literal'`
- `'integer.code'`
- `'integer.hexadecimal'`
- `'integer.octal'`
- `'integer.decimal'`

## Settings

All settings are required to exist, and behavior is not tested for situations
where they are absent. If it is desired to remove or clear a setting, use an
empty string (`''`) and/or an empty table (`{}`). Flags (i.e. `boolean`
values) must be set to either `true` or `false`.

### General Settings

#### `name` _(`string`)_

The name of the project, and the name used when referencing and working with
numerous files throughout the project.

#### `flags` _(`table`)_
#### `flags.debug` _(`boolean`)_

A flag to indicate whether to allow supported statements prefixed with the
`debug` keyword to be built into the map.

#### `input` _(`table`)_
#### `input.map` _(`string`)_

The path to the file that will be used as the basis for the working map.

#### `output` _(`table`)_
#### `output.directory` _(`string`)_

The path to the directory where the project work files will be placed.

#### `patch` _(`table`)_
#### `patch.directory` _(`string`)_

The path to the directory that contains the files 'common.j' and 'blizzard.j'.

#### `scripts` _(`table`)_
#### `scripts.directory` _(`string`)_

The path to the directory that contains the project's JASS scripts.

#### `scripts.files` _(`table`)_

The file list that represents the JASS portion of the project.

#### `imports` _(`table`)_
#### `imports.directory` _(`string`)_

The path to the directory that contains all files to be imported into the
working map. Note that subdirectory structure is preserved during the import
process.

#### `objects` _(`table`)_
#### `objects.directory` _(`string`)_

The path to the directory containing files used during object loading.

#### `objects.files` _(`table`)_

The file list containing all Lua files to use during object loading.

#### `constants` _(`table`)_
#### `constants.gameplay` _(`table`)_
#### `constants.gameplay.directory` _(`string`)_

The path to the directory containing files used when loading gameplay related
constants.

#### `constants.gameplay.files` _(`table`)_

The file list containing all Lua files used when loading gameplay related
constants.

#### `constants.interface` _(`table`)_
#### `constants.interface.directory` _(`string`)_

The path to the directory containing files used when loading interface related
constants.

#### `constants.interface.files` _(`table`)_

The file list containing all Lua files used when loading interface related
constants.

### Command Settings

#### `prefix` _(`string`)_

A prefix to put before all commands. On Windows, this should probably be set
to an empty string (`''`). On other systems, the use of Wine will probably be
necessary, and the prefix should be adjusted accordingly.

#### `pjass` _(`table`)_
#### `pjass.options` _(`table`)_

A list of options to provide to pjass.

#### `optimizer` _(`table`)_
#### `optimizer.tweaks` _(`string`)_

The path to the optimizer tweaks file.