# macOS Background Removal for OBS

## Introduction

This plugin provides an OBS filter to remove a persons background using macOS' built-in [Vision](https://developer.apple.com/documentation/vision) API.

This plugin requires OBS 31.1 or later. The release builds are made against OBS 33 on macOS 13 or later.

## Getting started

This is a fork of [sebastian-s-beckmann/obs-mac-backgroundremoval](https://github.com/sebastian-s-beckmann/obs-mac-backgroundremoval) that reworks how the mask is produced and applied: the soft matte from Vision is used as alpha instead of a hard threshold, each mask is composited onto the frame it was computed from, and the default quality is *Accurate*.

Download the `.zip` from the [releases](https://github.com/danieljrich/obs-mac-backgroundremoval/releases) page. The build there is for Apple silicon Macs and is not notarized, so it takes a few manual steps:

- Quit OBS.
- Unzip the download and move `obs-mac-backgroundremoval.plugin` into `~/Library/Application Support/obs-studio/plugins/` (create the folder if it doesn't exist; in Finder use Go → Go to Folder…). Remove any older copy of the plugin first.
- macOS marks downloaded files as quarantined, which stops OBS from loading the plugin. Clear that in Terminal:

  ```
  xattr -dr com.apple.quarantine ~/Library/Application\ Support/obs-studio/plugins/obs-mac-backgroundremoval.plugin
  ```
- Start OBS.

The filter can be found in the filters window under "Effect Filters".
It can be used on any type of source, not just Video Capture Devices!

## Settings

- **Quality**: which Vision model to use. *Accurate* produces a 2016x1512 matte and takes roughly 35 ms per frame on recent Apple silicon, so it tops out just under 30 fps. *Balanced* (512x384, ~10 ms) and *Fast* (256x192, ~5 ms) keep up with 60 fps.
- **Keep mask in sync with video**: holds each frame back until its own mask is ready, so the edge never trails behind movement. This delays the video by one frame in Balanced/Fast and two to three in Accurate; add the same amount as audio sync offset if lip sync matters. Turn it off for zero added delay at the cost of the mask lagging during motion.
- **Edge position / Edge softness**: at the defaults (0.5 / 1.0) the soft matte from Vision is used as-is. Lower the softness for a harder cut, move the position to grow or shrink the cut-out.
- **Edge refinement**: snaps the matte's edges to edges in the picture. Matters most for the low-resolution Balanced and Fast mattes.

## Building locally without Xcode

`./build-local.zsh` builds the plugin for this Mac's architecture using only the Xcode Command Line Tools, against the OBS installed in `/Applications`. `./build-local.zsh install` also places it in OBS's per-user plugin folder, moving any previous copy to `plugins-backup` next to it.

## License and Thanks
This plugin is licensed under the terms of the General Public License, Version 2. You can find the full text in the `LICENSE` file in this repository.

Huge credits also go to pkv from OBS who implemented a similar filter in OBS for NVIDIA GPUs. His code was referenced heavily in the creation of this plugin, especially during the early stages.

The build system is based on the [OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate). Its build instructions also apply to this plugin.
