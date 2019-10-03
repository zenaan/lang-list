# Canonical Computer Syntax UID List

Syntax in the context of the lang-list `syntax_uid` means an identified computer language (or
data or information) representation format which is viewable by humans.

**`syntax_uid`** is:
 - a globally unique syntax identifier
 - protocol versioned
 - release versioned
 - inclusive of every identified syntax
 - inclusive of every syntax category
 - internationalized
 - includes (optional) `deprecated`, `obsolete`, `unknown` and `alias_of` fields

Syntax categories include:
 - programming languages, including low level (e.g. asm) languages
 - document markup (/markdown) styles
 - tagged markup languages
 - common combination formats (e.g. `html_cheetah`)
 - log file formats
 - program output formats
 - configuration syntaxes
 - data representation formats
 - protocol formats



## Background

[EditorConfig](https://editorconfig.org/) could use a `syntax_uid` for mapping that which is
inter alia referred to as _file type_, _[programming] language [name]_, _syntax_, _[file]
format_ or sometimes even _[file] extension_.  The concept handled here is "display syntax",
i.e. a syntax as viewed by humans, and is hereinfter referred to as "**syntax**".
Other programs may also find `syntax_uid` useful.

Commonly used syntax names often clash e.g. [APM](https://en.wikipedia.org/wiki/APM).

File extensions are far from unique - there are 1000s, possibly 10s of 1000s.

The concept "file type" or "[file format](https://en.wikipedia.org/wiki/File_format)" includes
concepts such as file extension, encoding, and textual vs binary representation; e.g. an "XML
file" may be stored as a "plain text" file which may be encoded say in UTF-8 or UTF-16LE, and
further it may be stored in a compressed format, either textual or binary (".zip" for one simple
binary compression format example).

Given an arbitrary file, there are many ways to [identify file
type](https://en.wikipedia.org/wiki/File_format#Identifying_file_type), including any preferred
combination of the following or other file type identification systems:
 - filename extension
 - internal metadata
 - internal "magic numbers"
 - external metadata
 - MIME types and other 'standard' mappings
 - operating system specific type-codes
 - operating system- and filesystem-specific extended attributes

The [editorconfig](http://editorconfig.org/) project, in particular editorconfig#190 and
editorconfig#404, inspired the `lang-list`.

The initial creation has been some rather tedious effort - from here the list should be
relatively stable, broadly applicable, and easy to maintain and translate.



## lang-list Data Structure

Syntax uids primary store is in the file `syntax_uids.yaml`, a [YAML](https://yaml.org/) file.

Per-application `syntax_uid` maps are in the subdirectory `app_maps/` with one YAML file per
application, named `${APPNAME}.yaml` .

Boolean values may be `true` or `false` only.

The lang-list data structures are as follows:

 - **`syntax_uids`** (`syntax_uids.yaml`): _map_, keyed by field name:

   1. `protocol`: _integer_, currently `0`, monotonically increasing if necessary

   1. `version`: _integer_, initially `0`, monotonically increasing with each public release

   1. `released`: _integer_, release date in "ISO format, no punctuation" i.e. `YYYYMMDD`

   1. `uids`: _map_, keyed by `$syntax_uid`


 - **`$syntax_uid`** (`uids` key): _map_, keyed by field name:

   1. `unknown`: _boolean_, optional, `true` if this uid is not (yet) properly identified,
      `false` otherwise or if not present

   1. `alias_of`: _string_, optional, MUST be non-empty if present; and the value of the
	  field is the `syntax_uid`, the key is an alias of that uid

   1. `deprecated`: _boolean_, optional, `true` if this `syntax_uid` is deprecated,
      `false` otherwise or if not present;
	  `deprecated` MUST only be present if `alias_of` is also present

   1. `family`: _string_, optional, if present, the value is the `syntax_uid` of the "syntax
      family" with which this `syntax_uid` is associated

   1. **`name`**: _map_ keyed by [IETF BCP 47 language tag](https://en.wikipedia.org/wiki/IETF_language_tag)
	  (a _string_ identified herein as `$LANG`), with a minimum inclusion of
	  "`{name: {en: "English syntax name"}}`",
	  including _only_ the LANG `en` in the root `syntax_uids.yaml` file,
	  `name` is optional in the case of an alias or unknown `syntax_uid`, **mandatory** otherwise

   1. `supercedes`: `list` of  _string_, optional, list of obsolete `syntax_uid` syntaxes
      superceded by this `syntax_uid`

   1. `influenced_by`: `list` of _string_, optional, a list of `syntax_uid` identifying
      languages which influenced this language

   1. `obsolete`: _boolean_, optional, `true` if this syntax is "obsolete",
      `false` otherwise or if not present


 - **`app_map`** (`app_maps/${APPNAME}.yaml`): _map_, keyed by field name:

   1. `protocol`: _integer_, currently `0`, only increase if necessary

   1. `version`: _integer_, initially `0`, monotonically increasing with each public release of
      this specific app_map

   1. `released`: _integer_, release date in "ISO format, no punctuation" i.e. `YYYYMMDD`

   1. `syntax_uid_to`: _map_, keyed by `$syntax_uid`


 - **`syntax_uid`** (`syntax_uid_to` key): _map_, keyed by field name:

   1. `map`: _string_, mandatory, the application's syntax name corresponding to this
      `syntax_uid`

   1. `run`: list of _string_, optional, list of commands and/ or settings to apply, in
      combination with the `map` value, in order to cause `syntax_uid` to apply



## lang-list Lookup Protocols

Protocols for using `syntax_uid` are necessary to provide for:
 - backwards compatibility
 - forwards compatibility
 - deprecatability
 - changing, adding, and removing of aliases

In the protocols (as pseudo code functions) below:
 - `.` (period) means lookup or access a map key (field)
 - `:=` means assign RHS to LHS
 - `==` and `!=` are equality comparison
 - `""` means the empty string
 - `$` means dereference or "get the value stored in this variable"
 - `NULL` means a non-existent map or field
 - The error() function MUST show an error where errors are normally shown.
 - The warn() function MUST show a warning where warnings are normally shown.


### `str_to_syntax_uid(STR)  # Convert STRing to syntax_uid`
```
str_to_syntax_uid(STR) {
	# Convert STRing to syntax_uid:
	syntax_uid := syntax_uids . uids . $STR . alias_of
	if syntax_uid != ""
	then
		if syntax_uids . uids . $STR . deprecated == "true"
		then
			warn("syntax_uid '$syntax_uid' is deprecated and will be removed, use '$syntax_uid' instead")
		endif
	else
		syntax_uid := $STR
	endif
	if syntax_uids . uids . $syntax_uid == NULL
	then
		error("syntax_uid '$STR' does not exist")
	endif
}
```

### `str_to_app_sid(STR)  # Convert STRing to app syntax id`
```
str_to_app_sid(STR) {
	# Convert STRing to app syntax id
	syntax_uid := str_to_syntax_uid($STR)
	app_sid := app_map . syntax_uid_to . $syntax_uid . map
	if app_sid == "" then warn("$APP does not support the syntax_uid '$syntax_uid'")
}
```

### `app_sid_to_syntax_uid(SID)  # Convert app_SID to syntax_uid`
```
app_sid_to_syntax_uid(SID)
	# Convert app syntax id to syntax_uid
	syntax_uid := app_map . syntax_uid_to . first_key_for_value($syntax_uid)
	if syntax_uid == "" then error("$APP syntax id '$SID' does not have a lang-list syntax_uid")
}
```



## Example with EditorConfig

For a new EditorConfig application plugin, copy `template-app_map.yaml` to
`app_maps/program_name.yaml` , and update each `syntax_uid_to` entry as follows:
 - comment out each line not supported by your program, using "`#`" (sharp/hash sign)
 - edit the string after "`map: `" to match your program's name for that `syntax_uid` seen near
   the start of the line; be sure to keep a space character after the "`:`" (colon) as well as
   keep the trailing "`}`" (closing brace).

Next implement the above lookup protocol functions, `str_to_syntax_uid(STR)`,
`str_to_app_sid(STR)` and `app_sid_to_syntax_uid(SID)`, in your plugin
(if they are not already available in your editor).

Voi la, the application's syntax id and the corresponding lang-list `syntax_uid` (e.g. `java`)
can now be used to lookup editorconfig language-specific settings, e.g. in an editorconfig group
named say "`[: syntax_uid=java]`";

and similarly the corollary, an editorconfig file-group setting for `syntax` can ensure the
correct syntax highlighting in your editor.

Example:
```ini
	[src/sh/*]
	syntax = sh
	indent_size = 8

	[src/bash/*]
	syntax = bash
	indent_size = 4

	# use this if you are happy with editor auto detection for Java-syntax files:
	[: syntax_uid=java]
	indent_size = 3
```

Note: The `syntax_uid.yaml` file is not normally loaded by editorconfig plugins, only the
`app_maps/$EDITOR.yaml` file should be needed.



## Jobs - `app_maps/${MY_EDITOR}.yaml` - `l10n/${LANG}/syntax_uids.yaml`

To apply the lang-list to another editor, copy `app_maps/vim.yaml` to
`app_maps/${MY_EDITOR}.yaml` and update each `map:` value to the corresponding syntax ID for
your editor.  Implement the lookup protocol functions.
Find happiness in gratitude.

To translate `syntax_uid` names, see section "Localisation..." below.
Find happiness in gratitude.



## Indentation

YAML files per specification, are unfortunately space indented (control freaks will control in
freaky ways). Although some (many?) YAML parsers may treat tabs as spaces, `languages.yaml`
shall be a YAML specification compliant file. See here:

 - https://stackoverflow.com/questions/19975954/a-yaml-file-cannot-contain-tabs-as-indentation
 - http://yaml.org/faq.html



## Localisation - `l10n/` Subdirectory

Localization translation files translating `syntax_uid` names, README.md and other material,
are located in files with the same name and relative location, as the corresponding file in the
root directory, but instead located under the `l10n/${LANG}/` directory.

To be clear, `syntax_uid` name translations are _not_ included in the root `syntax_uids.yaml`
file, but in the appropriate `l10n/${LANG}/syntax_uids.yaml` file, and similarly
`l10n/${LANG}/README.md` for a translation of the `README.md` file.

Such localisation files must be:
 - `UTF-8` encoded
 - "three space chars" indented
 - For `YAML` files translations, YAML files with the same structure as the original file.

`$LANG` is the IETF BCP 47 language tag identifying the corresponding localized human language.
For information about the `IETF BCP 47` "human language" tag, see:

 - https://en.wikipedia.org/wiki/IETF_language_tag
 - http://cldr.unicode.org/index/cldr-spec/picking-the-right-language-code
 - https://en.wikipedia.org/wiki/Lists_of_ISO_639_codes
 - https://salsa.debian.org/iso-codes-team/iso-codes
 - https://stackoverflow.com/questions/2511076/which-iso-format-should-i-use-to-store-a-users-language-code

Each localization file must have the same structure as `languages.yaml` but less content. In
particular the file contains the following YAML mapping keys:

 - `protocol: 0` - _integer_, mandatory
 - `version: 0` - _integer_, mandatory, where the version number starts at `0` and is the version for this
   localisation file
 - `name: {$LANG: "..."}` - _string_, optional, translation of "computer language syntax UID"
 - `uids:` - _map_, mandatory, keyed by `$syntax_uid`
 - Do NOT include the alias and unknown groups (the first two syntax uid groups).
 - Each `$syntax_uid` contains ONLY the `{name: {$LANG: "..."}}` map
 - That is, remove all fields other than the `name` map, such as `family`, `successor`,
   `influenced_by` and `supercedes`, and comments (except for the section dividers and copyright
   header line) should also be removed.
 - See `templates/syntax_uids.yaml` for an example to start from (copy this file and edit
   accordingly), but do note that entries in the root `syntax_uids.yaml` will be added, and some
   changed, over time;
   `git` commands can be used to identify such differences.



## Feature requests and support

For feature requests and support, please search the [lang-list issues
list](https://github.com/zenaan/lang-list/issues) and if your feature or issue is not present,
add a new issue.

If you wish to register your interest in an issue, click the "subscribe" button -
please do _not_ add "+1"s or "mee too" comments.

You may set your GitHub settings to receive emails for updates on issues you are subscribed to.



## License
`lang-list` is licensed by the `GNU Lesser General Public License version 3` aka `LGPL3`.

The LGPL reflects the need for lang-list to be usable as a library in relation to any other
software license.



## See also
See also:

 - https://en.wikipedia.org/wiki/List_of_programming_languages
 - https://en.wikipedia.org/wiki/Markup_language
 - https://en.wikipedia.org/wiki/List_of_markup_languages
 - https://en.wikipedia.org/wiki/Common_Log_Format
 - https://en.wikipedia.org/wiki/Configuration_file
 - https://medium.com/web-development-zone/a-complete-list-of-computer-programming-languages-1d8bc5a891f
 - https://github.com/garabik/grc

