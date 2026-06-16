# A‑Simple‑DiscordToUnity‑Rich‑Presence
### Lightweight Discord Rich Presence integration for Unity

[![Release](https://img.shields.io/badge/release-v1.0.0-blue?style=for-the-badge)](https://github.com/foulfoxhacks/A-Simple-DiscordToUnity-Rich-Presence)
[![Unity](https://img.shields.io/badge/Unity-2021.3%20LTS-2b2b2b?style=for-the-badge)](https://unity.com/)
[![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)](LICENSE)

A minimal, easy‑to‑integrate Discord Rich Presence bridge for Unity projects. Designed for quick setup and small games that want to show player status in Discord without heavy SDK overhead.

---

## Table of Contents

- [Overview](#overview)  
- [Key Features](#key-features)  
- [Requirements](#requirements)  
- [Quick Start](#quick-start)  
- [Discord App Setup](#discord-app-setup)  
- [Unity Integration Guide](#unity-integration-guide)  
  - [Importing the Package](#importing-the-package)  
  - [Example C# Script](#example-c-script)  
  - [Updating Presence](#updating-presence)  
- [Configuration](#configuration)  
- [Example Usage](#example-usage)  
- [Troubleshooting](#troubleshooting)  
- [Development & Contributing](#development--contributing)  
- [Changelog](#changelog)  
- [License](#license)  
- [Contact](#contact)

---

## Overview

This project provides a simple, cross‑platform approach to display Discord Rich Presence from Unity. It uses a lightweight IPC wrapper (or Discord GameSDK if you prefer) to send presence updates such as **state**, **details**, **timestamps**, **assets**, and **buttons**.

Designed for:
- Small to medium Unity projects  
- Rapid prototyping and indie games  
- Developers who want a minimal dependency surface

---

## Key Features

- Easy Unity package import  
- Simple C# API for updating presence  
- Support for timestamps, assets, and interactive buttons  
- Works in Editor and standalone builds (platform caveats apply)  
- Example scenes and scripts included

---

## Requirements

- **Unity:** 2021.3 LTS (recommended) or later  
- **Scripting Runtime:** .NET Standard 2.0 / C# 8 compatible  
- **Discord:** Desktop client installed for IPC; or use Discord GameSDK for builds  
- **Platforms:** Windows, macOS, Linux (IPC availability varies by platform)

---

## Quick Start

1. **Clone the repo**
   ~~~bash
   git clone https://github.com/yourname/A-Simple-DiscordToUnity-Rich-Presence.git
   ~~~

2. **Open in Unity**  
   Launch Unity Hub and open the `A-Simple-DiscordToUnity-Rich-Presence` project folder.

3. **Import Plugins**  
   If using the included `Plugins/discord_game_sdk.dll`, ensure it’s placed under `Assets/Plugins/` for the correct platform.

4. **Open Example Scene**  
   `Assets/Scenes/Example.unity` demonstrates initialization and presence updates.

---

## Discord App Setup

1. Go to the **Discord Developer Portal** and create a new application.  
2. Under **OAuth2 → General**, copy the **Application ID** (this is your `CLIENT_ID`).  
3. Under **Rich Presence → Art Assets**, upload images for `large_image` and `small_image` (names used in the Unity script).  
4. No OAuth redirect is required for local IPC‑based Rich Presence; for GameSDK or web flows, configure redirect URIs as needed.

**Example config (JSON):**
~~~json
{
  "client_id": "123456789012345678",
  "assets": {
    "large_image": "game_large",
    "small_image": "game_small"
  }
}
~~~

---

## Unity Integration Guide

### Importing the Package
- Copy `Assets/Plugins/` and `Assets/Scripts/` from this repo into your Unity project.  
- Ensure any native DLLs are placed in `Assets/Plugins/x86_64` (or platform‑specific folders).

### Example C# Script

Create `Assets/Scripts/DiscordRichPresence.cs` and paste:

~~~csharp
using System;
using UnityEngine;
using System.Runtime.InteropServices;

public class DiscordRichPresence : MonoBehaviour
{
    // Example: lightweight IPC wrapper (replace with your wrapper or GameSDK)
    [DllImport("discord_ipc")]
    private static extern bool Discord_Initialize(string clientId);

    [DllImport("discord_ipc")]
    private static extern bool Discord_UpdatePresence(string jsonPayload);

    [DllImport("discord_ipc")]
    private static extern void Discord_Shutdown();

    public string clientId = "YOUR_CLIENT_ID";
    public string details = "In Menu";
    public string state = "Idle";
    public string largeImage = "game_large";
    public string smallImage = "game_small";

    void Start()
    {
        bool ok = Discord_Initialize(clientId);
        if (!ok) Debug.LogWarning("Discord IPC init failed. Ensure Discord is running.");
        UpdatePresence(details, state);
    }

    public void UpdatePresence(string details, string state, long startTimestamp = 0)
    {
        var payload = new
        {
            state = state,
            details = details,
            timestamps = startTimestamp > 0 ? new { start = startTimestamp } : null,
            assets = new { large_image = largeImage, small_image = smallImage }
        };
        string json = JsonUtility.ToJson(payload);
        Discord_UpdatePresence(json);
    }

    void OnApplicationQuit()
    {
        Discord_Shutdown();
    }
}
~~~

> **Note:** The above uses a placeholder native plugin `discord_ipc`. Replace with your actual IPC wrapper or the Discord GameSDK bindings.

### Updating Presence
Call `UpdatePresence(...)` from your game logic when player state changes (e.g., entering a match, pausing, or finishing a level).

Use timestamps to show elapsed time:

~~~csharp
long unixNow = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
UpdatePresence("Playing Level 1", "In Match", unixNow);
~~~

---

## Configuration

- **Client ID:** Set in the script or via a JSON config file.  
- **Assets:** Upload images in the Discord Developer Portal and reference their keys in `assets.large_image` and `assets.small_image`.  
- **Environment:** For builds, ensure the native plugin is included for the target platform.

**Sample `discord_config.json`:**
~~~json
{
  "client_id": "123456789012345678",
  "large_image": "game_large",
  "small_image": "game_small"
}
~~~

Load this file at runtime if you prefer not to hardcode the client ID.

---

## Example Usage

**Initialize in a manager script:**

~~~csharp
public class GameManager : MonoBehaviour
{
    public DiscordRichPresence discord;

    void Start()
    {
        discord.UpdatePresence("Starting Game", "Main Menu");
    }

    public void OnLevelStart(string levelName)
    {
        long start = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
        discord.UpdatePresence($"Playing {levelName}", "In Game", start);
    }

    public void OnLevelEnd()
    {
        discord.UpdatePresence("In Menu", "Idle");
    }
}
~~~

**Clear presence:**

~~~csharp
// If your wrapper supports clearing:
Discord_UpdatePresence("{}"); // or call a dedicated clear function
~~~

---

## Troubleshooting

- **Discord not detected:** Ensure the Discord desktop client is running for IPC. GameSDK requires proper initialization and a valid client ID.  
- **Editor vs Build:** IPC may behave differently in the Unity Editor. Test in a standalone build for accurate behavior.  
- **Permissions:** On some Linux setups, socket permissions can block IPC. Run Discord and your game under the same user.  
- **Assets not showing:** Confirm asset keys match names uploaded in the Discord Developer Portal.  
- **DLL load errors:** Place native libraries in the correct `Assets/Plugins/<platform>` folder and set platform settings in the Inspector.

---

## Development & Contributing

- **Code style:** C# 8, .NET Standard 2.0 compatibility.  
- **Testing:** Open the example scene and run in Editor; build for target platform to test native IPC.  
- **Contributing:** Fork → branch → PR. Use clear commit messages and include tests or example usage for new features.  
- **Issue template:** Provide Unity version, platform, and steps to reproduce.

---

## Changelog

- **v1.0.0** — Initial release: basic IPC wrapper example, Unity scripts, example scene, and documentation.

---

## License

~~~text
MIT License

Copyright (c) 2026 [NDG] Sammy The Foxxo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
~~~
---

## Contact

Created by **FoulFoxHacks** — open issues or PRs on GitHub: https://github.com/foulfoxhacks/A-Simple-DiscordToUnity-Rich-Presence.
