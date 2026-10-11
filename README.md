# owldrop-indicator

AirDrop to and from nearby iPhones, iPads and Macs from a tray icon with a GTK
drop-down menu, in the style of nm-applet. It doesn't depend on any one
desktop or window manager: it runs in any tray that speaks StatusNotifierItem
(XFCE, MATE, Cinnamon, KDE, GNOME with the AppIndicator extension, or
waybar, polybar and the like on a bare WM).

Under the hood it drives the [omdrop](https://github.com/t4t5/omdrop-plugin)
command, which does the AirDrop protocol work. omdrop's ownis designed for only Omarchy. This one works (hopefully) everywhere else, as it implements a standard notification area applet. Tested and verified to be working in XFCE4 via the Status Tray Plugin for `xfce4-panel`.

Click the icon for the drop-down, which has everything in one place:

```
┌───────────────────────────────────────────────┐
│ AirDrop                                 [ ON] │
│ Visible as "test-system"                      │
│ 8:59 left                                     │
│ Visible for   [Once|10 min|30 min|1 hr|Always]│
│ Who can send            [Everyone | Contacts] │
├───────────────────────────────────────────────┤
│ Send to                                       │
│ James iPhone          -48 dBm  [Files…][Link…]│
│ Studio Mac            -61 dBm  [Files…][Link…]│
│ Look up device names                          │
├───────────────────────────────────────────────┤
│ [ Name… ]       [ Folder… ]     [Open folder] │
│                                  Quit owldrop │
└───────────────────────────────────────────────┘
```

It grows as devices are found, staying against the panel. Escape, or a click
anywhere else, closes it. Middle-click turns receiving on or off, on trays
that pass a middle click on (XFCE's does unless "Menu is primary action" is
set, when a middle click opens the drop-down too).

## Requirements

- **omdrop**, set up with `omdrop setup`, and a radio backend for it:
  [omdrop-owl](https://github.com/t4t5/omdrop-owl) for MediaTek and Intel
  cards, or omdrop-awdl for Apple Silicon Macs.
- A tray that speaks StatusNotifierItem: XFCE's Status Tray plugin, KDE,
  waybar's `tray` module, and most others do.
- A polkit agent, which asks for your password when the radio starts, and a
  notification daemon.

## Install

On Arch:

```sh
makepkg -si
```

This installs `owldrop-indicator`, its icons, a menu entry and an autostart
entry, so it starts with your next login. To start it now, run
`owldrop-indicator &`. To try it without installing, run `./owldrop-indicator`
from this directory.

Elsewhere, install `python-gobject`, GTK 3 and libnotify, then copy `owldrop-indicator` onto your PATH,
the icons into `~/.local/share/icons/hicolor/scalable/status/`, and the
`.desktop` file into `~/.config/autostart/`.

## How it works

The indicator keeps no AirDrop state. Like omdrop's own panel, it runs
the `omdrop` command for every action (`on`, `off`, `visibility`, `name`,
`dir`, `send --to`, `peers --json`) and draws what `omdrop status --json`
reports, every 2 s while receiving and every 10 s while off, so a change
made on the command line shows up in the menu.

- **The drop-down is a window, not a menu.** An open GTK menu never grows:
  rows added while it is open (devices, as they are found) leave it the
  height it opened with, scrolling. So the icon is a StatusNotifierItem of
  owldrop's own rather than an AppIndicator, and it publishes no menu: a tray
  shows an icon's menu instead of passing the click on, so with one, the
  drop-down could never open. On
  Wayland the drop-down can't be placed under the icon, so it opens at the
  pointer and closes when it loses focus.
- **Nearby devices** come from `omdrop peers --json`, which only reads the
  radio's peer table. **Look up device names** runs
  `omdrop peers --json -n --stream`. It is opt-in for the same reason as in
  omdrop: asking a name connects to each device and briefly announces your
  Apple ID over Bluetooth.
- **Received files** are announced by following the receiver's log
  (`journalctl --user -u airdrop-receiver.service`), the lines `omdrop watch`
  reads, with **Open** and **Show in folder** actions. omdrop's own
  notifications go through Omarchy's notifier, so they don't appear
  anywhere else.
- **Stay visible for** is remembered in `~/.config/owldrop-indicator/settings.ini`.
  Changing it while receiving re-arms the window with `omdrop on DURATION`.

The `omdrop` command is found through `$OWLDROP_CLI`, then `PATH`, then
`~/.local/bin/omdrop`.
