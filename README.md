# MOO Package Manager (MPM)

## About
The MOO Package Manager is a configurable utility for packaging code up on one MOO and making it available for installation on another MOO. At the core it is provided with an object which acts as the `origin object` which is the starting point and primary piece of your package. It then populates and serializes a dependency graph, which is turned into a serialized version (using maps) of your package. The serailized map can then be copied to another MOO, or made available via GitHub (or
anywhere) for import.

// TODO: this is not possible currently
The package manager will then support you importing a package via an @import-package command, or by browsing registered repositories for packages to utilize.

### What's in the package?

#### Origin Object
* The .name, .aliases, and cored references
* All verb name & code directly defined on the `origin object` along with the `verb supplementary data` about the verb (IE: this none this), and the `verb_info` including if the verb was owned by a `wizard`
* All properties defined on the `origin object` or its parent (needed since an ancestor can define a prop that is overwritten on the child) as well as the `property supplementary data` such as permissions and the actual value of the property.

#### Ancestor Objects
* For each ancestor of the `origin object` we store: .name, .aliases, cored references, all properties
* All the verbs and `supplementary verb data`  on the ancestor(s) that are referenced in a pass() by the `origin object`. This is recursive through the ancestor tree, so if an ancestor also calls pass() it will continue to serialize the verb on ancestors
* All the verbs and `supplementary verb data` on the ancestor(s) that are called directly from any code in the package
* All the verbs and `supplementary verb data`  on the ancestor(s) that are called on the `origin object` directly but not defined on the `origin object` (as they fall through to the ancestor automatically)
* All the properties and `supplementary property data` on the parent

#### Other Objects
* The .name, .aliases, and cored references of any object referenced by any code in the package
* Verbs and `supplementary verb data` on any object referenced by any code in the package
* Any property and `supplementary property data` referenced in any code in the package

## Features

## Installation

## Usage

## Options

## Caveats
* By default MPM will not serialize verbs/props on #1, as this can make a package huge. This option can be changed.

## Unsupported
* Does not support #0.propname that are not objects (IE: #0.debugging_enabled)
* Does not support dynamic verb or prop references (IE: $string_utils:("verbname") or $string_utils.(propvar))

## Contributing
