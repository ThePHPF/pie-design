# Pickle investigation

Investigate how viable resurrecting Pickle is and adopting it as new PECL
replacement.

## Overview

Pickle is/was an effort led by Pierre
Joye [and contributors](https://github.com/FriendsOfPHP/pickle/graphs/contributors)
to replace PECL, started around 2014 time.

## Resources

* https://wiki.php.net/rfc/pickle
* https://github.com/FriendsOfPHP/pickle

## High Level Differences

### Installation method for end users

**Pickle:** give it a package name/URL/path/etc, it will figure out a
composer.json (if it doesn't already exist),
figure out config.m4/w3, compile and install.

**npecl:** closer to how you use composer; `npecl install <package>`, it will
resolve deps using Composer with Packagist
metadata. Advantage: allows defining downstream deps (although quite uncommon).

In both cases, Composer/Packagist team are willing to help with any support
needed in Composer.

**TLDR:** The end user experience seems very similar, although the mechanism
for "finding source" is extremely
different. The actual build/compile (Linux) and binary finding (Windows) is
quite similar. Pickle seems to rely on the
Windows DLLs existing in PECL repo though; this would need updating if we were
to adopt.

### Conversion from PECL to Composer

**Pickle:** "magic conversion" means maintainers don't have to support Pickle
explicitly. Ext maintainers could use
`pickle convert`, and keep support for PECL by not removing `package.xml`.
Unclear if both would need to continue to be
maintained or not.

**npecl:** no magic conversion planned (at least initially). No
magic `config.m4/.w32` parsing, `composer.json` would
have to define the configure options explicitly. `package.xml` can be kept for
BC, if required, likely both would need
to be maintained.

**TLDR:** npecl bar to entry is slightly higher, as ext maintainers would
explicitly have to support the new format.
This could be a good thing though - as it forces maintainers to think about how
the package is presented, rather than
Pickle guessing (no idea if it could get things wrong, perhaps in practice this
isn't something that would happen?)
In both cases, Windows builds will need some attention from all ext maintainers
ANYWAY, as Win DLLs are no longer added
in PECL.

### Registry maintenance

**Pickle:** Pickle in its current form (generally) expects `pecl.php.net` to be
usable (although it does support other
schemes, e.g. a git repo, path, etc.); a new source-code location scheme would
be needed to use Packagist. However,
this means Pickle could be used for "old PECL" as well as supporting
composer.json

**npecl:** Already planning to use Packagist.

**TLDR:** since Pickle is already established (although it has not released a
first major yet), breaking changes may
be necessary to update the code.

## Main Options

* Fork Pickle, under the PHP Group repo, update namespaces etc., and do the work
  to bring it up to speed. Assuming the
  Licence and copyright permit.
* Start a new project, adopt relevant/useful parts of code where necessary.
  Assuming the Licence and copyright permit.
* Start a new project, without using any Pickle. All code from scratch.

## Conclusion

We have discussed several approaches, and we are working with several members of
the PHP community to ensure which is
the best way forward. Naturally, given the work that Pickle has done in the
past, we did spend some time investigating
the approach Pickle took. After some discussion in the team moving this forward,
comparing various approaches, we have
decided to start a fresh project. However, we are planning to use some of the
parts from Pickle to avoid re-writing
them from scratch. We will of course be retaining Pickle's copyright and
licence, as we highly value the work that
Pickle contributors have made.

## Notes

* Adoption was low (perhaps because not an "official" PHP project?)
* Would need a little updating/refreshing; last release was Jun 2022, so not too
  old, but would need careful review
* Potential approach suggested: start a new project, but grab components /
  useful elements from Pickle, assuming
  the licence allows it. This should be a transparent process, acknowledging the
  original contributors, since a lot of
  work was already put into Pickle.
* Preference seems to be to not parse `config.m4`/`config.w32` (as Pickle does
  currently) but to list explicitly in
  `composer.json` instead
* Pickle's approach seems to be to "convert package.xml to a composer.json"
* Converting a `package.xml` will result in a `composer.json` and a `RELEASES`
  file; the latter is unnecessary in npecl
* Plans in npecl for `composer.json` are more developed (`php-ext` top
  level, `require` operation to match Composer)
* Pickle has a way to determine multiple source locations;
  e.g. `pickle install <tgz-file|pecl|git-url|src-path|pickle?>`
  see `\Pickle\Package\Convey\Command\Type::determine`. If we wanted to retain
  this flexibility, we would implement
  the entire Composer/Packagist dependency resolution here. However, since npecl
  currently intends to lean heavily on
  Composer (and "optionally" Packagist) to resolve dependencies and find the
  package, this flexibility is mostly
  provided out the box
* Pickle did not handle upgrading privileges very well; something we want to
  handle better already:
  ```
  $ php pickle.phar -vvv install /home/james/workspace/scoutapm/scout-apm-php-ext/
  +-----------------------------------+----------+
  | Package name                      | scoutapm |
  | Package version (current release) | 1.10.0   |
  | Package status                    | stable   |
  +-----------------------------------+----------+
  [whether to enable scoutapm developer build flags] (default: no):
  The following error(s) happened: make install failed
  Would you like to read the log?yes
  <snip>
  1: make install
  2: Installing shared extensions:     /usr/lib/php/20230831/
  2: cp: cannot create regular file '/usr/lib/php/20230831/#INST@102233#': Permission denied
  2: make: *** [Makefile:89: install-modules] Error 1
  ```
* Pickle complexity much higher, possibly due to its flexibilty, and more
  development to add bells and whistles,
  e.g. `pickle completion`, presumably for shell auto-completion, but it didn't
  work for me (got message
  `Detected shell "bash", which is not supported by Symfony shell completion (supported shells: "").`).
* Pickle has a `release`, which appears to make a `tgz` somewhat compatible, but
  I did note that many build artefacts
  are included in this (presumably these have not been filtered out anywhere,
  not sure if there is a mechanism for this)
* Pickle has a `validate` which only operates if `package.xml`, which seems at
  odds with the goal of moving to use
  `composer.json` - although perhaps provided only for backward compatibility.
* From a practical standpoint, it would make sense to move from
  the `FriendsOfPHP` repo to `php` on GitHub, if it was
  to be adopted officially by PHP project, either by way of a fork, or
  discussing with maintainers first.
* Pickle does have existing tests, although it uses `atoum` which is not
  something many people are familiar with,
  compared to phpunit for example.
* Pickle licence is New BSD - is this compatible with PHP Licence? Adopting
  Pickle (or even taking parts of it) would
  require licences to be compatible; also bear in mind if licences are NOT
  compatible, we can't adopt any amount of the
  code, presumably without agreement from all the contributors...
    * New BSD is probably compatible. However, using any code from Pickle would
      either need re-licensing to the PHP
      Licence (mentioned in the doc), or we'd have to adopt both licences I
      think? Most likely, we'd need to mark in the
      LICENCE file both the PHP Licence and New BSD licence (for code
      originating from Pickle) as well as the copyright
      for both.
