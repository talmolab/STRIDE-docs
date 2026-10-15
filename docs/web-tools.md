# Web tools

Two browser tools accompany the pipeline on the [project page](https://talmolab.github.io/STRIDE-page/tools/).
Both run entirely in the browser: nothing is uploaded and no server is involved.

## Rig checker

Drop a video. The page reads resolution, frame rate, codec and bitrate from the MP4 container
without decoding it (the `moov` index is found wherever the camera put it, so a 130 MB clip is read
in milliseconds) and shows a frame. Then:

1. **Say what is in the video**: T-maze, another maze or corridor, open field, or treadmill, plus
   whether the paws are visible, the whole arena is in frame and the camera is perpendicular. The
   gait module needs visible paws (a camera below a transparent floor, or a side view); only the T
   geometry enables the decision module; a treadmill needs the belt speed and direction.
2. **Mark the scale**: two clicks on a known distance give px/cm.
3. **Mark the mouse**: two clicks from snout to tail base cross-check the scale against mouse size
   (adult mice measure about 6.3 to 7.5 cm in STRIDE's cohorts). A value far outside that range
   means the scale is wrong or the animal is not a mouse.
4. **Read the verdict and the recipe**: a per-module verdict against the
   [recording requirements](getting-started/recording.md), what to change, and a generated
   `config.yaml` batch block and command list with your fps and scale filled in, the stages to skip
   for your setup, a note on which pose model to use, and links to the matching docs pages.

If the browser cannot decode the codec (HEVC from DJI cameras, for example), export one frame and
drop it in for the scale step:

```bash
ffmpeg -ss 5 -i clip.MP4 -frames:v 1 frame.png
```

The checker scores the recording, not whether the trained pose models transfer to your rig.

## STRIDE viewer

Load a `.slp` pose file and a `.rois.yml` region file, optionally with the video. The viewer
parses the pose file with the lab's `sleap-io.js`, derives px/cm from `segment2` exactly as the
pipeline does, and runs a JavaScript port of the stride detector with the pipeline's default
thresholds. It shows the regions, the tail-base trajectory and the current pose on the frame, a
swing-and-stride timeline, summary tiles, and a per-stride table with CSV export.

The port reproduces the pipeline's stride boundaries on the bundled demo trial, and its stage-6
filters (confidence, edge strides, turning, backward strides) reproduce the pipeline's filtered set
on that trial (11 of 18 strides). Each filter can be switched off, removed strides can be shown, and
a speed band restricts the analysed strides to a range of walking speeds. Two clips can be loaded
side by side (for example a wild-type and a tau trial) with a table of medians and their difference,
best read at matched speed. For treadmill recordings, enter the belt speed and the direction the
belt surface moves in the frame: keypoints are moved into the ground frame before detection.

One known difference: the forward-direction filter uses the clip's own median stride length where
the pipeline uses the batch median. The pipeline's `gait_per_stride_filtered.csv` remains the
authoritative table.
