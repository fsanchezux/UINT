<img width="2196" height="806" alt="image" src="https://github.com/user-attachments/assets/957f9f1e-d671-4a68-83cd-060989ae9c06" />

# UIN'T

Nowadays we're pushed to vibe-code, which means paying for a subscription, and you don't always use up your token limit. That's where this project was born.
I'm developing a mod for Windows that adapts the UI/UX with a designer's approach, keeping things simple and adding a few quality-of-life features from Linux and macOS.

A [Windhawk](https://windhawk.net) mod for Windows 11: a Dynamic Island-style pill that reacts to music,
downloads, the clipboard, battery and more. When you maximize a window, it turns into a top bar.

It's a fork of [Dynamic Island for Windows](https://github.com/devcode90/Dynamic-Island-for-Windows) by
Himanshu (devcode90). Everything from the original still works. This fork adds the following.

## What this fork adds

- **Top bar with maximized windows.** The pill hides and a full-width bar appears
  (registered as an AppBar, so windows sit below it) with:
  - the 4 most recently used apps, and a button to go back to the previous one;
  - the time, which expands the island below the bar when clicked;
  - quick alerts (copied, Caps Lock, volume, low battery, notifications…);
  - shortcuts to Calendar, Weather, Notes and Pomodoro, plus network, Bluetooth, volume, battery and Control Center.
- **macOS-style Control Center.** Wi-Fi, Bluetooth, brightness (DDC/CI), volume and media in a single panel, with
  lists of networks and Bluetooth devices.
- **New cards:**
  - **Notes**, with a pinned note that stays visible.
  - **Files** dropped onto the island, in the same list as the notes.
  - **Pomodoro**.
  - **Widgets**, to turn cards on or off.
  - The **time** card cycles between clock, calendar and weather.
- **Focus on hover** (optional) and a **custom color palette**.
- Spanish interface.

## Installation

1. Install [Windhawk](https://windhawk.net).
2. In Windhawk: **Explore → Create a new mod**.
3. Delete the template, paste the contents of [`uint.wh.cpp`](uint.wh.cpp) and click **Compile mod**.
4. Optional: in the mod's **Settings** tab, open the text view and paste [`settings/uint.yaml`](settings/uint.yaml).

If you have the original mod (*Dynamic Island for Windows*) installed, disable it so you don't end up with two islands.

### Font

UIN'T uses [Inter](https://rsms.me/inter/) and installs it automatically: if you don't have it, the first time it starts it downloads
[`fonts/InterVariable.ttf`](fonts/InterVariable.ttf) and installs it for your user only, without administrator
permissions. In the meantime, or when offline, Segoe UI is used. If you set a different font in the *Font family* setting,
nothing is downloaded.

Inter is by The Inter Project Authors and is distributed under the SIL Open Font License
([`fonts/LICENSE-Inter.txt`](fonts/LICENSE-Inter.txt)).

### Making notifications work

Turn on **Settings → Privacy & security → Notifications → "Allow apps to access your notifications"**.

### Taskbar settings (optional)

[`settings/`](settings) also contains the settings I use to get the taskbar looking like a centered dock.
They are for these two mods, which you can install from the Windhawk catalog:

- `windows-11-taskbar-styler.yaml` → *Windows 11 Taskbar Styler*
- `taskbar-height-and-icon-size.yaml` → *Taskbar height and icon size*

## Development

The whole mod lives in a single file, [`uint.wh.cpp`](uint.wh.cpp), written in C++ with Direct2D and C++/WinRT. To test
a change, paste it into the Windhawk editor and compile it. To check only the syntax, you can use the clang that
ships with Windhawk (the command is in [`CLAUDE.md`](CLAUDE.md)).

Issues and pull requests are welcome.

## Credits and license

- Original mod: [Dynamic Island for Windows](https://github.com/devcode90/Dynamic-Island-for-Windows), by
  Himanshu (devcode90) and contributors (see the credits inside the mod).
- Fork: [fsanchezux](https://github.com/fsanchezux).

Licensed under [MIT](LICENSE).
