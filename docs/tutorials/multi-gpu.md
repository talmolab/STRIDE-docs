# Multi-GPU inference

Pose inference on full-length clips is the slow step. `stride pose-inference` and
`stride roi-inference` can be started several times on the same folder: worker *i* of *N* takes
its share of the videos, and every worker skips a video whose output already exists. The commands
below are templates; replace the paths.

```bash
# one worker per GPU, 2 GPUs: run each line in its own terminal
stride pose-inference -i /data/videos -m /models/pose/centroid -m /models/pose/centered_instance \
    --gpu 0 --worker-id 0 --num-workers 2
stride pose-inference -i /data/videos -m /models/pose/centroid -m /models/pose/centered_instance \
    --gpu 1 --worker-id 1 --num-workers 2
```

- The pose files are written next to the videos (`-o` chooses another folder).
- Check completeness afterwards: the number of `.slp` files must equal the number of videos.
  Start the same command again to retry only the misses.
- `--device cuda:N` is the long form of `--gpu N`.

Then run the pipeline. It finds the pose files, skips pose inference for them, and does the rest
(ROI inference is light: 10 frames per video):

```bash
stride tmaze /data/videos \
    --roi-model /models/roi/centroid --roi-model /models/roi/centered_instance \
    --out /data/stride_out
```

For `stride gait`, the same pose files are used:

```bash
stride gait /data/videos --px-per-cm 26.1 --out /data/gait_out
```
