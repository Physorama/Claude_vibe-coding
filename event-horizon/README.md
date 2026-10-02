# Event Horizon

A real-time black hole raytracer in one HTML file. Every pixel follows a light ray along a Schwarzschild geodesic. Rays bend around the hole, cross a relativistic accretion disk, and either fall in or escape to a procedural sky. There are no dependencies and no build step. Open `index.html` in a browser that supports WebGL2.

![lensing](https://img.shields.io/badge/geodesics-Schwarzschild-ffad5c) ![webgl2](https://img.shields.io/badge/WebGL2-single%20file-93c6ff)

## What's simulated

| Effect | How |
|---|---|
| Light bending | Each pixel integrates `a = −1.5 h² r⃗ / r⁵` with a leapfrog step scaled by `r`. In Newtonian form this gives the exact shape of Schwarzschild null geodesics (h = specific angular momentum, r<sub>s</sub> = 1). |
| Shadow & photon ring | Comes out of the integration. Capture happens for impact parameters below 3√3/2 r<sub>s</sub>; rays that orbit near r = 1.5 r<sub>s</sub> pile up into the thin ring. |
| Disk | Infinitely thin, semi-transparent, inner edge at the ISCO (3 r<sub>s</sub>), with a Novikov–Thorne-style flux profile `r⁻³(1 − √(r_in/r))`. Temperature ∝ flux^¼. |
| Turbulence | 3D fBm sampled in polar coordinates and advected at the Keplerian rate ω = √(M/r³). Two phases are crossfaded, so the differential shear never winds up into stripes. |
| Doppler beaming | The emitter's velocity is √(M/(r − 2M)). Shift g = D · √(1 − r<sub>s</sub>/r). Colour comes from a blackbody at T·g, and brightness scales as g⁴. |
| Sky | Three star layers with blackbody colours, plus a lensed galactic band with dust lanes. A UV-grid mode shows the distortion directly. |

## Grading pipeline

Scene-linear HDR → bloom (dual-filter mip chain) → exposure / white balance / saturation / pivoted contrast → view transform → sRGB.
The view transforms are **AgX**, the same family as Blender 4.x's default, plus **AgX Punchy**, fitted **ACES**, and Standard. Lens fringing and film grain are applied last, in display space.

## Using it with Blender

**HDRI export.** *Render equirectangular HDRI* traces a 360° scene-linear panorama from the camera position and saves a Radiance `.hdr`. In the claude.ai viewer it arrives wrapped in a `.zip`, because the viewer only saves common file types. To use it:

1. World → Surface → **Environment Texture** → open the `.hdr`.
2. The camera's forward direction lands at the centre of the image, which Blender maps to **+X**. Add a *Mapping* node with *Texture Coordinate → Generated* and rotate Z to aim it.
3. The disk is a real light source, so metal hulls and glass pick up the blueshifted side. Try camera distances of 8–12 r<sub>s</sub> for an HDRI where the hole dominates the sky.

**Plates.** *Save frame* writes the graded PNG at screen resolution, with the panel's lens shift removed. Hide the UI with **H** first if you are framing a shot.

**Recreating it in Geometry Nodes or shaders.** The useful trick is the force law. If you march a ray as a particle under `−1.5·h²·P/|P|⁵`, you get correct black-hole lensing with no tensor maths. That works in an OSL script or in a Simulation Zone that steps points along curves.

## Controls

| Input | Action |
|---|---|
| drag / one finger | orbit |
| scroll / pinch | dolly (4–60 r<sub>s</sub>) |
| `space` | pause disk rotation |
| `S` | save PNG |
| `H` | hide UI (double-click to bring it back) |
| `1`–`5` | presets: Gargantua, Physical, Edge-on, Top-down, Lensing grid |

Performance: *Balanced* traces 320 steps per pixel at ¾ resolution. Drop to *Draft* on integrated GPUs.
