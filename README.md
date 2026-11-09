# LINCE

Desktop OCR for LLMs driving real machines through hardware KVMs. One HTML file, no dependencies, no network, no model at runtime.

LINCE reads a screenshot and returns every word, button, field and icon as a numbered zone with click coordinates. Served from a GL-KVM's web root, the same page captures the screen through HDMI and clicks through USB HID — nothing is installed on the target machine. A model writes a script; the page runs it.

## Install

Copy the file into the GL-KVM's web root:

```sh
scp lince-r8.html root@<kvm-ip>:/usr/share/kvmd/glweb/lince.html
```

Open `https://<kvm-ip>/lince.html` in Chrome or Edge. The page checks for a kvmd session and shows a login box if it needs one — same credentials as the KVM's own UI. Scripts you want to load by URL go in the same folder.

## Use

1. **CONNECT** — opens the kvmd WebSocket; the HDMI capture wakes up and HID is live.
2. **LOOK** — captures the target screen and reads it: zones, text, coordinates. **AUTO LOOK** repeats every few seconds.
3. Tap a zone on the picture. **CLICK THE SELECTED ZONE** clicks it on the target through USB. With **DRIVE** on, every tap clicks for real, records a **TRAIL** step, and takes the next look.
4. The **TRAIL** is the walkthrough: each step keeps the zone, its coordinates, the screen it was on, and the screen that came after the click. Hand it to a model before it drives so it starts with the spatial knowledge instead of guessing.
5. **RUN SCRIPT** runs the JavaScript in the script box; **STOP** ends it between steps.

Coordinates: kvmd takes −32768…32767 on both axes, 0,0 at screen centre. Every zone carries its `kvmd` pair already.

## Scripts

The script box runs its text as an async function with `L` in scope:

```js
var zs = await L.look();        // capture + read; zones with .text, .x, .y, .kvmd
L.zones();                      // the last read
L.find(/save/i);                // zones whose text matches
await L.click(zone);            // or L.click(kx, ky) in kvmd units
await L.key('Enter');           // L.keys('MetaLeft', 'KeyR') for a chord
await L.type('hello');          // typed through the KVM's keyboard
await L.wheel(-3, kx, ky);     // scroll at a point; positive is up
await L.wait(800);
L.log('note');                  // to the log under the box
if (L.stopped()) return;       // STOP was pressed
```

To load a script from the KVM instead of pasting it:

```js
var s = await (await fetch('/mission.js')).text();
return await new Function('L', 'return (async function () {' + s + '})();')(L);
```

## Background tabs

The page works with its tab hidden or covered — reading and waits run through a Web Worker, not timers the browser can throttle. LINCE can sit in a background Chrome tab on the machine it drives, with the working window in front. Start the script, switch to the target window, walk away. For long runs, add the KVM's address to Chrome's *Always keep these sites active* so Memory Saver doesn't discard the tab.
