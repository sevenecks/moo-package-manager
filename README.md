# MOO Package Manager (MPM) 1.0

## About
The MOO Package Manager is a configurable utility for packaging code up on one MOO and making it available for installation on another MOO. At the core it is provided with an object which acts as the `origin object` which is the starting point and primary piece of your package. It then populates and serializes a dependency graph, which is turned into a serialized version (using maps) of your package. The serialized map is then encoded and can then be copied to another MOO / made available online and installed via the MOO Package Manager.

## Warnings

* The MOO Package Manager is in `open beta` and should be considered only mildly stable. You use this code at your own risk.
* You should always test install new packages on a dev server to make sure nothing breaks.
* After loading a package but prior to installing, you should `@view-package loaded` and review the verbs it will touch, and possibly review the code being installed by evaling `;parse_json($mpm.loaded_package, "embedded-types")` to ensure you are OK with what you are installing.
* By default the MOO Package Manager comes with one registered package repository which points to this repos [/packages](/packages) directory. You can add others at your own discresion, and at your own risk.

## Requirements
* [ToastStunt 2.7+](https://github.com/lisdude/toaststunt) (it may work on older versions, perhaps even Stunt, with some modifications)
* Ability to make outgoing network connections (the package manager uses the `curl` builtin from ToastStunt for these connections)
* A wizard bit
* LamdaCore or ToastCore derived MOO (or you'll need to do some hacking!)
* $string_utils, $object_utils, $command_utils, @program 
* $diff_utils (this is available through the package manager, and the package manager can install it without it existing)

## Installation
Hopefully, this will be the last time you need to manually install MOO code! At least, that's the goal. In order to install the MOO Package Manager you will need to copy and paste some code. The package manager code itself is quite long, so including it in this README is not effective. The code is stored in files, linked below.

**Installing The Package Manager**

1. `@create #78 named MOO Package Manager`
2. `@corify newObj# as $mpm` 
3. Open the [MOO Package Manager Code](/code/moo_package_manager)
4. Copy and paste the code into your MOO
5. Open the [MOO Package Manager User Code](/code/moo_package_manager_user)
6. Copy and paste the code into your MOO
7. @create $waif named Memory Waif
8. @corify newObj# as $memory_waif
9. @prop $memory_waif.":data" [] rw

> Note: `#78` is typically the Generic Utility object, the same parent that $string_utils has. It may differ on your MOO.

> Note: `@corify` adds a new property to `$sysobj` (`#0` on most systems). If you don't have this verb for some reason you can just `@prop $sysobj.mpm newObj#`

> Note: If you have an issue copying the MOO Package Manager User Code because `me` is not defined, just change all references to `me` to the object# of your player bit (yourself).

> Note: If you have trouble creating a property with [] (map) then create it with any value and then evaluate `;$memory_waif.:data = []`

You will now have an $mpm object which encapsulates the majority of the MOO Package Manager's code. You will also have several helper verbs on your player bit that will allow you to interact with the package manager. For now, we are concerned with two of these: `@load-package` and `@install-package`.

**Installing Your First Package**

The MOO Package Manager is mostly self contained. However, it does rely on a package called `Diff Utilities` which it uses to present code diffs. The Diff Utilities is a repackage / update of the original Stunt utility that came bundled with the `Improvise.db`. We need to install this package in order for the MOO Package Manager to operate properly when it needs to update an existing verb (fourtunately, this won't be an issue, as Diff Utils is a brand new utility that you don't have on your MOO).

To install the Diff Utility follow these steps:

1. `@load-package`
2. Select `View Available Packages`
3. Select `Slither's MOO Packages`
4. Select the lastest version of `Diff Utilities`
5. Review the package data and select `yes` when ready
6. `@view-package loaded` to see what the package will install
7. `@install-package` to kick off the package installation
8. Review the data presented and enter `yes` when ready to install.
9. Follow the prompts to create a new object and install the package.

> Note: For more information on installing a package see the [Installing a Package](#installing-a-package) section.

You should now have a working package manager, and a working Diff Utilities which you can reference with `$diff_utils`.

## What's in a package?
A package is a collection of everything needed to recreate an object and its dependencies on another MOO.

### Origin Object
The `origin object` is the most important piece of your package. It serves as the starting point in building the dependency graph. By default all the verbs/props of the `origin object` are serialied. However, this behavior can be over-ridden.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Verbs | The verbs defined on the object | All verbs serialized by default, but you can override and provide a list of verbs |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |

### Ancestor Objects
`Ancestors` are the parent(s) of the `origin object`. We serialize each of the ancestors but unlike the `origin object` we only serialize what we need from the `ancestor`.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |
| `pass()`ed verbs | The verbs on the `origin object` that `pass()` to an ancestor | Always Serialized |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |

### Other Objects
A package may rely on more than it's `ancestors`. The dependency graph that is built scans for cored references to other objects that are defined in the code. For example if your `origin object` defines a verb that makes a call to `$string_utils:name_and_number()` then we will include `$string_utils`. This inclusion essentially creates a sub package, treating `$string_utils` as the `origin object` but serializing only the verbs on `$string_utils` that are called elsewhere in the package. If a verb in the `$string_utils` sub package calls `$command_utils:suspend_if_needed` then another sub package is created within the `$string_utils` sub package, treating `$command_utils` as the origin object, and so on and so forth, until all of the dependencies have been serialized throughout the graph.

| Serialized | Description | State |
| ------------- | ------------- | ------------- | 
| Cored References | Any props on $sysobj that have a value matching this object | Always Serialized |
| Properties | The properties defiend on the object & their value | All props serialized by default, but you can override and ignore properties |
| Property Supplementary Data | Permissions | Serialized for every included prop |
| referenced  verbs | The verbs on this object that are referenced elsewhere in the package code | Always Serialized |
| Verb Supplementary Data | `verb_args()`, `verb_info()`, verb owned by a `wizard` | Serialized for every included verb |

### Not Included

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

> Multiple inheritance is not supported.

### Package Creation Options

`@make-package` accepts a number of arguments outlined below:

| Argument | Description | Required |
| ------------- | ------------- | ------------- | 
| object number | the object number of the `origin object`, must be first argument provided | yes |
| --select-verbs | enable dynamic selection of verbs on `origin object` to include via interactive prompt | no |
| --fully-serialize-ancestry | enable full serialization of ancestors verbs (excluding objects whose parent is $nothing) | no |
| --serialize-#1 | enable full serialization of ancestors whose parent is $nothing | no |
| --verb-list=verbname1,verbname2,... | only serialize `origin object` verbs in this comma seperated list (no space after the `,`!) | no |
| --ignore-prop-list=propname1,propname2,#20.propname,... | if no obj# is provided, ignores any prop on any object with specified propname, if obj# is provided, ignores that prop on only that obj# | no |
| --reset-prop-value-list=obj.prop,obj.prop,... | if provided, when serializing obj.prop it will be reset to an empty version of whatever value it holds to an empty/false value. | no |
| --only-origin-object | Ignore all package dependencies and only serialize the origin object + ancestry | no |
| --ignore-all-cored-props | Ignore all cored properties, meaning the props & objects they reference will not be serialized in the package | no |
| --dont-serialize-cored-aliases | If any object being serialized has cored aliases, such as $string_utils, they will not be included |
| --ignore-all-non-cored-props | Ignores any properties defined on objects, causing them to not be serialized in the package |
| --only-include-prop-list=obj.prop1,prop2,... | Only serialize properties included in this list | no |
| --post-install-verb=verbname | Specify a verb that will be automatically run after a successful installation. Verb must exist on `origin object` and cannot accept any arguments. | no |
| --target-self | this option overrides the default behavior and forces the package to be installed on the `player` installing it. Must be used in conjunction with --only-origin-object | no |
| --allow-dynamic-verb-calls | this will prevent package creation from aborting when a dynamic verb call is detected. use with care. | no | 
| --allow-dynamic-prop-calls | this will prevent package creation from aborting when a dynamic prop call is detected. use with care. | no |
| --dont-serialize-ancestry | UNTESTED ALPHA FEATURE. This will prevent ancestors from being serialized, essentially setting the parent of all serialized objects to $nothing | no |
| --dry-run | generates the package but does not save it, instead offers to display the generated package map | no |

> Note: Arguments can be provided in any order, except for the object number, which must be first.

An example of using --reset-prop-value-list to `@make-package` a new version of the MOO Package Manager (in this case, the obj# for the package manager is #24836). In order to do this we employ the `--reset-prop-value-list` and `--only-origin-object` arguments to only serialize the MPM itself, as well as reset props that are specific to the instance of the object and not needed in the package itself. It also enables `--dry-run` to avoid saving the package map. It also enabled
`--ignore-prop-list` and passes in some properties to ignore completely on all objects in the package. It then specifies we should allow serializing to continue when we detect dynamic prop calls via `--allow-dynamic-prop-calls`.

```
@make-package $mpm --reset-prop-value-list=#24836.log,#24836.created_packages,#24836.installed_packages,#24836.loaded_package,#24836.last_created_package_map,#24836.last_created_package_encoded --dry-run --only-origin-object --ignore-prop-list=object_size,last_location,realname,weight,movement_queue,debug,type_history,create_data,instance_id,create_date --allow-dynamic-prop-calls
```

The `--reset-prop-value-list` argument can only reset certain properties:

| Type | Reset Value |
| ---- | ----------- |
| INT | 0 |
| STR | "" |
| LIST | {} |
| MAP | [] |
| BOOL | false |
| FLOAT | 0.0 |
| OBJ | #-1 |

All other property values (such as WAIF, ANON, ERR) will throw `E_ARGS` with the value it failed to reset, and package creation will be aborted.

An example of serializing the MPM Helper Verbs that a `wizard` uses to make/load/install packages:

```
@make-package #22664 --verb-list=@make-package,@view-package,@install-package,@load-package --only-origin-object --ignore-all-cored-props --dont-serialize-cored-aliases --dont-serialize-ancestry --allow-dynamic-verb-calls --ignore-all-non-cored-props --target-self --dry-run
```

In the above example we made use of a variety of command-line options:
* `--verb-list` to specify the ONLY verbs on the object we wanted to serialize
* `--only-origin-object` to only serialize the `origin object ` thus MPM does not to dive into the dependency graph
* `--ignore-all-cored-props` so we don't get cored references in our package map, that we wouldn't be using anyway
* `--dont-serialize-cored-aliases` so we don't serialize any props on $sysobj that point to the `origin object`
* `--dont-serialize-ancestry` so we don't package up the parents of the `origin object`
* `--allow-dynamic-verb-calls` so that `@install-package` (which can dynamically call a verb on the installed package when install completes) doesn't raise an error when MPM detects the dynamic verb call
* `--dry-run` so we can test out packaging without saving the package to `.created_packages`.
* `--ignore-all-non-cored-props` so that we don't serialize any of the props on #22664 (good thing too, since `player` bits typically have a lot of props
* `--target-self` to make this package install on the `player` who runs `@install-package`

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
| MPM Version | The version of MPM that was used to create the package | yes |

### Testing a Created Package

It's important that you test your newly created package on a dev server before making it available to the public. It would be preferable to test the package with a stock [ToastCore](https://github.com/lisdude/toastcore) database. This will ensure it works as expected without any potential hidden dependencies that cause it to function properly on your MOO, that were some how not serialized as part of the package.

## Loading a Package

The `@load-package` verb is used to browse packages/package info in  registered package repositories and download a package into the MOO Package Manager, thus preparing it for installation. It also provides the option of specifying a package's URL directly, allowing you to load a package from anywhere.

> Note: Only one package can be loaded at a time. Loading a new package will overwrite the previously loaded package.

## Viewing a Package

`@view-package` will display a pretty printed dump of the package, which you can use to review. You can also review the `$mpm.laaded_package` property directly, to see the entire package map.

After creating a package you can view it with `@view-package created`.

After you have loaded a package, you can view it with `@view-package loaded`.

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

## Making Packages Available
The MOO Package Manager makes it easy to copy packages from one MOO to another, or to make your packages available online. There is one way to copy your package directly to another MOO and two ways of  making your package available online that will be discussed below.

First, let's discuss the relevant pieces of data that are created when you generate package.

**Package Map**

This is a map containing the meta data about your package as well as the package data (your actual package objects/verbs/props/etc). When the package is created, the `package map` is stored as a `map` in `$mpm.last_created_package_map`. This is useful for you if you want to explore the created package map or copy it to another MOO.

**Copying the Package Map**
You can also choose to copy this map over to another MOO with the MOO Package Manager and set the `$mpm.loaded_package` property with the data, thus making it available to `@view-package` and `@install-package`. This is the basic way of moving a package from one MOO to another.

> Warning: This may prove more difficult if you have a very large package. You won't be able to use `@set` to set the property and you will instead need to use an eval such as `;$mpm.loaded_package = packagemapdata`.

**Encoded Package Map**

The `$mpm.last_created_package_encoded` property will contain your encoded package. This is the `package map` after bering converted to json, encoded in binary, and then base64 encoded.

**Making a Single Package Available**

If you would like to make a single package available online, you can copy the contents of `$mpm.last_created_package_encoded` into a plaintext file and upload it to a website, commit it to a Git Repository, or place it wherever you want online. You can then provide the URL to the file to anyone who would like to install it and they can use the `@load-package` command and select the `Load package directly from URL` option, specifying the URL and loading the package.

> Note: If you are using Github, you need the URL to the 'raw' version of your file. You can find this by opening the file in Github, then clicking the 'raw' button. The url will look something like: `https://raw.githubusercontent.com/sevenecks/moo-package-manager/master/packages/diff_utils_1_0`.

**Making a Package Repository**

Alternatively, you can make a package repository. A package repository contains a `package_list` with `package headers`. The `package headers` contain the meta data about packages (name, version, links to where the package can be downloaded, etc). A package repository can have as many packages in it as you like. Those packages must have `package headers` listed for each of the packages.

> Note: For an example of how a package repository might look, look no further than this git repository, which also acts as the `Slither's MOO Packages` repository. The `package_list` containing the `package headers` along with all the package code files is located in the [packages](/packages) directory.

**Generating Package Headers**

`package headers` are generated from the package meta data stored in your instance of the `$mpm`. When you create a package, the `$mpm.created_packages` map is updated with the newly created package. This stores every package you have created, including the multiple different versions of a package you may have created (for instance if you are updating your packages as you fix bugs or add functionality).

To generate package headers for your `package_list` you can simply execute:

```
;$mpm:dump_package_headers()
```

This will display your package headers. Now just simply copy and paste (ensuring one package per line) into a `package_list` file, and place the package list as well as your code, online (we recommend in a GitHub Repo).

You have created a package repository. This package repository will not be available to any MOO Package Manager by default and will need to be added by anyone who wishes to download packages from your repository.

### Adding Package Repositories
By default the only package repository the MOO Package Manager has, point's to this Github Repo, and is registered as `Slither's MOO Packages`. These are trusted packages.

If you would like to add another package repository, you can register that repository with your MOO Package Manager by executing:

```
;$mpm:register_package_repository(name, url);
```

> Warning: Please take care when adding package repositories. You should review any code you are adding to your MOO before using `@install-package`. 

## Caveats
* By default MPM will not serialize verbs/props on ancestors who have a parent of $nothing (IE: #1), as this can make a package huge. This can be over-ridden..

## Unsupported
* Does not support dynamic verb or prop references (IE: $string_utils:("verbname") or $string_utils.(propvar))
* Does not support serializing verbs/props that have binary data in them (IE: ~OE)

## Known Issues
* There is some issue with generate_json where it fails saying invalid argument when passing a package_map at certain times. I'm trying to track down what it is. It might be on very large maps, or it might be certain characters being in the map. If you have this issue and can pin it down let me know.
* If you have a very large package your MOO may not display the entire encoded package, even if you use `notify()`. In these cases please use something like ;notify(me, $mpm.last_created_package_encoded[1..100000]); notify(me, $mpm.last_created_package_encoded[100001..$]) and then join the two (making sure you don't have any line breaks or spaces (sometimes there is a trailing space when you join lines, delete it!)
* If you use GitHub to host your packages, be aware that the `raw.githubusercontent.com` version of your file may be cached for ~5 minutes, so if you update it, you may need to wait a bit after pushing changes.

## Contributing

### Opening Issues
If you have an issue with using the package manager, feel free to open an issue.

### Contributing a Package
For now reach out to Slither on the [ToastStunt Discord](https://discord.gg/XyXP43e).

### Contributing to the Package Manager Source
- If you would like to contribute the best way to do so is by joining the [ToastStunt Discord](https://discord.gg/XyXP43e).
- Pull requests are also welcome

### Contributing a Package Repository
For now reach out to Slither on the [ToastStunt Discord](https://discord.gg/XyXP43e).
