# PuffTracker
**Keep track of persistent puffs/projectiles**

This library allows you to keep track of puffs (or other actors) that were previously spawned, so that you don't spawn more than one on the same actor at the same time. It consists of a `struct PuffTracker`, with the following functionality:

**Track a new puff.** The `Add` method adds a puff to the array of tracked puffs. It sets the puff's `tracer` pointer to the target actor, and the `target` pointer to the source of the puff. It also sets the `master` pointer to the originating weapon, if one is provided.

**Find a tracked puff for an existing actor.** The `Find` method searches for a previously-added puff attached to the given target actor. It can optionally filter by the puff's class.

**Move a puff to the nearest point on the target from the source.** The `PlaceAtNearestPoint` method moves a puff so that it is on the target actor, positioned between the target and the source according to the target's radius and height. If no such puff exists, one is spawned.

## Contents

This consists of three files:

* `PuffTracker.zc` contains `struct PuffTracker`. This is the file you want to copy into your mod.

* `demo.zc` adds a pseudo-weapon demonstrating the functionality. See below for how to use it.

* `zscript.zc` just `#include`s the above two files.

## Using the Demo

The actual `PuffTracker` code is game-independent, but the demo only works in Doom 1/2/TNT/Plutonia. So:

1. Download this repository onto your computer.
2. Load the `PuffTracker` folder into GZDoom as a mod, with a suitable Doom IWAD.
3. `give Puffer`
4. Push weapon button 3 (where the shotgun normally lives).
5. Hold down your fire button, and aim at some monsters.

A continuously looping puff will appear on each monster you aim at, even after your aim moves away, and “sticks” to the monster as it moves around. Only one puff is spawned per monster, and the pseudo-weapon keeps track of which monsters you've “tagged”. All puffs are deleted when you let go of the fire button.

## Using in Your Mod

1. Copy `PuffTracker.zc` into your mod. (Rename it if you wish.)
2. `#include` it from your main ZScript file.
3. In the class from which you need to track puffs (a weapon, a monster, etc), add a field to it like this:

	```
	private PuffTracker pt;
	```

4. Add puffs to track them.
	* If you want a puff to be placed closest to the source like in the demo, call `pt.PlaceAtNearestPoint` with the source actor, target actor, and the puff class to spawn. (You can also give flags, which are the same `PF_*` flags as understood by the `Actor.SpawnPuff` method. If you want to keep track of the weapon that the puffs originated from, supply that as well.)
	* If you want to spawn and position your puffs some other way, and just want to keep track of them, call `pt.Add` with the puffs and related actors.
5. Look up the puff attached to a specific target using `pt.Find`, or iterate over all tracked puffs in the dynamic array `pt.Puffs`. Note that some entries in the `Puffs` array may become `null` when the puffs are destroyed, and then become non-`null` when a new puff is inserted into the empty array slot, so check for nulls as needed.
