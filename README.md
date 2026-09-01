# solar-movie-viewer

A browser-based movie viewer for solar images, built for phones first.

**Live site: <https://sun-data.github.io/solar-movie-viewer/>**

Pick a channel, pick a time window, and the viewer assembles a movie from
[Helioviewer](https://helioviewer.org) imagery that you can scrub, loop, pan,
and pinch-zoom. The channel list covers the nine
[SDO](https://sdo.gsfc.nasa.gov)/AIA passbands plus the HMI line-of-sight
magnetogram.

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
* **Track rotation** de-rotates every frame to the timestamp of the frame that
  was showing when you pressed it, so the solar surface holds still and only
  genuine evolution moves. The view itself does not move, so the limb stays
  where it is.
* Drag to pan, pinch or scroll to zoom, double-tap to zoom in.

## Differential rotation

Tracking remaps each frame rather than sliding it. For every point: screen to
helioprojective, to heliographic, advance the longitude by the local rotation
rate, project back, and sample there. The rate is Snodgrass (1984) for small
magnetic features, converted from sidereal to synodic because the observer
rides along with Earth, and the tilt of the solar axis (B0, up to 7.25
degrees) is carried through, so features move vertically as well as
horizontally.

Two deliberate approximations. The remap is built from a mesh of small affine
patches, because Canvas 2D applies only affine transforms and WebGL cannot
touch these frames at all: neither Helioviewer nor the SDO archive sends CORS
headers, so the images taint the canvas and cannot be used as textures or read
back. And cells touching the limb are drawn unwarped, because the map is
singular there and refining the mesh does not converge.

Good to well under a pixel on a zoomed-in region, which is what tracking is
for. Not a substitute for a real reprojection if you need to measure
something.

## Implementation

The whole viewer is a single self-contained `index.html` with no build step and
no dependencies. Frames come from the Helioviewer `takeScreenshot` API, are
decoded as ordinary images, and are composited onto a canvas with a view
transform tracked in arcseconds so that pan and zoom stay registered across
changes in plate scale.

## Deployment

GitHub Pages serves `index.html` from the default branch. Pushing to `main`
publishes.
