## [1.5.0] - TBD
* fixed comments in package_build_referenced_verbs_map 
* added :match_with_regex verb so we can start centralizing many of the :match_whatever verbs that have basically the same code
* removed `match_this_verbs_in_code`, added .REGEX_MATCH_THIS_VERBS prop and replaced with calls with :match_with_regex
* removed `match_this_props_in_code`, added .REGEX_MATCH_THIS_PROP prop
* removed `match_this_dynamic_prop_calls_in_code`, added .REGEX_MATCH_DYNAMIC_PROP_CALLS
* removed `match_this_dynamic_verb_calls_in_code`, added .REGEX_MATCH_DYNAMIC_VERB_CALLS
* removed `match_cored_props_in_code` and replaced with prop and call to match_with_regex
* removed `match_cored_verbs_in_code` and replaced with prop and call to match_with_regex
* fixed a bug in `package_build_referenced_verbs_map` where it was not using setadd properly, causing a tb
* fixed a bug in `package_build_referenced_props_map` where it was not using setadd properly, causing a tb
* updated :package_serialize to only show messages about all cored props/verbs/etc if lists are not empty
* updated :package_serialize with ansi colors, and updated how some of the messages are presented
* updated debugging info in :package_serialize to have colors, only show when present
* added color to package_serialize_verb_data debugging on dynamic prop/verb calls
* added command line argument --skip-generic-dependencies to allow skipping of utility packages any ToastCore/LambdaCore is going to have
* added `generic_dependencies` prop with the basic ToastCore verbs from {"building_utils", "command_utils", "gender_utils", "list_utils", "match_utils", "object_utils", "perm_utils", "seq_utils", "set_utils", "string_utils"}
* added `skip-generic-dependencies` command line option to readme
* updated confirm_package_creation_options with `skip-generic-dependencies`
* removed useless debugging from :graph_verb_exists and :graph_prop_exists that was spammy
* added `--convert-short-cored-to-long` command line option, updated guide, added to confirm_package_creation_options
* repackaged Code Scanner @ 1.2 to include needed meta info
* added --ignore-all-cored-verbs to ignore cored verbs referenced in serailized code
* repackaged Read Selection @ 1.2 to include meta info

## [1.4.0] - 2022.01.30 12:34PM
* Fixed typo in @make-package args
* Made $mpm:dump_package_headers do a sort on the headers first so they are alphabetical
* Updatd @load-package to use $string_utils:left to format the # for package list
* Updated dump_package_headers to dump the package_id instead of the hash
* Updated @Load-package to show [installed] and [update available] based on status of installed packages
* added lastest_package_installed to return the most recent installed version of a package based on version #
* Added Handle Lagging Task System package at 1.0
* Added Handle lagging task wizard verbs package @ 1.0
* fixed match_cored_props_in_code so that it doesn't match a trailing ,}" when matching
* fixed @make-package argument for --reset-prop-values
* made cored prop references actually get serialzied when referenced from another objects verbs, this also fixes an issue where a dependent object that defined only props would not be included in the package
* Added suspend_if_needed to display_verb_data
* A few other QOL/bug fixes that aren't worth noting as they don't change behavior in an incompatible way
* updated package rewire error when fewer ancestors exist on target MOO, to be more clear

## [1.3.1] - 2022.01.24 9:01PM
* Added MIT license
* Fixed formatting in README
* Added The Almighty Scheduler package
* Added The Almighty Scheduler Wizard Verbs package
* Updated Enhanced Coder verbs to 1.1 - added @tree
* Fixed all packages to have a package id

## [1.3.0] - 2022.01.23 2:25PM
* Added String Utils Enhanced @ 1.1
* Added Enhanced Coder Verbs @ 1.0
* Added $mpm:manage_packages() to let users review/edit/delete packages from .created_packages and .installed_packages
* Added package_id, package_changelog to log_created_package, causing it to be stored in .created_packages
* Added package_id to log_installed_package, causing it to be stored in .installed_packages
* Added package_changelog, package_url, mpm_version to .installed_package map on install
* Added @manage-packages verb for MPM Wizard Verbs, acts as a wrapper for $mpm:manage_packages
* Added Manager Packages section to readme
* Moved loading/installing/viewing packages section above Making a package section in README, since people are probably more interested in that to start.
* Moved log_installed_package above post install note and post install verb in @install-package to facilitate better logging of MPM being upgraded
* Added handle_post_install to $mpm to make copying over props and data easier and not require user to do anything besides confirm
* Updated wording around hash mismatch in @install-package
* Updated wording around previous versions not detected
* removed debugging from $mpm:slice_ancestor_name_number_aliases_and_cored
* Added warning in Installing a Package readme section about updating cored references
* Updated wording in verbs for installing packages to make them more concise and clear, and add ansi
* Made :find_object always return $nothing when is_mpm_update returns true, so we don't prompt the user to use the existing object, ever
* Made package_update_object never install cored references when upgrading $mpm
* Added :display_encoded_package() and added option to call it when @make-package completes
* Removed completed TODOs from @make-package 
* Added --strip-trailing-comments option to @make-package to remove trailing comments in verbs, added to readme as well, added :strip_trailing_comments_in_code to support this
* Updated @make-package examples
* Updated MPM to 1.3
* Updated MPM Wizard verbs to 1.3
* Added new option to :is_mpm_update to make it run silently without user prompts, since we call it multiple times now
* Updated readme to point out $mpm:display_encoded_package() instead of the evals it had in place of that in the warning inside Making a Package
* Updated /code/moo_package_manager to 1.3
* Updated /code/moo_package_manager_wizard_verbs to 1.3
* Updated readme upgrading instructions
* Full Release of MPM 1.3

## [1.2.0] - 2022.01.20 5:29PM
* Removed --ignore-all-props from @make-package argument list since it doesn't exist
* Added String Utils Enhanced 1.0 package
* Packaging up all options used when creating a package so that they can be used when installing a package
* Made --dont-serialize-ancestry option detected when installing a package so we don't try to rewire the parentage when it doesn't match, and so it doesn't try to update ancestors which won't actually exist in the package
* Updated readme to callout how to handle large package displays via notify
* Added $mpm:version verb to return MPM version
* Added check for package version against $mpm:version to block installing packages created with a newer version of MPM than is installed. this will prevent compatibility issues.
* Updated @load-package to show a 404 error when detected and added try/catch to offer to display the returned data when there is a processing error (or we hit a redirect, etc)
* Updated @view-package to support showing verb and prop data. Added $mpm:display_property_data and $mpm:display_verb_data to support this.
* Added ANSI support for Sindome style and DarkOwl style color utils
* Completely redid @view-package to utilize ansi, format information better, provide info in a better order, and added displaying of options the package was created with (new verb $mpm:display_package_options supports this)
* Updated example @make-package for creating a new version of $mpm
* Added MPM Wizard Verbs 1.2
* Added MPM 1.2
* Added Upgrading MPM section to readme
* Updated warnings section
* renamed moo_package_manager_user_code to moo_package_manager_wizard_verbs
* fixed a few typos and poorly written sentences in the readme
* full editing pass on readme
* updated About section of readme

## [1.1.0] - 2022.01.19 7:22PM
* Fixed some typos in commented code
* Added --ignore-all-cored-props option
* Added --dont-serialize-cored-aliases option
* Added --ignore-all-non-cored-props option
* Added & removed some TODOs
* Added --only-include-prop-list option
* Added --post_install-verb option
* Added --target-self option
* Fixed a bug with detecting dynamic verb calls
* Fixed bug in @view-package when trying to display anobject with $nothing as parent
* Added --allow-dynamic-verb-calls option
* Added --allow-dynamic-prop-calls option
* Made MPM actually check for dynamic prop calls
* Added debug for when we start serailizing props, including all props we are serializing
* Added --dont-serialize-ancestry  option
* Updated how options are displayed when confirming package creation options, now only displaying if true/has a value to avoid spam
* Added package_id which will be unique for each package, and carry over when packages are updated
* Fixed @load-package message to point to @install-package and not @import-package
* Added new prompts and checks around --target-self packages on make and install
* Added new MPM 1.1 package
* Added new MPM Wizard Verbs package
* Updated code/moo_package_manager_user code to MPM Wizard Verbs 1.1
* Updated code/moo_package_manager code to MPM 1.1
* Added KNOWN ISSUES section
* Added Change Log section
* Added Upgrading section

## [1.0.2] - 2022.01.19 1:41AM
* Added read selection package
* Added code scanner package

## [1.0.1] - 2022.01.18 9:40PM
* Added moo_package_manager_1_0 to packages
* Updated package headers
* Updated readme with more info
* Added more details on using MPM
* Added install instructions
* Added code directory
* Added moo_package_manager code
* Added moo_package_manager_user code

## [1.0.0] - 2022.01.18 7:00PM
* Added initial full readme
* Added initial package_list
* Added diff utils & map utils packages
