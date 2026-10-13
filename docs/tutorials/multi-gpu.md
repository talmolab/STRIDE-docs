# Multi-GPU inference

Pose inference on full-length clips at 120 fps is the slow stage. The worker pattern splits a
directory across processes: worker *i* of *N* takes the videos whose sorted index satisfies
`index % N == i`, and every worker skips a video whose `.slp` already exists.

```bash
# 2 workers per GPU on 4 GPUs = 8 workers
for g in 0 1 2 3; do
  for w in 0 1; do
    wid=$((g*2 + w))
    CUDA_VISIBLE_DEVICES=$g stride pose-inference -i "$DAY" --model "$CENTROID" --model "$CENTERED" \
      --gpu 0 --worker-id $wid --num-workers 8 > "$DAY/pose-w$wid.log" 2>&1 &
  done
done
wait
```

- `--gpu 0` is correct inside each loop iteration: `CUDA_VISIBLE_DEVICES` makes the chosen card
  appear as device 0.
- Check completeness afterwards: the number of `.slp` files must equal the number of clips.
  Relaunch the same command to retry only the misses.
- Two machines can share a day: because workers skip existing outputs, a second machine walking
  the list from the other end only ever duplicates the single clip where the two meet.
- Memory: each worker holds both models; 8 workers fit comfortably on 24 GB cards at batch
  size 16.

ROI inference is light (10 frames per clip) and runs with many more workers per GPU.
