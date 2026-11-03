# TV Detox prompt: TCL 65P635 Pro

Open Claude Code in this folder (`tv-fix\tv-detox`) on the laptop that is on the same Wi-Fi as the
TV, then paste everything inside the box below.

Before you paste, on the TV: Settings > System > About > press OK on **Build** 7 times, then go
back to Developer options and turn on **USB debugging** (and **Network debugging** or **ADB over
network** if it is there). Keep the TV on for the whole session.

```text
Help me clean up my Android TV and make it faster, using the tv-detox repo in this
folder. Use this local copy: don't download the repo again. It has three changes
of mine: an "All apps" tile, one "Inputs" tile that lists the HDMI inputs instead
of a tile per input, and apps.txt set to my top row.

My TV is a TCL 65P635 Pro (Google TV on Android TV 11). Its IP is 192.168.31.95.
I'm on Windows. Run the scripts from Git Bash (sh build.sh, sh install.sh ...).

1. Before you change anything, tell me in plain words:
   this turns off system apps, and that has some risk.
   A few TVs got stuck at the logo or kept restarting,
   and you're an AI, so you can make mistakes. undo.sh
   and your undo list turn everything back on. Ask me to
   confirm I want to go ahead.

2. Set up the laptop. Check for adb, Java 17 and the Android SDK
   (platforms;android-34 and build-tools;34.0.0). Install what's missing,
   tell me what you're installing first, and set ANDROID_HOME (and JAVA_HOME
   if build.sh can't find Java). Then run `sh build.sh`. It must end with
   "Built out/tvhome.apk". If it doesn't compile, fix the error in my
   changes (src/com/ansub/tvhome/MainActivity.java: showAllApps,
   showInputs, showInputsMenu and the bottom row in buildTiles) without
   changing anything else, and tell me what you fixed. Keep key.jks:
   updates must be signed with the same key. install.sh uses out/tvhome.apk
   when it's there, so it installs this build, not the GitHub release.

3. Connect to the TV with `adb connect 192.168.31.95:5555`. Tell me when
   the "Allow debugging?" prompt will appear so I can tick "Always allow".
   Use port 5555, not Wireless debugging: turning off Google TV's setup app
   cuts a Wireless debugging connection. Check the TV reports Google TV,
   Android 11 and the device name (getprop ro.product.device; the P635
   range is "G07"). If it's different from that, tell me before going on.

4. Back up first, before any change: save the full package list, the
   disabled-package list, the user-installed list (pm list packages -3),
   getprop, and `settings list global/secure/system` to a backup folder
   here. Note the current keyboard (settings get secure default_input_method),
   the screensaver settings (screensaver_enabled, screensaver_components,
   screensaver_activate_on_sleep) and free memory.

5. Run `sh supports.sh 192.168.31.95:5555` and `sh candidates.sh
   192.168.31.95:5555`, then list every app in plain English: safe to turn
   off, needed by the TV, and apps I use. Read docs/debloat.md for the rules.
   For a list tested on a TV from the same range, see
   ../cleanyourtv/examples/tcl-75p635/debloat.txt. Treat it as a reference
   only, never as a list to apply, and follow my keep list below where the
   two disagree.

6. Ask me before every change. Turn apps off with
   `pm disable-user --user 0`; never uninstall. Run one pm command per shell
   call. Work in small batches. After each batch, wait 15 seconds, restart,
   check the TV starts normally, and ask me to try the remote's TV/CH, LIST,
   INFO, Input, Settings and app buttons. (A TCL 55P635 owner lost several
   of these and never found which app caused it.) Keep a list of what you
   changed and how to undo it, in check.sh's format (`off <package>` per
   line), saved here as my-tv.txt.

7. Never turn off:
   - Google Play services (com.google.android.gms), Google Play, Settings,
     System UI, the keyboards (com.tcl.inputmethod.international and Gboard),
     Bluetooth, or anything else the TV needs to start. If you're not sure,
     leave it on.
   - TCL's TV apps: com.tcl.tv (Live TV; it's also the only screen that
     shows HDMI), com.tcl.tvinput (HDMI and tuner), com.tcl.suspension
     (input picker), com.tcl.useragreement (privacy switches),
     com.tcl.providers.config, and any com.mediatek.* TV or remote service.
     The 75P635 list turns off com.tcl.suspension and com.tcl.useragreement.
     Don't follow it on those two.
   - My casting. I use Samsung DeX and Windows' "Connect to a wireless
     display" (both Miracast / screen mirroring) and AirPlay from Apple
     devices. Before the cleanup, find which packages provide Miracast and
     AirPlay on this TV (look for miracast, wfd, mirror, multiscreen,
     airplay, airserver, realtek, and check with `dumpsys package` what
     each one runs). Show me the list and keep all of them on, including
     com.tcl.MultiScreenInteraction_TV if it's there. If you can't find an
     AirPlay app at all, tell me. I don't use Chromecast
     (com.google.android.apps.mediashell): ask me before turning it off.
   - Every app I installed myself (everything in `pm list packages -3`).
     That includes Hik-Connect, a web browser and File Manager+. Also keep
     the system WebView (com.google.android.webview or com.android.webview),
     which the browser and Hik-Connect may need.
   - The screensaver: com.google.android.apps.tv.dreamx and
     com.android.dreams.basic. I want the screensaver to keep working.

8. Install the home screen with the apps I want on the top row, in this
   order: JioHotstar, YouTube, Hik-Connect, Netflix. Find their real package
   names on the TV first (apps.txt has my guesses: in.startv.hotstar or
   in.startv.hotstaronly, com.google.android.youtube.tv, com.connect.enduser
   or com.hikvision.hikconnect, com.netflix.ninja) and show them to me.
   Then do the full replacement test below.

   Full replacement test (I've agreed to try this, and it's reversible):
   a. Check who saved the Wi-Fi: `adb shell "dumpsys wifi | grep -o
      'cname=[^ ]*'"`. If it's com.google.android.tungsten.setupwraith,
      stop and ask me to forget the Wi-Fi network on the TV and join it
      again. Turning the setup app off would erase that network, and the
      TV would drop off Wi-Fi.
   b. Tell me exactly what this turns off (Google TV's home screen,
      com.google.android.apps.tv.launcherx, and its setup app,
      com.google.android.tungsten.setupwraith), and confirm with me. Then run:
      `GOOGLE_TV=1 YES=1 YES_SETUP=1 sh install.sh 192.168.31.95:5555 "<the four packages, comma separated>"`
   c. Wait 30 seconds, restart, and check: Home opens tv-detox and stays
      there after the restart; the top row is my four apps; the "All apps"
      tile (bottom row) lists every app, including Hik-Connect, the browser
      and File Manager+, and opens them; the "Inputs" tile lists every HDMI
      input and switches to the one I pick, and holding OK on it lets me
      rename an input; HDMI works.
   d. Ask me to test, one at a time: DeX mirroring from my Samsung phone,
      Windows' wireless display (Win+K), AirPlay from an Apple device, the
      screensaver (start it with `adb shell am start -n
      com.android.systemui/.Somnambulator` and ask me if it shows), and the
      remote buttons from step 6.
   e. If DeX, Windows mirroring, AirPlay or the screensaver broke, roll back
      in this order and test again after each step:
        1) `adb shell pm enable com.google.android.tungsten.setupwraith`, restart.
        2) If it's still broken: `adb shell pm enable com.google.android.apps.tv.launcherx`, restart.
      If step 2 was needed, Google TV's home screen stays on. Then make Home
      open tv-detox with its Accessibility "Home button" service (README,
      "Anything to watch out for on Chromecast with Google TV?": the adb
      version that keeps other accessibility services on). Tell me which
      step fixed it.
   f. If the TV gets stuck at the logo or keeps restarting, use README's
      "My TV is stuck restarting" steps straight away.

9. Turn off animations. Leave the TV's sleep timer as it is, and keep the
   screensaver on with its current settings: this is an LED TV, not OLED,
   but I want the screensaver. If the screensaver setting got changed,
   put it back to what you noted in step 4.

10. Wait 30 seconds after the last change, then restart the TV once and
    check it starts up normally before you say you're done. Show me free
    memory before and after. Run `sh check.sh 192.168.31.95:5555 my-tv.txt`
    and show me the result. TCL firmware may turn com.tcl.notereminder
    back on at every restart. If it does, note that in my-tv.txt rather
    than fighting it.

11. Once everything is checked, tell me to turn off USB debugging and
    Network debugging (or Developer options) on the TV. After I say I
    have, check that `adb connect 192.168.31.95:5555` is refused, and
    show me that. Left on, other devices on my network can ask to control
    the TV. I only turn it back on for undo.sh or updates.

When you're done, write a short UNDO.md here: how to turn everything back
on (undo.sh, then `pm enable` for each line in my-tv.txt), how to turn
Google TV's home and setup app back on, and how to change the top row from
the remote.
```

## What's different from the stock tv-detox prompt

- Builds the home screen from this folder with `build.sh`, so it includes the **All apps** tile and a
  single **Inputs** tile. OK on Inputs lists the HDMI inputs and switches to the one you pick; holding
  OK on it lets you pick an input to rename or hide. Hidden inputs come back by holding OK on **Add app**.
- Protects Miracast (Samsung DeX, Windows wireless display) and AirPlay. Chromecast is optional.
- Keeps every app you installed yourself (Hik-Connect, browser, File Manager+) and the WebView they use.
- Keeps the screensaver on and leaves the sleep timer alone.
- Sets the top row to JioHotstar, YouTube, Hik-Connect and Netflix.
- Tries replacing Google TV's home screen fully, then tests casting and rolls back step by step if
  anything breaks.
- Adds the TCL P635 notes: keep `com.tcl.suspension` and `com.tcl.useragreement`, test the remote
  buttons after each batch, check who saved the Wi-Fi, use port 5555.
