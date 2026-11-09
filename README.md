# LINCE

**One HTML file that reads a screen into click zones — and, served from a KVM, becomes the eyes and hands that drive the machine plugged into it.**

LINCE reads a desktop screenshot and returns every piece of print, button, field and icon as a numbered zone with coordinates. Served from a KVM running kvmd, the same page photographs the target through the KVM's HDMI capture and clicks and types through its USB. Nothing is installed on the target, nothing leaves the page, and no model runs at runtime: a model writes the script, the page runs it.

## Install

Copy the file into the KVM's web root, named as it will be served:

```sh
scp lince-r8.html root@<kvm-ip>:/usr/share/kvmd/web/lince.html     # kvmd web root
scp lince-r8.html root@<kvm-ip>:/usr/share/kvmd/glweb/lince.html   # GL.iNet build of kvmd
```

Open `https://<kvm-ip>/lince.html` in Chrome or Edge, accept the KVM's self-signed certificate, and sign in with the KVM's own login when the page asks. Scripts you want to load by URL go in the same folder.

## Use

1. **CONNECT** — opens the KVM's event socket; the capture wakes up.
2. **LOOK** — photographs the target and reads it: zones, text, coordinates. **AUTO LOOK** repeats it every few seconds.
3. Tap a zone on the picture. **CLICK THE SELECTED ZONE** clicks it on the target. With **DRIVE** on, every tap clicks for real, is recorded as a **TRAIL** step, and takes the next look — the walk gets recorded by walking it.
4. The **TRAIL** is that walk: each step keeps the zone, its coordinates, the screen it was on and the screen that came next. Copy it out and hand it to a model before it drives.
5. **RUN SCRIPT** runs the JavaScript in the script box; **STOP** ends it between steps. The box remembers its contents.

Coordinates: kvmd takes −32768…32767 on both axes, 0,0 at the screen centre. Every zone already carries its `kvmd` pair, so nothing is converted by hand.

## Scripts

The box runs its text as the body of an async function with `L` in scope:

```js
var zs = await L.look();        // photograph + read; zones with .text, .x, .y, .kvmd
L.zones();                      // the last read
L.find(/save/i);                // zones whose text matches
await L.click(zone);            // or L.click(kx, ky) in kvmd units
await L.key('Enter');           // L.keys('MetaLeft', 'KeyR') for a chord
await L.type('hello');          // typed through the KVM's keyboard
await L.wheel(-3, kx, ky);      // scroll at a point; positive is up
await L.wait(800);
L.log('note');                  // to the log under the box
if (L.stopped()) return;        // STOP was pressed
```

To run a script kept on the KVM instead of pasting it, the box holds one line:

```js
var s = await (await fetch('/mission.js')).text(); return await new Function('L', 'return (async function () {' + s + '})();')(L);
```

## Running it from the target

The page keeps working with its tab hidden or its window covered — the reading and every wait run through a Web Worker, which the browser doesn't throttle. So LINCE can sit in a background tab on the very machine it drives, with the window being worked on in front: start the script, bring that window forward, walk away. For long runs, add the KVM's address to Chrome's *Always keep these sites active* so Memory Saver doesn't discard the tab.
