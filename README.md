# MoCapViewer

<img src="icon/mocapviewer_icon_128.png" width="96" align="right" alt="MoCapViewer icon">

[繁體中文](https://github.com/WangI-Lin/MoCapViewer/blob/main/README.zh-TW)

**A Mokka-style viewer for C3D, TRC and BVH motion-capture files, written in MATLAB.**

MoCapViewer shows markers, skeletons, force plates and ground-reaction forces in 3D, plots marker, analog and
model-output curves, edits events, and saves the result as C3D, TRC or CSV. The mouse and keyboard work as in
[Mokka](https://biomechanical-toolkit.github.io/mokka/).

Everything it needs is in this repository. C3D files are read and written with
[ezc3d](https://github.com/pyomeca/ezc3d) (MIT licence), which is included; no MATLAB toolbox is required.

Author: **Dr. I-Lin Wang** (<ilin@gms.ndhu.edu.tw>)

## Quick start

```matlab
cd MoCapViewer
MoCapViewer                     % empty window, then File > Open... or drop a file on it
MoCapViewer('trial.c3d')        % open a file directly
```

- Needs MATLAB **R2021a or newer** (developed on R2026a, Apple Silicon).
- ezc3d 1.7.2 binaries are included for macOS (Apple Silicon and Intel) and Windows 64-bit. On Linux, build ezc3d
  with its MATLAB binding and put `ezc3dRead` and `ezc3dWrite` in `ezc3d/glnxa64/`.
- macOS, first start only: if the MEX files are blocked by Gatekeeper, run once in Terminal
  `xattr -dr com.apple.quarantine "<repository folder>/ezc3d"`.
- Do not keep the Biomechanical ToolKit or the Moveck Bridge on the MATLAB path: they use the same `btk*` function
  names. MoCapViewer removes them from the path when it starts.

Three TRC files to try are in `samples/`.

## What it does

**3D view**
- Left-drag rotates, middle-drag or Alt + left-drag pans, the mouse wheel zooms.
- Click selects a marker, Ctrl/Cmd + click adds to the selection, Shift + drag selects a box,
  Shift + Ctrl + click toggles a trajectory.
- Perspective, Top, Bottom, Front, Back, Left and Right views, in 1, 2 or 4 panes.
- Floor grid, axis triad, force plates with their **number (1, 2, 3 …) drawn flat on the plate**,
  ground-reaction force vectors and centre of pressure.
- Marker colour, radius, visibility and trajectory, and segments (links and faces), saved as Mokka `.mvc` files.
  Helen Hayes, CGM2 and Shadow configurations are in `models/` (*Model > Load configuration...*).

**Left panel**
- Markers, Virtual markers, Segments, Analogs and Model outputs, as in Mokka.
- Model outputs are the points a C3D file lists in `POINT:ANGLES`, `FORCES`, `MOMENTS`, `POWERS` and `SCALARS`.
  They are not drawn as markers; clicking one plots it.

**Time bar and events**
- Current-frame knob, region-of-interest handles and event markers (Alt + drag moves an event).
- Playback in real time, at 1/2, 1/4, 1/5 or 1/10 speed, or every frame, looping inside the region of interest.
- Insert, edit and delete events (table editor, Alt + 1…9, right-click on the time bar).
- Gait events detected from the vertical force of the force plates.
- Crop to the region of interest, renumber frames, set the first frame, resample, convert mm and m.

**Marker and analog tools**
- Markers: delete, rename, fill gaps (spline, pchip, linear), zero-lag Butterworth filter, average marker,
  distance and angle curves.
- Analogs: remove offset, Butterworth low-, high- and band-pass, rectify, scale, RMS or moving-average envelope,
  delete. The result replaces the channel or becomes a new `label_suffix` channel.

**Charts**
- Several series with a legend, hover read-out, click to jump to a frame, mouse-wheel zoom on X.
- X axis in frames, seconds or **% of the gait cycle** (all cycles, or mean ± SD).
- Chart data exports to CSV.

**Files**
- Drag and drop `.c3d`, `.trc`, `.bvh` and `.mvc` files onto the window.
- BVH joints become markers and the hierarchy becomes a `Skeleton` segment, scaled to a 1.75 m body height or a
  unit you choose. Saving as C3D converts BVH to C3D.
- Save as C3D or TRC. Export markers, analog channels, model outputs and events to CSV.
- One synchronised video with a time offset, or a video without motion data.

The full manual (Traditional Chinese) is in [docs/manual.zh-TW.md](docs/manual.zh-TW.md).

## Scripting

```matlab
app = MoCapViewer('trial.c3d');     % trial.mvc in the same folder is applied automatically
app.setFrame(120);
app.startPlayback(); app.stopPlayback();
app.setSelection([3 5]);
app.loadModelConfig('models/HelenHayes.mvc');
app.saveModelConfig('out.mvc');
```

- After changing the code while a window is open, run `MoCapViewer.closeAll; clear classes`.
- Checks without the window: `selftest_MoCapViewer('trial.c3d')` and `selftest_bvh('walk.bvh')`.

## Standalone application

With MATLAB Compiler:

```matlab
build_mac      % macOS:   build/MoCapViewer/MoCapViewer.app and an installer
build_win      % Windows: build\MoCapViewer\MoCapViewer.exe and an installer
```

The built application contains ezc3d and no Moveck Bridge code, so it can be shared, for example on GitHub
Releases. Ship [LICENSE](LICENSE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) with it. People who run it
need the free MATLAB Runtime of the same MATLAB release; the installer can download it.

## How C3D files are handled

MoCapViewer calls BTK-style functions (`btkReadAcquisition`, `btkGetMarkers`, …). In this repository those
functions are in `btk_ezc3d/` and are implemented on ezc3d; the core is `btk_ezc3d/mcv_btk.m`.

Differences from the Biomechanical ToolKit to be aware of:

- Force plates of type 1 to 4 are supported. No ground-reaction force is computed for other types.
- C3D files are saved as floating point, Intel byte order, with analog data in real units
  (`ANALOG:SCALE` = 1, `OFFSET` = 0). Camera masks are not kept.
- ezc3d uses the absolute value of `ANALOG:OFFSET`, so a file with negative offsets can give different analog
  values.

`compare_with_bridge.m` is a developer check: if you have the Moveck Bridge, it reads the same C3D file with the
Bridge and with ezc3d and lists every difference.

## Repository layout

- `MoCapViewer.m`: the application, one `classdef`. `MoCapViewerApp.m` is the entry point of the built application.
- `btk_ezc3d/`: BTK-compatible `btk*` functions implemented on ezc3d.
- `ezc3d/`: ezc3d 1.7.2 MATLAB binaries (`maca64`, `maci64`, `win64`).
- `mcv_read_trc.m`, `mcv_read_bvh.m`, `mcv_write_trc.m`, `mcv_write_csv.m`: TRC and BVH readers, exporters.
- `mcv_mvc.m`: Mokka `.mvc` read and write.
- `mcv_butterworth.m`, `mcv_fill_gaps.m`, `mcv_resample.m`, `mcv_envelope.m`, `mcv_cycle_normalize.m`: signal
  processing without toolboxes; these also run in GNU Octave.
- `mcv_file_drop.m`, `mcv_file_dialog.m`, `mcv_param_dialog.m`, `mcv_segment_dialog.m`: drag and drop, dialogs.
- `selftest_MoCapViewer.m`, `selftest_bvh.m`, `compare_with_bridge.m`: command-line checks.
- `build_mac.m`, `build_win.m`: standalone build scripts.
- `models/`, `samples/`, `icon/`, `docs/`: `.mvc` configurations, sample TRC files, icons, manual.
- `edition.txt`: contains `viewer`; it selects this edition of the shared `MoCapViewer` class.

## Other limitations

- The band-pass Butterworth filter is a high-pass and low-pass cascade, so the gain at the centre is slightly
  below 1.
- Not implemented from Mokka: STL and image-sequence export, Vicon VSK and VST skeletons.
- `.mvc` files follow the Mokka 0.6 format, and files saved here open in Mokka. Marker labels must match the data
  (case-insensitive).
- Event times use frame 1 = 0 s by default, as Mokka and Vicon do. Change it in
  *Settings > Event time reference*.

## Acknowledgements

- [Mokka](https://biomechanical-toolkit.github.io/mokka/) and the
  [Biomechanical ToolKit](https://github.com/Biomechanical-ToolKit), for the interaction model and the API this
  viewer follows.
- [ezc3d](https://github.com/pyomeca/ezc3d) by Michaud and Begon (pyomeca), for the C3D reader and writer.

## Licence

MoCapViewer is released under the MIT licence; see [LICENSE](LICENSE).

Third-party parts are listed with their licences in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md): ezc3d (MIT),
the BTK behaviour that `btk_ezc3d/` follows (BSD 3-Clause), `mcv_file_drop.m` (based on uiFileDnD, MIT),
`models/CGM2.mvc` (from pyCGM2, CC BY-SA 4.0) and `samples/*.trc` (OpenSim models, Apache-2.0).
