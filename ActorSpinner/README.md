# ActorSpinner
**Rotational control for GZDoom actors**

This is a small library for GZDoom's ZScript language, for the purpose of controlling a continuously rotating actor. It consists of a `struct ActorSpinner`, with the following functionality:

**Apply angular velocity.** `ActorSpinner`'s `Vel` field is the angular velocity of the controlled actor. Every time the `Tick` method is called on that `ActorSpinner`, it updates the `Roll` of the controlled actor.

**Accelerate.** The `DesVel` field holds the desired angular velocity. Each tic, `Vel` will increase or decrease toward `DesVel` until reaching it. The `Accel` field determines the rate of acceleration.

**Come to a stop, at a specific angle.** This is the most interesting part. It's called *landing*. When the `LandAt` method is called, rotation will accelerate such that the desired angular velocity is reached at the same moment that the actor's current `Roll` is (close to) the given *landing angle*. For instance, you might want a spinning rod to come to a stop when it's exactly horizontal, so that the player character can realistically pick it up.

## Contents

This consists of three files:

* `ActorSpinner.zc` contains the `struct ActorSpinner`. This is the file you want to copy into your mod.

* `demo.zc` adds a pseudo-weapon demonstrating the functionality. See below for how to use it.

* `zscript.zc` just `#include`s the above two files.

## Using the Demo

The actual `ActorSpinner` code is game-independent, but the demo only works in Doom 1/2/TNT/Plutonia. So:

1. Download this repository onto your computer.
2. Load the `ActorSpinner` folder into GZDoom as a mod, with a suitable Doom IWAD.
3. `give GrabSpinner`
4. Push weapon button 3 (where the shotgun normally lives).
5. Hold down your alt fire button (typically the right mouse button) to select a desired angular velocity. It'll count up from zero as long as you hold the button down, and stop when you let go. The counter is shown in your primary ammo display.
6. Hold down your primary fire button (typically the left mouse button).

A decoration will appear in front of you and spin up to the angular velocity you selected. It will stay at the desired velocity as long as you hold down the button. Once you let go, it will spin down, eventually coming to a stop at the same angle as it started from (i.e. straight upright).

You can reverse the rotation by pressing the reload button. If the actor is already spinning, it will turn around and start spinning the other way.

## Using in Your Mod

1. Copy `ActorSpinner.zc` into your mod. (Rename it if you wish.)
2. `#include` it from your main ZScript file.
3. In the actor class you need to control, add a field to it like this:

	```
	private ActorSpinner spinner;
	```

4. Initialize it. In your `BeginPlay` or `PostBeginPlay` method, set `mo` field to the actor being controlled, and the `Accel` field to the rate of acceleration. If you want it to start spinning right away, also set the `DesVel` field to the desired angular velocity. Example:

	```
	override void BeginPlay() {
		super.BeginPlay();
		spinner.mo = self;
		spinner.Accel = 1;
		spinner.DesVel = 25;
	}
	```

5. Have it do its thing. In your `Tick` method, call `Tick` on the `ActorSpinner` once every tic. Example:

	```
	override void Tick() {
		super.Tick();
		spinner.Tick();
	}
	```

And that's it! Now you can control its spin through the `DesVel` field, and spin it down using the `LandAt` method.

When you're ready to have it spin down, do like this:

	spinner.DesVel = 0;
	spinner.LandAt(0 /* the angle you want to land on */);

If you need to monitor whether it's finished landing, call the `IsLandingInProgress` method. It'll return `false` only when the landing sequence is finished. Example from the demo actor:

	// Begin spinning down.
	#### # 0 { invoker.Spinner.DesVel = 0; invoker.Spinner.LandAt(0.); }
	
	// Wait until spinning down is finished. Jump past this state once that happens.
	#### # 1 A_JumpIf(!invoker.Spinner.IsLandingInProgress(), 1);
	wait;

	// Spinning down done. Remove the demo actor.
	#### # 0 { invoker.SpinningThing.Destroy(); }
	goto Ready;
