# Quick-start Guide

## What you'll need
+ Unity (tested against Unity `5.6.1f1`, `5.6.4f1`, and `2071.2f1`)
+ XCode (tested against 8.x and 9.x)
+ An iPad (any iPad [Osmo supports](https://support.playosmo.com/hc/en-us/articles/115009542208-Which-iPad-models-is-Osmo-compatible-with-))
+ Osmo Hardware (base+mirror and any tiles you plan to use)
+ The Osmo SDK

To set up a new project:

1. Clone the git repo and its subrepos from: [sdk-public](https://github.com/tangibleplay/sdk-public)
	+ Run `git submodule init` and `git submodule update` to clone subrepos
2. 

2. Add the `TangibleManager` script to the scene
  - choose a deck (collection of pieces, like Words, Numbers, Coding) by filling out the `Deck_` outlet
3. Now you can access the TangibleManager.Instance and use any of the public API to receive information about the physical pieces (TangibleObject).

# Available Decks

The SDK supports several Osmo standard decks of physical pieces:
  - Words - upper and lower case letters, red and blue, shipped with the Words game.
  - Numbers - digits and dice faces, shipped with the Numbers game.
  - Coding - the various tiles shipped with Osmo Coding Awbie.
  - Dominocodes - printable barcodes, convenient to prototype new games [(printable link)](./dominocodes.pdf).

# Testing In-Editor
To play with the pieces in the editor, `TangibleManager` uses the `OnScreenController` to simulate pieces. These pieces are rendered on a separate layer named `TangibleLayer` which is automatically added to the layers of the project.

The simulated pieces are stored in virtual drawers. You can open / close drawers by pressing `X` (this is configurable on the `TangibleManager`). You can grab pieces and place them back inside drawers.

To rotate pieces, press and hold your mouse button over a tile and hold `A` or `D` to rotate.

For coding pieces, pressing `W` `A` `S` `D` will set the direction arrow on pieces. Pressing `1` `2` `3` `4` `5` will set the quantifier attached. Press `SPACE` to send a 'play' button press.

# Examples

We provide a set of example scenes which demonstrate various ways to use the SDK. These range from simply dumping out raw data to a small puzzle game using Coding tiles. Explore, mangle, and poke at these to your heart's content.

## Basic Example
`Examples/BasicExample/BasicExampleScene.unity`

A simple example of dumping out values seen by the computer vision to screen. Swap out the Deck of the `TangibleManager` on the scene to use different types of tiles.

In our `Update()` call of `Tangible.SDKExamples.BasicExample.Game` you will see we use the current `AliveObjects` list pulled from the singleton instance of `TangibleManager`.

![basic_example](Images/Examples/basic.png)


## Basic Setup Example
`Examples/BasicNumbers/BasicNumbersScene.unity`

An example of how to use the setup configuration values in order to instruct the user on how to properly set up the Osmo hardware.

If you are testing this in the editor, look at the `VisionSetup` object in the scene. You can modify which flags are sent (see the [`VisionSetup`](#visionsetup) section for more).

![example_basic_setup](Images/Examples/setup.png)


## Basic Numbers
`Examples/BasicSetupExample/BasicSetupScene.unity`

In this example, we subscribe to the events `OnObjectEnter` and `OnObjectExit` from the `TangibleManager`. In the case of subtraction, we also register for the event `OnLocationChanged` because we need to order by X position when tiles move (even if no new tiles enter or exit the play area)

![example_basic_numbers](Images/Examples/numbers.png)

## Basic Words
`Examples/BasicWords/BasicWordsScene.unity`

An example of a game which only registers with `OnObjectEnter`. Once a letter has been detected, the game treats it as guessed. `OnObjectExit` is ignored because letters cannot be unguessed.

![example_basic_words](Images/Examples/words.png)

## Coding Painting
`Examples/CodingPainting/CodingPaintingScene.unity`

See the [Testing In-Editor](#testing-in-editor) section for how to play in-editor with Coding tiles.

The game of life, using coding tiles to seed different species on the board.

This game uses Coding tiles, and the `ClusterManager`. It uses the most simple decision of what cluster to use by always choosing the largest one.

The `ClusterManager` subscribes to the `OnUpdatedTangibleObjects` event which fires an event every time the computer vision returns with a new set of data. The rate at which this happens is not the same as your Unity update rate. It’s generally 10-20 times per second on a newer iPad.

See `CodingClusterService` for how the game decides what is in a cluster and what is not.

![example_coding_painting](Images/Examples/coding_painting.png)

## Coding Valley
`Examples/CodingValley/CodingValleyScene.unity`

See the [Testing In-Editor](#testing-in-editor) section for how to play in-editor with Coding tiles.

An example of a more complete small puzzle game using the Coding tiles.


![example_coding_valley](Images/Examples/coding_valley.png)

## Timeline Game
`Examples/TimelineGame/TimelineScene.unity`

An example game using domino codes. It uses a custom deck (which ignores domino code values that are not used by the game). To play this on device, print out a set of domino codes from `dominocodes.pdf` in the `sdk-readme` directory of the SDK.

In order to make your own custom decks, see the [Deck](#deck) section.

![example_timeline_game](Images/Examples/timeline_game.png)

# API

# TangibleManager
- [TangibleManager.AliveObjects](#aliveobjects)
- [TangibleManager.OnObjectEnter](#onobjectenter)
- [TangibleManager.OnObjectExit](#onobjectexit)
- [TangibleManager.OnUpdatedTangibleObjects](#onupdatedtangibleobjects)
- [TangibleManager.Mute](#mute)

`TangibleManager` is the main interface to the OsmoSDK. You can access all the `TangibleObject`s here which represent the physical pieces.

The class implements multiple ways of generating the input, primarily:
  - If the game is running in the Unity Editor, it will use the `OnScreenController` which simulates pieces.
  - Otherwise, it will use the `PhysicalController` which relies on the `VisionFramework` and the camera input.

![tangible_manager_inspector](Images/TangibleManager/inspector.png)

## AliveObjects
```csharp
public IEnumerable<TangibleObject> AliveObjects { get; }
```

All `TangibleObject`s that are `Alive`. This is useful if you are starting up some level and want to know if vision already has some recognized pieces.


## OnObjectEnter
```csharp
public event Action<TangibleObject> OnObjectEnter;
```

Invoked whenever a `TangibleObject` becomes `Alive`. This is the case when the piece (whether simulated or not) is recognized from vision.

## OnObjectExit
```csharp
public event Action<TangibleObject> OnObjectExit;
```

Invoked whenever a `TangibleObject` becomes not `Alive`. This happens when the piece is no longer recognized by vision and after a certain amount of time / frames have passed. You can modify how long this takes by changing the values on your `TangibleManager` MonoBehaviour on your scene.

The `TangibleObject` persists because the piece can disappear from vision for a couple frames due to any number of factors including obstruction from hands, etc.

## OnUpdatedTangibleObjects
```csharp
public event Action OnUpdatedTangibleObjects;
```

Invoked whenever the `VisionFramework` has processed and returned new data. The `VisionFramework` does NOT run in lockstep with Unity, it is completely asynchronous.

It will generally run at approximately 10-20 frames per second, but on older devices it may be even slower (especially if you are doing a lot of CPU processing in Unity).


## Mute
```csharp
public void Mute(bool mute);
```

Muting the `VisionFramework` will disable all computer vision processing until you unmute. This is useful if you want to save CPU resources when you don't need any data from the `VisionFramework` (such as if you're on a home screen or a settings screen).


# TangibleObject
- [TangibleObject.Id](#id)
- [TangibleObject.UniqueId](#uniqueid)
- [TangibleObject.Alive](#alive)
- [TangibleObject.Visible](#visible)
- [TangibleObject.Location](#location)

`TangibleObject` is the C# class that represents a physical piece in the Deck.

## Id
```csharp
public int Id;
```

`Id` is mapped to a piece in the Deck (this is non-unique since there can be multiples of the same piece - i.e. walking block in Coding)

## UniqueId
```csharp
public readonly int UniqueId;
```

Given that there can be multiple of the same pieces per deck, `TangibleObject`s have a `UniqueId` which is used to map the vision output to the last closest unique physical piece.

This is a best effort calculation. You will never see two objects with the same `UniqueId`, but you may see see two `UniqueId`s where you would have thought there should have been one (generally worse on older iPads with worse cameras where we cannot keep track of tiles as well).

## Alive
```csharp
public bool Alive { get; }
```

`Alive` means that this piece is in the play area. This includes persisted pieces that were not in the last vision frame.

## Visible
```csharp
public bool Visible { get; }
```

`Visible` means that this piece is in the play area. This **does not** includes persisted pieces that were not in the last vision frame. `Visible` values will flicker on/off as the vision algorithms work with imperfect camera input.

## Location
```csharp
public Location Location { get; }
```

`Location` contains all the data of the piece's current position and orientation.

```csharp
public class Location {
  public float X { get; }
  public float Y { get; }

  // angle in degrees
  public float Orientation { get; }
}
```

# TangibleDebugOverlay
- [TangibleDebugOverlay.Show](#show)
- [TangibleDebugOverlay.Hide](#hide)

To debug what the vision is seeing on device, we have provided a class called `TangibleDebugOverlay` which has two methods `Show()` and `Hide()`. You can show the debug overlay at anytime to display ghost tiles on the screen matching the `TangibleObject`s active in `TangibleManager`.

You can toggle the TangibleDebugOverlay through the editor panel that appears when you press 'S'. Use the "Debug Overlay" checkmark at the bottom. You'll see the raw id values overlayed on top of the tiles you are manipulating.

![debug_overlay_hidden](Images/TangibleDebugOverlay/hidden.png)
![debug_overlay_shown](Images/TangibleDebugOverlay/shown.png)

## Show
```csharp
public static void Show();
```

`Show` starts up DebugOverlay objects which will render ghost pieces on the screen that correspond to `TangibleObject`s in `TangibleManager`.

## Hide
```csharp
public static void Hide();
```

Hides the DebugOverlay objects.

# VisionSetup
- [VisionSetup.SetState](#setstate)
- [VisionSetup.GetSetupFlags](#getsetupflags)

Using computer vision, we can detect if the device is setup properly and prompt for certain actions if needed. To do this, we use the `VisionSetup` script. You can access it, if it exists in the scene, by calling `VisionSetup.Instance`.

We have provided basic UI for dealing with setup flags that require user action. To use it, add the `BuiltInSetupUI` script to your scene.

![vision_setup_fullscreen](Images/VisionSetup/fullscreen.png)
![vision_setup_passive](Images/VisionSetup/passive.png)

To customize the visuals, look at these prefabs (Modify if you are using fullscreen setup or not in the `SetupUI` child of `VisionSetup`)
+ `BuiltInSetupFlagConstant` - Points to images used by the prefabs below
+ `BuiltInSetupUIDropdown` - Prefab used for the non-blocking setup dropdown
+ `BuiltInSetupUIFullScreen` - Prefab used for the blocking full screen setup

## SetState
```csharp
public void SetState(VisionSetupState state);
```

For performance reasons, you can set the state to be Inactive or Active. Inactive means that vision setup is not checking for new data. Active means that vision setup will check for new data every vision frame.

```csharp
public enum VisionSetupState {
	Inactive,
	Active,
}
```

## GetSetupFlags
```csharp
public ReadOnlyCollection<SetupFlag> GetSetupFlags();
```

Get all the flags that vision is reporting. You can parse this in order to display to the user how you want them to correct the incorrect base/mirror setup.

```csharp
public enum SetupFlag {       // an integer bit mask with these bit values
	Run = 0,                  // 1
	Perfect = 1,              // 2
	BadOrientation = 2,       // 4
	MirrorUncentered = 4,     // 16
	MirrorMoveLeft = 5,       // 32
	MirrorMoveRight = 6,      // 64
	NoMirrorDetected = 7,     // 128
	PushMirrorDown = 8,       // 256
	NoMotionData = 12,        // 4,096
	OsmoBoardDetected = 16,   // 65,536
	Error = 31,               // 2,147,483,648
}
```

If you are testing in editor, you can control the flags sent on your `VisionSetup` object.

![vision_setup_unity_flags](Images/VisionSetup/unity_flags.png)


# Legal
The latest [SDK License Agreement](https://docs.google.com/document/d/1YK82HsDxKN9U_w3t507ON6N_rN6XuUH8af9n4wB2z5A/edit#)

For any other questions, contact sdk@playosmo.com
