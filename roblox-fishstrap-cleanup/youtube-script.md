# YouTube Script — "Roblox Stuck on Weird Skies? Here's the Real Fix"

A non-technical, viewer-friendly script for explaining the Fishstrap custom-sky cleanup to fellow Roblox players. About 2–3 minutes of speaking time. Read straight or paraphrase.

Companion guide (link from your video description): https://github.com/Arc-y7b/discoveries/tree/main/roblox-fishstrap-cleanup

---

## Hook (first 10 seconds — keep eyes on screen)

So you installed Fishstrap. You loved the custom skies. Then you uninstalled it… and the weird skies are still there. The default Roblox skies are gone. Roblox Rivals doesn't look right anymore. Sound familiar? Stick around — I'm gonna show you exactly why this happens, and how to fix it in about five minutes. And trust me, it's not what 99% of YouTube tutorials are telling you to do.

## The Problem

Here's what's going on. When you uninstall Fishstrap from the normal "Add or Remove Programs" menu, Windows tells you it's gone. But it's lying to you. Three things get left behind on your computer.

**One** — a Fishstrap config folder hiding in your AppData.
**Two** — a folder full of Fishstrap log files in your Temp.
**Three** — and this is the one nobody talks about — Roblox's own asset cache.

That cache is the real villain here. Roblox keeps a hidden folder where it saves a copy of every texture, model, and yes — every sky — it's ever downloaded. While Fishstrap was running, it was sneaking custom skybox files into that cache. When you uninstalled Fishstrap, **those custom files stayed in the cache**. So Roblox is still pulling the modded skies, even though Fishstrap is gone.

## The Detective Work

Now, when I first looked at this, I almost fell into a trap. There's a folder inside Roblox's install with sky textures sitting right there — `content\sky` — and most tutorials will tell you "just delete those and you're done." I checked. Hashed the files. They were already the genuine Roblox originals. The mod wasn't there.

The mod was in the **asset cache**. Two-point-six gigabytes of cached game files, with the custom skies hiding inside. That's why generic "fixes" don't work for this problem.

## The Fix (the part that matters)

Here's what you actually need to do. I'll keep it simple:

1. Make sure Fishstrap is uninstalled. Settings, Apps, find it, remove it.
2. Close Roblox completely. And I mean completely — Roblox leaves zombie processes running in the background even after you close the window. Open Task Manager and end every "Roblox" process you see. This is the step everyone misses.
3. Delete the leftover Fishstrap folders in AppData.
4. Wipe the Roblox asset cache — that's the `rbx-storage` folder and a few database files next to it.
5. Launch Roblox. First load takes longer because it's redownloading the genuine assets. After that — defaults are back.

## The Payoff

That's it. Custom skies gone, Roblox Rivals looks the way the developers intended.

Now I wrote the entire step-by-step guide — every PowerShell command you need, every troubleshooting tip if something goes wrong, the nuclear option if your install is really cooked — and I put it on GitHub completely free. Link's in the description. Bookmark it, share it with your friends who are stuck with the same problem.

## CTA

If this saved you from reinstalling Roblox or from giving up and living with weird skies forever — hit that subscribe button. I dig into the stuff under the hood that other channels don't bother with. Next video I'm planning to cover [your next topic]. See you there.

---

## Bonus assets

### Video title options
- "The Roblox Sky Fix Nobody Else Is Telling You"
- "Why Uninstalling Fishstrap Doesn't Fix the Custom Skies (Real Fix)"
- "Roblox Rivals Default Skies Are GONE? Here's How to Get Them Back"

### Pinned comment / video description blurb

> Full step-by-step guide with every command: https://github.com/Arc-y7b/discoveries/tree/main/roblox-fishstrap-cleanup
>
> The short version: Fishstrap leaves custom skybox files in Roblox's hidden asset cache. Uninstalling Fishstrap doesn't clear it. We need to wipe the cache manually, but you have to kill Roblox's background processes first or Windows won't let you delete the files.

### Thumbnail text idea
"Fishstrap Uninstalled… BUT WHY ARE SKIES STILL CUSTOM?"
