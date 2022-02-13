# TODO List for MPM

## @make-package
- add dependency check to see if we've already stored the cored references to something;
- figure out how we want to deal with bf_ verbs;
- we need to add a check for if the verb's name has changed ie: $command_utils:sin suspend_if_needed to confirm they want the new verbs name;
- make cored reference updates happen at the end of the package;
- rename fully serialize parent to fully serialize ancestry;
- store if the programmer needs to be a wiz, essentially if any verbs are owned by a $wiz;
- test with verbs and props that have an extra : like for waifs;
- figure out how to handle binary in props -- use @compare after serializing $su which has a binary_conversions code;
-- ;$critical($su.ascii == decode_binary(\" !\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~7E~09\"));
-- decode_binary may be returning a space for a tab?;
-- something with how it's decoding, where we might need to reencode props?;
- make sure we are calling out ToastStunt in the repo and in the lambda-moo guide;
- make sure when we are adding new names to verbs we call it out;
- add ability to save default options that are used, such as ignore-prop-list;
- figure out what happens when we are dealing with waifs;
- add ability to ignore a list of objects;

## @load-package

load a package for processing;
- make a strip_binary verb that loops over a list and pulls out the strings;
- figure out how we are going to load the package header information;
- trim incoming data of spaces;

## @view-package

- make this show cored references and parent heirarchy

## @install-package

- add ability to scan for verbs that need wizard perms before denying creation;
- add dry run option which is default, and real run option which needs to be passed in;
- add ability to specify a non wizard owner of non wizard verbs;
- installing on toaststunt with toast $recycler gives weird results when doing $reyccler:valid;
- when an object is detected during install, we should give the option of providing an alternative object. see case of #20 being String Utils but having a different $string_utils;

## $mpm:package_serialize

- we need to deal with verbs with multiple names
- consider adding a commandline argument to ignore this verbs that are referenced but not included to avoid the warning when they aren't included in the package
