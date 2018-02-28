

# Quick-start

Note: this SDK was tested against Unity 5.6.1f1.

See Assets/Examples for examples.

To set up a new project:

1. Download and import the unityPackage to your project ([download link](https://github.com/tangibleplay/sdk/raw/master/OsmoSDKPackage.unitypackage))
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

The simulated pieces are stored in virtual drawers. You can open / close drawers by pressing 'S' (this is configurable on the TangibleManager). You can grab pieces and place them back inside drawers.


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

<br>

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

## Alive
```csharp
public bool Alive { get; }
```

`Alive` means that this piece is in the play area. This includes persisted pieces that were not in the last vision frame.

## Visible
```csharp
public bool Visible { get; }
```

`Visible` means that this piece is in the play area. This **does not** includes persisted pieces that were not in the last vision frame.

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
