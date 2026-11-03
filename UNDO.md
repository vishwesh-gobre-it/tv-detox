# Undoing the TV cleanup (TCL 65P635 Pro, 192.168.31.95)

Everything was turned off, never uninstalled, so all of it can be turned back on. The full list
of changes is in `my-tv.txt`. The original package lists and settings are in `backup-2026-11-03/`.

## Before any of these: turn debugging back on

1. On the TV: Settings > System > About, press OK on **Build** 7 times (only if Developer
   options is hidden), then turn on **USB debugging** and **Network debugging** in Developer options.
2. On the laptop, open Git Bash in this folder and run `adb connect 192.168.31.95:5555`.
   If the TV asks "Allow debugging?", tick "Always allow" and press Allow.

## Turn everything back on

```sh
# 1. Google TV's home screen and setup app back on, tv-detox removed
sh undo.sh 192.168.31.95:5555

# 2. Every app in my-tv.txt back on
grep '^off ' my-tv.txt | cut -d' ' -f2 | while read -r p; do
  adb -s 192.168.31.95:5555 shell pm enable "$p" </dev/null
done

# 3. Animations back on
adb -s 192.168.31.95:5555 shell settings put global window_animation_scale 1
adb -s 192.168.31.95:5555 shell settings put global transition_animation_scale 1
adb -s 192.168.31.95:5555 shell settings put global animator_duration_scale 1

# 4. Restart
adb -s 192.168.31.95:5555 reboot
```

To bring back one app only: `adb shell pm enable <package>`, then restart.

Stan (`au.com.stan.and`) and Rakuten TV (`tv.wuaki.apptv`) were already off before the cleanup
and aren't in `my-tv.txt`, so step 2 leaves them off.

## Only Google TV's home screen back (keep the cleanup)

Run `sh undo.sh 192.168.31.95:5555`. It turns Google TV's home screen
(`com.google.android.apps.tv.launcherx`) and its setup app
(`com.google.android.tungsten.setupwraith`) back on, removes tv-detox, and leaves every other
app as it is. Restart the TV afterwards. This also brings back the remote's Settings button.

## Change the top row from the remote

On the tv-detox home screen, hold **OK** on an app (or press **Menu**) to:

- **Move** it: slide it with the arrows, then press **OK**.
- **Replace** it with another app.
- **Remove** it from the home screen. The app stays installed.

To add an app, pick **Add app** in the bottom row. Changes are kept until `install.sh` is run
again with a new app list.

On the **Inputs** tile, press **OK** to pick an HDMI input. Hold **OK** to rename or hide one.
A hidden input comes back by holding **OK** on **Add app**.

## Known side effects (accepted)

- The remote's **TCL Channel** button stopped working after batch 1. To try getting it back:
  `pm enable com.tcl.waterfall.overseas`, then restart.
- The remote's **Settings** button and **app buttons** stopped working after the home screen
  swap. Settings are still reachable on screen (the **Settings** tile, then **TV settings**).
  For the app buttons, try `pm enable com.tcl.partnercustomizer`, then `com.tcl.ttvs`,
  restarting after each.
- TCL's firmware turns `com.tcl.notereminder` back on at every restart, so it was left on.
- AirPlay never worked on this TV. There is no AirPlay app on it.

## After an update

A TV update can turn apps back on. Turn debugging on, then run
`sh check.sh 192.168.31.95:5555 --fix my-tv.txt`, then turn debugging off again.
