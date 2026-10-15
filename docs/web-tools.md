# Web tools

Two browser tools accompany the pipeline on the [project page](https://talmolab.github.io/STRIDE-page/tools/).
Both run entirely in the browser: nothing is uploaded and no server is involved.

## Rig checker

Drop a video. The page reads resolution, frame rate, codec and bitrate from the MP4 container
without decoding it (the `moov` index is found wherever the camera put it, so a 130 MB clip is read
in milliseconds), shows a frame, and lets you click two points of a known distance for the scale.
It then scores the recording against the [recording requirements](getting-started/recording.md),
per module, and lists what to change.

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

The port reproduces the pipeline's stride boundaries on the bundled demo trial. It applies no
confidence, edge, direction or turning filters, so treat its numbers as a preview; the pipeline's
`gait_per_stride_filtered.csv` is the authoritative table.
