---
layout: default
title: target.json
section: reference/target
---

# target.json Reference
The `target.json` file is used to describe yotta targets. It defines where to
find the CMake toolchain used to control compilation, and the data that can be
used by modules to include different dependencies or compile different code
when building for different targets.

### Example File
```json
{
  "name": "frdm-k64f-gcc",
  "version": "0.0.7",
  "inherits": {
    "mbed-gcc": "~0.1.2"
  },
  "config":{
    "mbed":{
      "ksdk-mcu":true,
      "cortexm":4
    },
    "devices"{
      "foobar":{
        "baz": 123,
        "frequency": 11
      }
    }
  },
  "toolchain": "CMake/toolchain.cmake",
  "scripts": {
    "debug": ["valinor", "--target", "K64F", "$program" ],
    "test": [ "mbed_test_wrapper", "--target", "K64F", "$program" ]
  },
  "similarTo": [
    "frdm-k64f",
    "k64f",
    "ksdk-mcu",
    "mk64fn1m0vmd12",
    "mk64fn1m0",
    "mk64fn",
    "freescale",
    "cortex-m4"
  ]
}
```

## Properties

### `name` *required*
**type: String**

The unique name of the target. Globally unique. As the target also defines
which compiler is used to build, target names should normally explicitly state
which compiler they use (e.g. `some-target-gcc`, `some-target-armcc`).

Names can use only lowercase letters, numbers, and hyphens, but must start with
a letter. (This reduces problems with case insensitive filesystems, and
confusingly similar names.)


### <a href="#version" name="version">#</a> `version` *required*
**type: String (conforming to the [semver](http://semver.org) specification)**

`yotta` uses Semantic Versions for targets and modules. See the [module.json
reference](../reference/module.html#version) for details.

### <a href="#inherits" name="inherits">#</a> `inherits`
**type: hash of target-name to version-specification**

A target descriptions may inherit from a more generic target description. This
allows target descriptions for similar hardware devices to share a common base
implementation, overriding only the parts that need to be different.

Only a single base target may be specified for any target.

Example:

```json
  "inherits": {
    "mbed-gcc": "~0.1.2"
  }
```

### <a href="#similarTo" name="similarTo">#</a> `similarTo`
**type: Array of String**

An array of target identifiers that this target is similar to. This list is
used when resolving [target-specific dependencies](../reference/module.html#targetDependencies).

When calculating the dependencies to install, `yotta` uses the first section
from `targetDependencies` that matches one of the identifiers in this list.

The identifiers are arbitrary strings, and do not need to be the names of other
targets.

### <a href="#licenses" name="licenses">#</a> `licenses` *required*
**type: Array of objects: `{"url":"<URL to full license>", "type":"<SPDX license identifier>" }`**

The licenses property in module.json should include all of the licenses that
affect code in your module. For example:

```json
  "licenses": [
    {
      "url": "https://spdx.org/licenses/Apache-2.0",
      "type": "Apache-2.0"
    }
  ]
```

If you're starting a completely new module, and have freedom to choose the
license yourself, `yotta`'s preferred license is
[Apache-2.0](http://spdx.org/licenses/Apache-2.0), a permissive OSI-approved
open source license which provides clarity over the scope of patent grants.
`yotta` itself is also licensed under Apache-2.0.

### <a href="#description" name="description">#</a> `description`
**type: String**

Brief description of what this target is for. This helps other people to find
your target.
Include a `readme.md` file with a longer description, and preferably a photo of
the target platform.

### <a href="#keywords" name="keywords">#</a> `keywords`
**type: Array of String**

Keywords describe what this target is for, and help other people to find it.
For example, a target to build for a specific mbed board should be tagged with
`mbed-target:{mbedtargetname}` (where `{mbedtargetname}` should be replaced
with the mbed target name of the development board.

### <a href="#toolchain" name="toolchain">#</a> `toolchain`
**type: String (path relative to target root directory)**

Path to the target's CMake toolchain file. If this target [inherits](#inherits)
from another target that provides a functioning toolchain this property is
optional.

### <a href="#scripts" name="scripts">#</a> `scripts`
**type: hash of script-name to command**

Each command is an array of the separate command arguments.

The supported scripts are:

 * **debug**: this is the command that's run by `yotta debug`, it should
   probably open a debugger. `$program` will be expanded to the full path of
   the binary to be debugged.
 * **test**: this command is used by `yotta test` to run each test. `$program`
   will be expanded to the full path of the binary to be debugged. For
   cross-compiling targets, this command should load the binary on to the
   target device, and then print the program's output on standard out.

For example, the scripts for a native compilation target might look like:

```json
   "scripts": {
      "debug": ["lldb", "$program"],
      "test": ["$program"]
   }
```

### <a href="#config" name="config">#</a> `config`
**type: hash of config data**

Optional [config data](/reference/config.html) to define or override. For
details on the yotta config system, see the [config
reference](/reference/config.html).


### `debugServer` **deprecated: use scripts.debug instead**
**type: Array of String (command parts)**

Optional command to run a debug server.

### `debug` **deprecated: use scripts.debug instead**
**type: Array of String (command parts)**

Optional command to run when the `yotta debug` is invoked.

Use `$program` for the name of the program that's being debugged, for example:

```json
  "debug": [
    "gdb", "$program", "--eval", "target remote localhost:2331"
  ]
```

