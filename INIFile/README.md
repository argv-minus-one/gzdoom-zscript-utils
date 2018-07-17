# INIFile
**Parse INI-like lumps in ZScript**

This is a small library for GZDoom's ZScript language, for the purpose of reading INI-style data from lumps at run time.

## Contents

This consists of five files:

* `INIFile.zc` contains `struct INIFile` and `class INISection`. This is the file you want to copy into your mod.

* `demo.zc` adds two actors to demonstrate `INIFile`.

* `DemoMapDescriptions.1.ini` and `DemoMapDescriptions.2.ini` contain the data for the demo actors to load.

* `zscript.zc` just `#include`s the other ZScript files.

## Using the Demo

1. Download this repository onto your computer.
2. Load the `INIFile` folder into GZDoom as a mod, with an IWAD that contains maps named `MAP01` and `MAP02`.
3. `map map01`
4. `summon MapDescription` — This will show, on the console, the text in the INI files for `MAP01`.
5. `map map02`
6. `summon MapDescription` — This will show the text for `MAP02` instead.
7. `summon MapDescriptionDump` — This will show the combined contents of the two INI files, as ZScript sees them.

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

The main methods on `struct INIFile` are:

* `void ReadLumpsNamed(String name)`
	
	Reads INI data from all lumps with the specified name. If there is more than one lump by that name (for example, `DemoMapDescriptions.1.ini` and `DemoMapDescriptions.2.ini`), sections in the different lumps will be merged, and keys will be overridden by lumps that appear later in the load order.
	
	You can call this method more than once, with different lump names, to merge data from several different lumps by name. Once again, data read later overrides data read earlier.
* `String Get(String sectionName, String key)`
	
	Gets the value of the given key in the given section. Returns `""` (the empty string) if there is no such section and/or key.
* `String CurrentMapGet(String key)`
	
	Like `Get`, but looks for a section with the same name as the current map (e.g. `MAP01`). If you're using this library to store extra information about maps, and you want to look up that information for the current map, use `CurrentMapGet`.

You can also look up sections. This is faster if you need to get several values from the same section. The `INIFile` methods for doing so are:

* `INISection Section(String name)`
	
	Looks up a section by name. Returns `null` if there is no such section.

* `INISection CurrentMapSection()`
	
	Looks up a section whose name equals that of the current map, such as `MAP01`. As with the `Section` method, it returns `null` if there is no such section.

`INISection` objects have these methods:

* `String Get(String key)`
	
	Gets the value of the given key in this section. Returns `""` (the empty string) if there is no such key.

* `String CurrentMapGet()`
	
	Gets the value of the key with the same name as the current map (e.g. `MAP01`). This is like the `CurrentMapGet` method on `INIFile`, but for looking up keys rather than sections.

## INI Format

An INI file may contain:

* Comments. This is a line that starts with `#`. All text on a comment line is ignored. Note that `#` characters appearing after other text are *not* considered comments.
* Section headers. These are written like `[MAP01]`, for a section named “MAP01”. A section header must appear on a line by itself, with no other text.
* Key-value pairs. These are written like `Description=The first map.`, for a key named “Description” with the value “The first map.” Text before the equal sign is the key, and text after is the value.
* Continuations. A line containing no equal sign is interpreted as another line of the previous key's value. This allows values to be multi-line. (You can see an example of this in [DemoMapDescriptions.2.ini](DemoMapDescriptions.2.ini).)

Section and key names are **case sensitive**. A section header `[map01]` is *not* equivalent to `[MAP01]`. Note that `level.MapName` is *not* guaranteed to be all-uppercase, so if you use it to look up a section or key, you need to do something like this:

	let mapName = level.MapName;
	mapName.ToUpper();
	let value = data.Get("SomeSection", mapName);

Whitespace is trimmed from the beginning and end of each line. Whitespace around the `=` separating a key and value is also trimmed.

Line endings must be either MS-DOS (CR LF) or Unix (LF) style. The line endings used by very old Macs (CR) are not supported. (Modern Macs use Unix style line endings.)

Keys can exist outside of any section (that is, with no section header above them). They will be placed in a section whose name is `""` (the empty string).
