# Data

No data are included in this repository. Obtain both datasets from their authors and respect their licences
(both restrict use to non-commercial research; LAV-DF is derived from VoxCeleb2 and inherits its terms).

| Dataset | Used for | Source (please verify the current link and access procedure) |
|---|---|---|
| LAV-DF (Cai et al., DICTA 2022) | training, validation, within-dataset test | https://github.com/ControlNet/LAV-DF |
| FakeAVCeleb v1.2 (Khalid et al., 2021) | cross-dataset test only | https://github.com/DASH-Lab/FakeAVCeleb (access is by request form) |

## Expected layout

```
data/
  raw/
    LAV-DF/                    train/ dev/ test/ metadata.json (or metadata_min.json)
    FakeAVCeleb_v1.2/          meta_data.csv  RealVideo-RealAudio/ RealVideo-FakeAudio/ FakeVideo-RealAudio/ FakeVideo-FakeAudio/
  feats/                       created by notebook 02 (one .npz per clip) -- not committed
  runs/                        checkpoints and logs -- not committed
```

## How labels are derived (what the audit in notebook 01 enforces)

* **LAV-DF**: a clip is *fake* if `n_fakes > 0`, otherwise *real*. The group id is the fake clip's `original` source video
  (a real clip is its own group), so a fake never lands in a different partition from the clip it was derived from.
  200 demonstration clips in a `demo dataset/` folder have no metadata and are excluded (136,304 clips remain).
* **FakeAVCeleb**: labels and generation methods are read from `meta_data.csv` (columns `source`, `method`, `type`, `path`);
  file names alone are not reliable (several fakes carry no method tag in the file name). Any clip with manipulated video
  *or* audio is labelled fake. The speaker folder `idNNNNN` is the subject.
* **FaceForensics++** is **not** used: audio extraction failed for all 9,431 files in the copy we obtained because none contained an audio stream.

## Audio

Audio is read from the `.mp4` files through torchaudio's FFmpeg backend. If loading fails for every clip, check that the
system FFmpeg libraries are installed and that `ffprobe <file>.mp4` lists an audio stream.
