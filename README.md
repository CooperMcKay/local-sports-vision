# Local Sports Vision

A local-only tool that analyzes phone video of a training session and shows how
form changes as fatigue builds. Nothing is uploaded: video, models and results
all stay on the owner's own Mac.

> **Status: early development.** The pose-tracking pipeline has been proven on a
> test clip. The MVP below is not built yet.

## MVP: indoor cycling

The first version targets a bike on an indoor trainer, filmed by a fixed phone.
A fixed camera, steady light and repetitive pedaling make it the cleanest case,
and every session can be compared with the last.

Planned MVP measurements (design in progress):

- Cadence and how it drifts over a ride
- Knee extension at the bottom of the pedal stroke (side view)
- Hip sway and left/right differences (rear view)
- Torso angle over time
- Early-session vs late-session comparison, and session-vs-session trends

Injury-risk flags are "worth a look" signals only. They are not medical advice
or a diagnosis.

## Long-term goal

Use this for all of my Ironman training: bike and run first, and basketball as a
side sport. Swimming is not planned. Comparisons are against my own history
(a personal dataset built up session by session), not elite benchmarks.

## How it works

- Pose estimation: ViTPose+ Large (`usyd-community/vitpose-plus-large`)
- Person detection: RT-DETR (`PekingU/rtdetr_r50vd_coco_o365`)
- Runtime: PyTorch + Hugging Face Transformers, Apple Silicon GPU (MPS)
- Early result: ~164 ms per frame on an M5 MacBook Pro; the first test clip
  tracked the person in 211 of 211 frames (visual check only, not yet validated
  against ground truth)

## Privacy and security

- Local only: no cloud API, no API keys
- Models pinned to exact commits and loaded from `safetensors` files only
- Personal videos, results and secrets are gitignored and never committed
- Dependencies pinned in `requirements.txt`

## Inspiration

Inspired by [jeremyipark/vision-demos](https://github.com/jeremyipark/vision-demos)
(Apache-2.0). This is an independent, local-first implementation.

## Setup (current)

Python 3.12 and ffmpeg are required. Create a virtual environment, then
`pip install -r requirements.txt`. Models are downloaded once from Hugging Face
at the pinned commits listed above. A full setup guide will follow with the MVP.

## License

Not chosen yet. Do not assume any license until one is added.
