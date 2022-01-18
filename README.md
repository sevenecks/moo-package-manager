# MOO Package Manager (MPM)

## About
The MOO Package Manager is a configurable utility for packaging code up on one MOO and making it available for installation on another MOO. At the core it is provided with an object which acts as the `origin object` which is the starting point and primary piece of your package. It then populates and serializes a dependency graph, which is turned into a serialized version (using maps) of your package. The serailized map is then encoded and can then be copied to another MOO, or made available online.

## Warnings

* The MOO Package Manager is in `open beta` and should be considered only mildly stable. You use this code at your own risk.
* You should always test install new packages on a dev server to make sure nothing breaks.
* After loading a package but prior to installing, you should `@view-package` and review the verb code to ensure you are OK with what you are installing.
* By default the MOO Package Manager comes with one registered package repository (this repos [/packages](/packages) directory. You can add others at your own discresion, and at your own risk.

## Requirements
* [ToastStunt 2.7+](https://github.com/lisdude/toaststunt) (it may work on older versions, but we suggest 2.7!)
* Ability to make outgoing network connections (the package manager uses the `curl` builtin from ToastStunt for these connections)
* A wizard bit
* LamdaCore or ToastCore derived MOO (or you'll need to do some hacking!)
* $string_utils
* $object_utils
* $command_utils
* $diff_utils (this is available through the package manager, and the package manager can install it without it existing)

## Installation
//TODO: write installation instructions
* Install the Diff Utils package

### What's in a package?
A package is a complete collection of everything needed to recreate an object and its dependencies on another MOO.

#### Origin Object
The `origin object` is the most important piece of your package. It serves as the starting point in building the dependency graph. By default all the verbs/props of the `origin object` are serialied. However, this behavior can be over-ridden.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Verbs | The verbs defined on the object | All verbs serialized by default, but you can override and provide a list of verbs |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |

#### Ancestor Objects
`Ancestors` are the parent(s) of the `origin object`. We serialize each of the ancestors but unlike the `origin object` we only serialize what we need from the `ancestor`.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |
| `pass()`ed verbs | The verbs on the `origin object` that `pass()` to an ancestor | Always Serialized |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |

#### Other Objects
A package may rely on more than it's `ancestors`. The dependency graph that is built scans for cored references to other objects that are defined in the code. For example if your `origin object` defines a verb that makes a call to `$string_utils:name_and_number()` then we will include `$string_utils`. This inclusion essentially creates a sub package, treating `$string_utils` as the `origin object` but serializing only the verbs on `$string_utils` that are called elsewhere in the package. If a verb in the `$string_utils` sub package calls `$command_utils:suspend_if_needed` then another sub package is created within the `$string_utils` sub package, treating `$command_utils` as the origin object, and so on and so forth, until all of the dependencies have been serialized throughout the graph.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |
| referenced  verbs | The verbs on this object that are referenced elsewhere in the package code | Always Serialized |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |

#### Not Included

Code referenced on objects such as player/dobj/iobj is deliberately not included in the package as this would create a gigantic dependency graph and essentially requrie updating a huge number of verbs just because a verb calls `player:tell()` once.

> Note: It should technically be possible to make a package out of an object like Generic Player (#6) but experiences may vary. That has not been tested at this point. Feel free to ping me if you do it and let me know how it goes!

## Making a Package

To make a package, you use the `@make-package` command and provide arguments. For example:

```
@make-package                          -> show the argument options
@make-package #20                      -> start creating a package out of #20
```

> Warning: The more complicated your code is, and the more reliant it is on other utilities/objects the larger your package is going to be.

When your package is finished, the package map and serialized package maps will be stored in the `$mpm.last_created_package_map` and `$mpm.last_created_package_hash` properties respectively.

> Note: See the section on Making Packages Available Online for more information on how to make your package available online.

### Package Creation Caveats

The package manager works best with mostly self contained code that doesn't rely on verbs across your entire MOO. In general, MOO code is pretty coupled, and so you may end up with a large dependency graph. If you are making a package, please consider how this might affect the usability of your package and consider decoupling your code, or creating smaller packages.

> Note: Since the package manager is still beta, lots of experimentation, testing, and updates to the package manager will likely be needed to make it as robust as possible and capable of handling dependencies more smartly. Suggestions are welcome!

It is currently not possible to serialize a package which contains binary data such as `~1E`. This is due to how we are currently encoding the packages. If you have suggestions for how we can do encode better, please reach out.

It is currently not possible to serialize packages that reference object numbers in their code. Use cored references instead.

It is currently not possible to serialize packages that make dynamic verb or property references such as:

```
verbname = "nn";
propname = "apropnamehere";
$string_utils:(verbname)(player);
$widget_utils.(propname);
$string_utils:("left")();
$widget_utils.("someprop");
```

This is due to the fact that we can't smartly serialize these references without actually parsing all the verb code to figure out what it is trying to call.

> Call to Action: Suggestions for how to deal with this are welcome. I'm considering just prompting the user to provide the verbs/props manually instead of failing the package creation entirely like we are doing now.

### Package Creation Options

`@make-package` accepts a number of arguments outlined below:

| Argument | Description | Required |
| ------------- | ------------- | ------------- | 
| object number | the object number of the `origin object` | yes |
| --select-verbs | enable dynamic selection of verbs on `origin object` to include via interactive prompt | no |
| --fully-serialize-ancestry | enable full serialization of ancestors verbs (excluding objects whose parent is $nothing) | no |
| --serialize-#1 | enable full serialization of ancestors whose parent is $nothing | no |
| --verb-list=verbname1,verbname2,... | only serialize `origin object` verbs in this comma seperated list (no space after the `,`!) | no |
| --ignore-prop-list=propname1,propname2,#20.propname,... | if no obj# is provided, ignores any prop on any object with specified propname, if obj# is provided, ignores that prop on only that obj# | no |
| --dry-run | generates the package but does not save it, instead offers to display the generated package map | no |

### Package Meta Data

There are a number of meta data fields associated with your package. These fields are serialized with your package, and also make up the package header information that can be made available in a package_list inside a package repository. The meta data is mainly collected by prompting the user during the package creation process, but some is autogenrated as well. The meta data saved for a package is outlined below:

| Name| Description | Required |
| ------------- | ------------- | ------------- | 
| Package Name | Name of your package, this should NOT include a version number | yes |
| Package Version | A float. Like 1.0 or 1.1, the version of your package | yes |
| Package Hash | An md5 hash of your package map, auto generated when you create a package | yes |
| Package Description | The description of your package. This is shown when browsing a package repository and when viewing/installing a package | yes |
| Package Created At | An autogenerated timestamp of when the package was created | yes |
| Package Created By | The name and number of the programmer who created the package | yes |
| Package URL | The url where the package can be found online. | no |
| Package Changelog | Holds the most recent updates to the package, for when you are updating an existing package | no |
| Package Post Install Note | A note shown to the user when the package is finished installing. Good for any additional setup options, or pointing to help files | no |

## Loading a Package

The `@load-package` verb is used to browse packages/package info in  registered package repositories and download a package into the MOO Package Manager, thus preparing it for installation. It also provides the option of specifying a package's URL directly, allowing you to load a package from anywhere.

> Note: Only one package can be loaded at a time. Loading a new package will overwrite the previously loaded package.

## Viewing a Package

After you have loaded a package, you can view it with `@view-package`. This will display a pretty printed dump of the package, which you can use to review. You can also review the `$mpm.laaded_package` property directly, to see the entire package map.

## Installing a Package

> Warning: Seriously, you should test install packages on a dev server and not on production. Especially if you are installing a package that updates commonly used verbs on commonly used objects such as $object_utils, $string_utils, etc.

> Warning: If you're going to YOLO it and install an untested package on production, please dump your database `@dump-database` first and make a copy of it on your server (in case you checkpoint after installing a package that breaks your MOO).

After loading and viewing a package you can `@install-package` to kick off the installation process. Installation is an interactive process that will require you to make decisions about how your new package is installed. There a number of decision points during the installation process which are outlined below.

When you first `@install-package` he package manager checks to see if the package hash matches the hashed version of the package map. If this fails, the package installation is aborted.

> Note: If you are seeing a hash mismatch, then there is probably a bug in how we are encoding the package, and you should open an issue or reach out on Discord (see Contributing section).

It will then check to see if this specific version of this package has already been installed (by checking `$mpm.installed_packages` for a matching package hash. If the package has already been installed, you will see a message alerting you of this, but you may continue your installation.

The package meta data (such as name, description, version, created at, etc.) will then be displayed and you will be prompted to confirm you want to install the package.

The package manager will then attempt to find a version of the object already existing on your server. It will check the obj# that was included in the hash (since many MOOs have the same obj# for utility packages like #20 (String Utils)). We then check if the name matches that of the package. If we are unable to find a matching object, we will check if the package has a cored reference, and if found, check their name against the package. You will then be prompted to decide if you want to update the existing object or create a new object.

Creating a new object is a good way to test a package that would otherwise make in-place updates to an existing object on your MOO. However, there are a few caveats. Even if you choose to create a new object, the dependencies of that object (IE: $string_utils, $object_utils, etc) will still be in place updates, as you are not given the option to create new versions of dependencies that exist on your MOO, as the packages code will be making cored references to those objects. However, you can always decline to make the in-place updates to the verbs.

> Note: If you are creating a test object, and are prompted to update cored references, you should decline to update those references. You can always reinstall the package on top of your existing test object, and update the cored references on the second go around.

If the package manager cannot find a matching object, it will search for the closest ancestor of the `origin object` that it can find on your MOO, and offer to recreate the object from that ancestor. This will create not only a new `origin object` but potentially new `ancestors`. For example if the package has an `origin object` with ancestors like this:

```
#1
  #78
     #200
        #500 (origin object)
```

And your MOO only has: 

```
#1
  #78
```

Then a new version of #200 and #500 will be created (with different object numbers most likely). And then package creation will continue.

At this point, the verbs, properties, and cored references of the objects in the package will begin to be created. Verbs and properties will be created if they do not exist.

If a verb exists and is different, you will be shown a diff and given the option to update or decline the update.

If a property exists and is different, you will be shown both property values and given the option to update or decline the update.

If a cored reference doesn't exist, it will be created.

If a cored reference exists and doesn't match, you will be propmted and given the option to accept or decline the update.

If at any point, an object that is needed doesn't exist (if for example the package needed $widget_utils, which your MOO didn't have), the package manager will find the closest ancestor that your MOO does have, and offer to create the parent from that, in the same way it did with the `origin object`.

> Note: Any property/verb additions or updates are logged in `$mpm.log` along with the old/new verb/prop values and any other data associated with the change. While the package manager doesn't currently offer a rollback option, if the worst happens, you'll have an audit log of the changes made and can attempt to manually fix/revert.

> Warning: the `$mpm.log` can get really long. You may want to clear it from time to time.

## Making Packages Available Online

## Caveats
* By default MPM will not serialize verbs/props on ancestors who have a parent of $nothing (IE: #1), as this can make a package huge. This can be over-ridden..

## Unsupported
* Does not support dynamic verb or prop references (IE: $string_utils:("verbname") or $string_utils.(propvar))
* Does not support serializing verbs/props that have binary data in them (IE: ~OE)

## Contributing

### Opening Issues
If you have an issue with using the package manager, feel free to open an issue.

### Contributing a Package
TBD. For now reach out to Slither on the [ToastStunt Discord](https://discord.gg/XyXP43e).

### Contributing to the Package Manager Source
- If you would like to contribute the best way to do so is by joining the [ToastStunt Discord](https://discord.gg/XyXP43e).
- Pull requests are also welcome
