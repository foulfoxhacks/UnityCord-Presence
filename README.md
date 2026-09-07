# UnityCord Presence

Editor-focused Discord Rich Presence for **Unity** and **VRChat Creator Companion (VCC)** projects.

**Created by [Sammy The Femboy Puppy](https://akasammythepuppy.me/)** · [Unity, VRChat & development portfolio](https://akasammythepuppy.me/work/)

UnityCord uses Discord's Game SDK to show the active Unity project, editor activity, detected VRChat SDK type, VPM package count, Unity version, and session duration in Discord.

## Features

- loads automatically with the Unity Editor
- detects VRChat avatar and world projects from `Packages/vpm-manifest.json`
- reports selected asset, script compilation, and asset-importing states
- cycles VCC SDK type, VPM package count, and Unity version
- reconnects when Discord is temporarily unavailable
- cleans up the native Discord SDK during assembly reload and editor shutdown
- still works as a normal Unity Editor presence when no VCC manifest is present

## Requirements

- Unity 2021.3 LTS or later
- 64-bit Windows Unity Editor
- Discord desktop
- Discord application with Rich Presence art assets
- included `discord_game_sdk.dll`

## Installation

Copy the repository files into these locations in the Unity project's `Assets` folder:

```text
Assets/
  Plugins/
    discord_game_sdk.dll
  Scripts/
    Control/
      Editor/
        NexiumBridge.cs
        VCCManifestReader.cs
    Internal/
      ActivityManager.cs
      Constants.cs
      Core.cs
      ImageManager.cs
      LobbyManager.cs
      StorageManager.cs
      StoreManager.cs
    Presence/
      DiscordAssets.cs
      DiscordPresence.cs
      DiscordTimestamps.cs
```

Keep `NexiumBridge.cs` and `VCCManifestReader.cs` under an `Editor` directory so editor-only APIs are not included in player builds.

## Discord application setup

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications).
2. Copy its Application ID.
3. Replace the `ClientID` value in `Scripts/Control/Editor/NexiumBridge.cs`.
4. Add Rich Presence art assets with the keys:
   - `vcc_logo`
   - `unity_icon`
5. Start Discord desktop.
6. Open the Unity project.

The Discord Application ID is public metadata. Do **not** put bot tokens, OAuth client secrets, or other credentials in the project.

## How it works

`NexiumBridge` loads through Unity's `InitializeOnLoad` behavior. It initializes Discord Game SDK, runs callbacks from the editor update loop, watches Unity editor state, and updates Rich Presence.

`VCCManifestReader` reads:

```text
Packages/vpm-manifest.json
```

when present to identify the VRChat SDK/project type and count VPM packages.

Projects without that file are treated as regular Unity Editor projects.

## Troubleshooting

### No presence appears

Confirm Discord desktop is running and the configured `ClientID` belongs to a valid Discord application.

### Rich Presence images do not appear

Confirm the Discord application includes assets named `vcc_logo` and `unity_icon`. Newly uploaded assets can take time to process.

### DLL load error

Confirm `discord_game_sdk.dll` is under `Assets/Plugins`, enabled for the editor, and compatible with your Unity Editor architecture.

### VRChat project detected as plain Unity

Confirm `Packages/vpm-manifest.json` exists and is readable.

### Repeated console warnings

UnityCord retries after failures. Check that Discord is running and that the native library can load.

## Development notes

This repository contains reusable source files rather than a complete Unity project. It does not ship a `.sln`, `.csproj`, sample scene, or standalone build configuration.

Test changes by importing the source into a Unity project and checking both the Unity Console and Discord presence.

## License

MIT License. Copyright (c) 2026 NDG Sammy The Foxxo.

## Creator and support

UnityCord Presence is created and maintained by **[Sammy The Femboy Puppy](https://akasammythepuppy.me/)** (`@foulfoxhacks`).

The broader Unity, avatar, VRChat, and development portfolio is documented at **[akasammythepuppy.me/work/](https://akasammythepuppy.me/work/)**.

For bugs and feature requests, [open an issue](https://github.com/foulfoxhacks/UnityCord-Presence/issues).
