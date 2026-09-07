# UnityCord Presence - A Discord Rich Presence for Unity

A small, editor-focused Discord Rich Presence integration for Unity and VRChat Creator Companion (VCC) projects. It uses Discord's Game SDK to show the current Unity project, editor activity, SDK type, package count, and session duration in Discord.

## Features

- Runs automatically when the Unity Editor loads
- Detects VRChat avatar and world projects from `Packages/vpm-manifest.json`
- Shows the selected asset, script compilation, or asset importing state
- Cycles through the VCC SDK type, VPM package count, and Unity version
- Reconnects when Discord is temporarily unavailable
- Cleans up the native Discord SDK during assembly reload and editor shutdown

## Requirements

- Unity 2021.3 LTS or later
- A 64-bit Windows Unity Editor
- The Discord desktop client
- A Discord application with Rich Presence art assets
- The included `discord_game_sdk.dll` native library

## Installation

Copy the repository files into the following locations in your Unity project's `Assets` directory:

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

Keeping `NexiumBridge.cs` and `VCCManifestReader.cs` in an `Editor` folder prevents editor-only APIs from being included in player builds.

## Discord application setup

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications).
2. Copy its Application ID.
3. Replace the `ClientID` value in `Scripts/Control/Editor/NexiumBridge.cs`.
4. Add Rich Presence art assets with these keys:
   - `vcc_logo`
   - `unity_icon`
5. Open the project in Unity while the Discord desktop client is running.

The application ID is public metadata, not a secret. Do not add bot tokens, OAuth client secrets, or other credentials to the source code.

## How it works

`NexiumBridge` is loaded automatically by Unity's `InitializeOnLoad` attribute. It initializes the Discord Game SDK, runs callbacks from the editor update loop, and updates the activity as the editor state changes.

`VCCManifestReader` reads `Packages/vpm-manifest.json` when present to identify the VRChat SDK type and count VPM packages. Projects without that file are shown as regular Unity Editor projects.

The `Scripts/Internal` files are the C# Discord Game SDK bindings used by the bridge. The small classes in `Scripts/Presence` are serializable presence data models for consumers that want to build additional tooling.

## Troubleshooting

- **No presence appears:** Confirm Discord is running and the `ClientID` belongs to an existing Discord application.
- **Images do not appear:** Confirm the application has assets named `vcc_logo` and `unity_icon`. Discord may take time to process newly uploaded assets.
- **DLL load error:** Confirm the DLL is under `Assets/Plugins`, enabled for the Unity Editor, and compatible with the editor architecture.
- **Presence reports Unity Editor instead of a VRChat project:** Confirm `Packages/vpm-manifest.json` exists and is readable.
- **Console warnings repeat:** The integration retries after failures. Check that Discord is running and that the native SDK library can load.

## Development

This repository contains reusable source files rather than a complete Unity project, so it does not include a `.sln`, `.csproj`, sample scene, or standalone build configuration. Test changes by importing the files into a Unity project and checking the editor console and Discord presence.

## License

MIT License. Copyright (c) 2026 NDG Sammy The Foxxo.

## Creator

Built by [Sammy The Femboy Puppy](https://akasammythepuppy.me/) ([@foulfoxhacks](https://github.com/foulfoxhacks)).

Explore more Unity editor tooling, VRChat work, and development projects in [Sammy's creator portfolio](https://akasammythepuppy.me/work/).

For UnityCord bugs and feature requests, [open an issue](https://github.com/foulfoxhacks/UnityCord-Presence/issues) or contribute a pull request in this repository.
