# INIFile
**Parse INI-like lumps in ZScript**

This is a small library for GZDoom's ZScript language, for the purpose of reading INI-style data from lumps at run time.

## Contents

This consists of six files:

* `INIFile.zc` contains `struct INIFile` and `class INISection`. This is the file you want to copy into your mod.

* `demo.zc` adds two actors to demonstrate `INIFile`.

* `DemoMapDescriptions.1.ini`, `DemoMapDescriptions.2.ini`, and `DemoClassDescriptions.ini` contain the data for the demo actors to load.

* `zscript.zc` just `#include`s the other ZScript files.

## Using the Demo

1. Download this repository onto your computer.
2. Load the `INIFile` folder into GZDoom as a mod, with an IWAD that contains maps named `MAP01` and `MAP02`.
3. `map map01`
4. `summon MapDescription` — This will show, on the console, the text in the INI files for `MAP01`.
5. `map map02`
6. `summon MapDescription` — This will show the text for `MAP02` instead.
7. `summon MapDescriptionDump` — This will show the combined contents of the two INI files, as ZScript sees them.
8. `summon ClassDescriptionDump` — This will show the result of performing `MergeByActorClass` on [`DemoClassDescriptions.ini`](DemoClassDescriptions.ini).

## Using in Your Mod

1. Copy `INIFile.zc` into your mod. (Rename it if you wish.)
2. `#include` it from your main ZScript file.
3. Instantiate `INIFile`, like this:
	
	```
	private INIFile data;
	```

4. Load data into it, like this:
	
	```
	override void PostBeginPlay() {
		super.PostBeginPlay();
		data.ReadLumpsNamed("DemoMapDescriptions");
	}
	```

5. Query the data, like this:
	
	```
	String description = data.Get("MAP01", "Description");
	```
	
	If you're looking for a section corresponding to the name of the current map, it needs to be in all-caps. The `Get` method can do that for you:
	
	```
	String description = data.Get(level.MapName, "Description", sectionNameToUpper: true);
	```

## API

There are two types: `struct INIFile` and `class INISection`.

### INIFile

Represents a complete INI file (or several of them). The main methods are:

* `void ReadLumpsNamed(String name)`
	
	Reads INI data from all lumps with the specified name. If there is more than one lump by that name (for example, `DemoMapDescriptions.1.ini` and `DemoMapDescriptions.2.ini`), sections in the different lumps will be merged, and keys will be overridden by lumps that appear later in the load order.
	
	You can call this method more than once, with different lump names, to merge data from several different lumps by name. Once again, data read later overrides data read earlier.

* `String Get(String sectionName, String key, String default = "")`
	
	Gets the value of the given key in the given section. Returns the given `default` value (which is, by default, the empty string `""`) if there is no such section and/or key.
	
	There are also `GetInt`, `GetDouble`, and `GetBool` methods, which return those types instead. Those methods take a default value as well, but will also use it if the key is present but the value is empty (that is, written like `IsAwesome=`).

* `String CurrentMapGet(String key, String default = "")`
	
	Like `Get`, but looks for a section with the same name as the current map (e.g. `MAP01`). If you're using this library to store extra information about maps, and you want to look up that information for the current map, use `CurrentMapGet`.
	
	Also like `Get`, there are the variants `CurrentMapGetInt`, `CurrentMapGetDouble`, and `CurrentMapGetBool`.

* `void MergeByActorClass(bool purgeSuperSubSections = false, bool purgeNoMatch = false)`
	
	See [“Merging Data by Actor Class”](#user-content-merging-data-by-actor-class), below.

You can also look up sections. This is faster if you need to get several values from the same section. The `INIFile` methods for doing so are:

* `INISection Section(String name)`
	
	Looks up a section by name. Returns `null` if there is no such section.

* `INISection CurrentMapSection()`
	
	Looks up a section whose name equals that of the current map, such as `MAP01`. As with the `Section` method, it returns `null` if there is no such section.

### INISection

Represents a single section of an INI file (or several, merged together). `INISection` objects have these methods:

* `String Get(String key, String default = "")`
	
	Gets the value of the given key in this section. Returns the given `default` value (which is, by default, the empty string `""`) if there is no such section and/or key.
	
	There are also `GetInt`, `GetDouble`, and `GetBool` methods, which return those types instead.

* `String CurrentMapGet(String default = "")`
	
	Gets the value of the key with the same name as the current map (e.g. `MAP01`). This is like the `CurrentMapGet` method on `INIFile`, but for looking up keys rather than sections.
	
	As with `Get`, there are the variants `CurrentMapGetInt`, `CurrentMapGetDouble`, and `CurrentMapGetBool`.

* `void Merge(INISection other, bool keepExisting = true)`
	
	Merges another `INISection`'s keys into this one. If `keepExisting` is `false`, key-value pairs in the other `INISection` will replace existing ones in this one; otherwise, existing key-value pairs will not be replaced (default).

## Merging Data by Actor Class

`struct INIFile` has this method:

```cpp
void MergeByActorClass(bool purgeSuperSubSections = false, bool purgeNoMatch = false)
```

This will scan all loaded actor classes, and merge together keys in the corresponding INI sections.

Section names ending with `+` apply to that class and all subclasses.  
Section names ending with `-` apply to that class and all superclasses.

For example, suppose you had this INI file:

```
[Weapon+]
CanHurt=1

[Medigun]
CanHurt=0
```

After performing `MergeByActorClass`, the result will be a section for the `Weapon` class and every subclass, containing the key-value pair `CanHurt=true`, except the `Medigun` class, whose section contains `CanHurt=false` instead. Then, you can load this information:

```cpp
INIFile ini;
ini.ReadLumpsNamed("ClassInfo.ini");
ini.MergeByActorClass();
…
Weapon someWeapon = …;
bool weaponCanHurt = ini.Get(someWeapon.GetClassName(), "CanHurt").ToInt();
```

In this example `weaponCanHurt` will be true unless `someWeapon` is of the class `Medigun`. It'll still be true if `someWeapon` is of some subclass of `Medigun`, because the section header `[Medigun]` applies to only the exact class. You could make it apply to all subclasses of `Medigun` too:

```
[Medigun+]
CanHurt=0
```

Similarly, you can apply a key-value pair to a class and all of its superclasses. For example:

```
[Shotgun-]
Awesomeness=1
```

Now, `Actor`, `StateProvider`, `Weapon`, `DoomWeapon`, and `Shotgun` will all have `Awesomeness=1`, but all other classes will not have an `Awesomeness` key.

## INI Format

An INI file may contain:

* Comments. This is a line that starts with `#`. All text on a comment line is ignored. Note that `#` characters appearing after other text are *not* considered comments.
* Section headers. These are written like `[MAP01]`, for a section named “MAP01”. A section header must appear on a line by itself, with no other text.
* Key-value pairs. These are written like `Description=The first map.`, for a key named “Description” with the value “The first map.” Text before the equal sign is the key, and text after is the value.
* Continuations. A line containing no equal sign is interpreted as another line of the previous key's value. This allows values to be multi-line. (You can see an example of this in [DemoMapDescriptions.2.ini](DemoMapDescriptions.2.ini).)

By default, section and key names are not case sensitive. A section header `[map01]` is equivalent to `[MAP01]`. In most cases, this is what you want. If it's not, section and/or key names can be made case-sensitive by setting the `SectionCaseSensitive` or `KeyCaseSensitive` field on the `INIFile` to `true`. Note that this must be done *before* loading.

Whitespace is trimmed from the beginning and end of each line. Whitespace around the `=` separating a key and value is also trimmed.

Line endings must be either MS-DOS (CR LF) or Unix (LF) style. The line endings used by very old Macs (CR) are not supported. (Modern Macs use Unix style line endings.)

Keys can exist outside of any section (that is, with no section header above them). They will be placed in a section whose name is `""` (the empty string).

### Multi-Sections

Section headers that contain the character `|` are treated specially. Instead of being processed as a single section, a section header like `[Fist|Pistol|Shotgun]` is treated as *multiple* sections (`Fist`, `Pistol`, and `Shotgun` in the example) with identical contents.

In other words:

	```
	[Fist|Pistol|Shotgun]
	SomeKey=SomeValue
	```

…is interpreted as…

	```
	[Fist]
	SomeKey=SomeValue
	
	[Pistol]
	SomeKey=SomeValue
	
	[Shotgun]
	SomeKey=SomeValue
	```

This shorthand should be useful for applying the same settings to several classes/maps/etc at a time.
