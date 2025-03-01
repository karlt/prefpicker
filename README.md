PrefPicker
==========
[![Build Status](https://travis-ci.com/MozillaSecurity/prefpicker.svg?branch=master)](https://travis-ci.com/MozillaSecurity/prefpicker)
[![codecov](https://codecov.io/gh/MozillaSecurity/prefpicker/branch/master/graph/badge.svg)](https://codecov.io/gh/MozillaSecurity/prefpicker)
[![Matrix](https://img.shields.io/badge/dynamic/json?color=green&label=chat&query=%24.chunk[%3F(%40.canonical_alias%3D%3D%22%23fuzzing%3Amozilla.org%22)].num_joined_members&suffix=%20users&url=https%3A%2F%2Fmozilla.modular.im%2F_matrix%2Fclient%2Fr0%2FpublicRooms&style=flat&logo=matrix)](https://riot.im/app/#/room/#fuzzing:mozilla.org)
[![PyPI](https://img.shields.io/pypi/v/prefpicker)](https://pypi.org/project/prefpicker)


Manage & generate prefs.js files for use when testing Firefox. This tool is intended to simplify the use and tracking of prefs used by
our fuzzing tools. The template files can be modified to allow the creation of custom prefs.js files without
the need to maintain a separate mostly duplicate version of a prefs file.

YAML Template Structure
-----------------------

The template document is made up of variants, prefs and values.

`pref` is the name of the preference that will be added to the prefs.js file. This is an unquoted string.
Valid prefs can be found in [all.js](https://hg.mozilla.org/mozilla-central/file/tip/modules/libpref/init/all.js) or in [StaticPrefList.yml](https://hg.mozilla.org/mozilla-central/file/tip/modules/libpref/init/StaticPrefList.yaml).

`value` can be a bool, int, string or null. Adding multiple potential values is possible.
When multiple values are present one is chosen at random when generating the output.
Using a value of null will skip adding the pref to the output prefs.js file.

`variant` is a subset of values to be used in place of the default values.
The default variant is used unless a variant is specified.

There are a few mechanisms in place to help keep the file in order:
- All prefs must have a default variant
- All variants be defined in the variant list
- All variants in the variant list must be used
- All variants must be a list and contain values

```yml
# example.yml
variant:      # list for extra variants, default is implied
 - alt        # name of variant
pref:
 pref.name:   # unquoted name of the pref used in prefs.js
   default:   # variant definition, default is required
     - 0      # potential value
   alt:       # extra optional variant
     - 1      # if multiple values are defined one is chosen randomly
     - null   # null is a special case meaning do not add the pref
```

Quick Setup
-----------

Use pip to install prefpicker.

```bash
user@machine:~/$ pip install prefpicker
```

Examples
--------

Use a built-in [template](https://github.com/MozillaSecurity/prefpicker/tree/master/prefpicker/templates) to generate an up-to-date `prefs.js` file.

```bash
user@machine:~/$ python -m prefpicker browser-fuzzing.yml prefs.js
```

Or generate a `prefs.js` file from a custom template using the `webrender` variant:

```bash
user@machine:~/prefpicker$ python -m prefpicker custom/template.yml ~/Desktop/prefs.js --variant webrender
Loading 'template.yml'...
Loaded 255 prefs and 5 variants
Generating 'prefs.js' using variant 'webrender'...
Done.
```

The resulting `prefs.js` file is ready to be used with Firefox. It will look something like this:

```js
// Generated with PrefPicker @ 2020-02-08 00:50:29 UTC
// Variant 'webrender'
/// ... snip
user_pref("fuzzing.enabled", true);
/// ... snip
// 'gfx.webrender.all' defined by variant 'webrender'
user_pref("gfx.webrender.all", true);
/// ... snip
// Fingerprint '13370ddba11'
```
