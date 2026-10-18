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
* **Label regions** marks every NOAA-numbered active region on the disk with
  its number; see below.
* Drag to pan, pinch or scroll to zoom, double-tap to zoom in.

A link can choose what the viewer opens on. `ch` is a channel by its chip
label (any case), `dur` the duration in hours and `nf` the number of frames,
each one of the menu values, and `ar=1` turns on the region labels; anything
else is ignored. For example,
<https://sun-data.github.io/solar-movie-viewer/?ch=DEM> opens on the DEM, and
<https://sun-data.github.io/solar-movie-viewer/?ch=171&dur=24&nf=96&ar=1> on
a labeled day of AIA 171.

## Region labels

The labels come from the daily Solar Region Summaries of the NOAA Space
Weather Prediction Center, which it publishes for the last 30 days as
[JSON](https://services.swpc.noaa.gov/json/solar_regions.json). Every
numbered region is labeled, with or without sunspots, by its full number
(NOAA reports 14551 as 4551). A dot marks the reported position and the
number sits just north of it.

Each summary gives positions at 24:00 UT, to the nearest degree, so a frame
is labeled from the last summary before its UT day, rotated on to the frame
time at the same rates as Track rotation. Hence a label can step by a few
arcsec at 00 UT when the next summary takes over, a region numbered today is
labeled from tomorrow, and a region behind the limb is left off. With
tracking on, the labels stay with the de-rotated disk.

## DEM channel

The **DEM** chip shows how much coronal plasma there is at each temperature,
as color: hue is temperature, from violet at 0.4 MK through green near 2 MK
to red above 6 MK, and brightness is the amount. One fixed scale covers every
temperature and every frame, so brightness and color compare across the disk
and across the whole window. The key beside or under the disk gives the
scale; it hides with the controls.

These frames are not from Helioviewer. A job that runs every 15 minutes
fetches the AIA near-real-time synoptic images from JSOC (six coronal
passbands, 2.4 arcsec/pixel), inverts every 3-minute slot with
`utu.dem.plowman`, colors the result with `colorsynth`, and publishes the
last 48 hours as full-disk JPEGs plus a manifest. Hence some differences from
the other channels:

* the window ends on the newest DEM frame rather than now. JSOC posts each
  image about 11 minutes after it is taken, so that is usually around half
  an hour behind real time, and more when GitHub runs the job late;
* each frame is the one nearest its place in the even spacing, so frames
  can be up to 1.5 minutes off it; where frames are missing, the movie has
  fewer frames than asked for;
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
