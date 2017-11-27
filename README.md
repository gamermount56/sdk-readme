# Quick-start

See this nicely formatted at https://github.com/tangibleplay/sdk-readme.

See Assets/Examples for examples.

To set up a new project:

1. Include the OsmoSDK plugin to your project's Plugins/iOS folder
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
- [TangibleManager.OnObjectEnter](#onobjectenter)
- [TangibleManager.OnObjectExit](#onobjectexit)
- [TangibleManager.AliveObjects](#aliveobjects)

`TangibleManager` is the main interface to the OsmoSDK. You can access all the `TangibleObject`s here which represent the physical pieces.

The class implements multiple ways of generating the input, primarily:
  - If the game is running in the Unity Editor, it will use the `OnScreenController` which simulates pieces.
  - Otherwise, it will use the `PhysicalController` which relies on the `VisionFramework` and the camera input.

<br>

## OnObjectEnter
```csharp
public event Action<TangibleObject> OnObjectEnter;
```

Invoked whenever a `TangibleObject` becomes `Alive`. This is the case when the piece (whether simulated or not) is recognized from vision.

## OnObjectExit
```csharp
public event Action<TangibleObject> OnObjectExit;
```

Invoked whenever a `TangibleObject` becomes not `Alive`. This happens when the piece is no longer recognized by vision and after a certain amount of time / frames have passed.

The `TangibleObject` persists because the piece can disappear from vision for a couple frames due to any number of factors including obstruction from hands, etc.

## AliveObjects
```csharp
public IEnumerable<TangibleObject> AliveObjects { get; }
```

All `TangibleObject`s that are `Alive`. This is useful if you are starting up some level and want to know if vision already has some recognized pieces.


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

## SetState
```csharp
public void SetState(VisionSetupState state);
```

For performance reasons, you can set the state to be InActive, Passive, or Active. InActive means that vision setup is not checking for new data. Passive means that vision setup will check for new data every 100 vision frames. Active means that vision setup will check for new data every vision frame.

```csharp
public enum VisionSetupState {
	Inactive,
	Passive,
	Active,
}
```

## GetSetupFlags
```csharp
public ReadOnlyCollection<SetupFlag> GetSetupFlags();
```

Get all the flags that vision is reporting.

```csharp
// is an integer bit mask with these bit values
public enum SetupFlag {
	Run = 0,                  // 1
	Perfect = 1,              // 2
	BadOrientation = 2,       // 4
	MirrorUncentered = 4,     // 16
	MirrorMoveLeft = 5,       // 32
	MirrorMoveRight = 6,      // 64
	NoMirrorDetected = 7,     // 128
	PushMirrorDown = 8,       // 256
	NoMotionData = 12,        // 4096
	OsmoBoardDetected = 16,   // 65536
	Error = 31,
}
```
