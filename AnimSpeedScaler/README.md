# AnimSpeedScaler
**Smooth scaling of actor animation speeds**

This is a small library for GZDoom's ZScript language, for the purpose of scaling the speeds of an actor's animations by a given amount.

## Contents

This consists of four files:

* `AnimSpeedScaler.zc` contains `struct AnimSpeedScaler`. This is the file you want to copy into your mod.

* `demo.zc` provides three demonstration weapons (`SpeedScalingPistol`, `SpeedScalingShotgun`, and `SpeedScalingChaingun`) and one demonstration monster (`SpeedScalingImp`).

* `cvarinfo` defines CVars for controlling the animation speeds of the demo actors. `weapon_speed_scale` controls the animation speeds of the weapons, while `imp_speed_scale` controls the animation speed of the monster.

* `zscript.zc` just `#include`s the above two ZScript files.

## Using the Demo

The actual `AnimSpeedScaler` code is game-independent, but the demo only works in Doom 1/2/TNT/Plutonia. So:

1. Download this repository onto your computer.
2. Load the `AnimSpeedScaler` folder into GZDoom as a mod, with a suitable Doom IWAD.
3. `take Pistol;give SpeedScalingPistol;give SpeedScalingShotgun;give SpeedScalingChaingun`
4. Observe that your weapon slots 2 through 4 now contain slowed-down versions of the vanilla weapons.
5. Play with the `weapon_speed_scale` CVar, and observe its effect on the speeds of your weapons. A positive value makes them slower, and a negative value makes them faster.
6. `god;summon SpeedScalingImp`
7. Observe that the imp in front of you is slower than usual.
8. Play with the `imp_speed_scale` CVar, and observe its effect on the speed of the imp's actions.

## Using in Your Mod

1. Copy `AnimSpeedScaler.zc` into your mod. (Rename it if you wish.)
2. `#include` it from your main ZScript file.
3. In the actor class you need to control, add a field to it like this:

	```
	private AnimSpeedScaler speeds;
	```

4. Initialize it. In your `BeginPlay` or `PostBeginPlay` method, set the `Target` field to the actor being controlled. Example:

	```
	override void BeginPlay() {
		super.BeginPlay();
		speeds.Target = self;
	}
	```

5. Call its `Tick` method from the controlling actor's. Example:

	```
	override void Tick() {
		super.Tick();
		speeds.Tick();
	}
	```

Now, you can control the speed of the actor's animations through the `speeds.Scale` field. The value of this field is the number of extra tics to add for each tic of animation. Negative values skip tics of animation, while positive values delay them. Here are some examples:

Scale | Effect
------- | ------
0 | No change.
1 | 100% slower. Each tic of an actor's state actually lasts for 2 tics.
2 | 200% slower. Each tic of an actor's state actually lasts for 3 tics.
0.5 | 50% slower. Every other tic of animation actually lasts for 2 tics.
-1 | 100% faster. In every 2 tics of an actor's state, 1 tic is skipped.
-2 | 200% faster. In every 3 tics of an actor's state, 2 tics are skipped.
-0.5 | 50% faster. In every 3 tics of an actor's state, 1 tic is skipped.

### Interpolation Issue

Setting `Scale` to a value greater than 0 interacts poorly with engine-provided animation interpolation, such as interpolated monster movement and the weapon raise/lower animation, because the engine has no way of knowing that it should adjust interpolation speed. The greater the `Scale`, the more pronounced this effect is.

To avoid this effect, only use a zero or negative `Scale`, that is, only make animations faster, not slower.
