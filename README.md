# solar-movie-viewer

A browser-based movie viewer for solar images, built for phones first.

**Live site: <https://sun-data.github.io/solar-movie-viewer/>**

Pick a channel, pick a time window, and the viewer assembles a movie from
[Helioviewer](https://helioviewer.org) imagery that you can scrub, loop, pan,
and pinch-zoom. Currently the channel list covers the nine
[SDO](https://sdo.gsfc.nasa.gov)/AIA passbands.

## Usage

* **Channel chips** select the AIA passband and reload the movie.
* **Duration / frames / resolution** control the time window, the number of
  frames requested, and the pixel size of each frame. Requesting many large
  frames can use a lot of memory, so the status line warns when the decoded
  sequence is likely to exceed what a phone browser will tolerate.
* **Fit disk** resets the view to the full solar disk.
* **Sharpen view** re-requests the current sequence at the plate scale of the
  region you have zoomed into, down to the 0.6 arcsec/pixel AIA native scale.
  Use this after zooming to recover detail rather than magnifying the
  full-disk frames.
* Drag to pan, pinch or scroll to zoom, double-tap to zoom in.

## Implementation

The whole viewer is a single self-contained `index.html` with no build step and
no dependencies. Frames come from the Helioviewer `takeScreenshot` API, are
decoded as ordinary images, and are composited onto a canvas with a view
transform tracked in arcseconds so that pan and zoom stay registered across
changes in plate scale.

## Deployment

GitHub Pages serves `index.html` from the default branch. Pushing to `main`
publishes.
