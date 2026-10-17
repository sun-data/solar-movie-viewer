# solar-movie-viewer

A browser-based movie viewer for solar images, built for phones first.

**Live site: <https://sun-data.github.io/solar-movie-viewer/>**

Pick a channel, pick a time window, and the viewer assembles a movie from
[Helioviewer](https://helioviewer.org) imagery that you can scrub, loop, pan,
and pinch-zoom. The channel list covers the nine
[SDO](https://sdo.gsfc.nasa.gov)/AIA passbands, the HMI line-of-sight
magnetogram, and a false-color differential emission measure (DEM) of the
last 48 hours.

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

A link can choose what the viewer opens on. `ch` is a channel by its chip
label (any case), `dur` the duration in hours and `nf` the number of frames,
each one of the menu values; anything else is ignored. For example,
<https://sun-data.github.io/solar-movie-viewer/?ch=DEM> opens on the DEM, and
<https://sun-data.github.io/solar-movie-viewer/?ch=171&dur=24&nf=96> on a day
of AIA 171.

## DEM channel

The **DEM** chip shows how much coronal plasma there is at each temperature,
as color: hue is temperature, from violet at 0.4 MK through green near 2 MK
to red above 6 MK, and brightness is the amount. One fixed scale covers every
temperature and every frame, so brightness and color compare across the disk
and across the whole window. The key beside or under the disk gives the
scale; it hides with the controls.

These frames are not from Helioviewer. An hourly job fetches the AIA
near-real-time synoptic images from JSOC (six coronal passbands, 2.4
arcsec/pixel), inverts every 12-minute slot with `utu.dem.plowman`, colors
the result with `colorsynth`, and publishes the last 48 hours as full-disk
JPEGs plus a manifest. Hence some differences from the other channels:

* the window ends on the newest DEM frame, up to about an hour and a quarter
  behind real time, rather than now;
* the frame count is an upper limit, met by taking every k-th frame so the
  spacing stays even;
* the resolution menu chooses between the 512 and 1024 pixel frames, and
  Sharpen view has nothing finer than 2.4 arcsec/pixel to fetch.

`?dem=URL` loads the DEM frames and manifest from somewhere else, such as a
local copy, which must send CORS headers.

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
no dependencies. Frames come from the Helioviewer `takeScreenshot` API (the
DEM from its published JPEGs), are decoded as ordinary images, and are
composited onto a canvas with a view transform tracked in arcseconds so that
pan and zoom stay registered across changes in plate scale.

## Deployment

GitHub Pages serves `index.html` from the default branch. Pushing to `main`
publishes.
