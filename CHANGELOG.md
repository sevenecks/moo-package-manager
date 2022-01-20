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
