# Local Sports Vision Analyzer (working title)

Status: planning. No product code has been written yet. Last updated 2026-11-05.

## Summary

A local-only, open-source tool that analyzes phone video of a training session
after the fact, for basketball, running and cycling. The user films a session
(phone on a tripod, or someone else filming) and gets feedback on form, how
fatigue changes that form over the session, efficiency, and injury-risk flags.
Results are saved per session so progress can be tracked over time.

Injury-risk flags are "worth a look" signals only. They are not medical
diagnoses, and the tool should say so wherever it shows them.

Everything runs on the owner's own Mac. Video and results never leave the
machine, and the public repo will contain code only.

## Reference projects

Inspiration only. We are building our own version, not forking.

- Reference repo: https://github.com/jeremyipark/vision-demos
  - Apache-2.0 licensed.
  - Small Python demos (running, basketball, push-ups, deadlift, climbing and others).
  - Uses ViTPose+ (pose) and, for basketball ball/rim, SAM 3.1, called through the
    VLM Run hosted gateway (video is uploaded to that service).
  - It has no cycling demo.
- Originating post: https://x.com/jeremyparkphd/status/2117672620283863070

## Our repo

- GitHub URL: https://github.com/CooperMcKay/local-sports-vision (private for now)
- Planned to be open-sourced after review.
- License: TBD (the reference repo is Apache-2.0; we have not chosen one).

## Decisions made so far

- Local only. No cloud API, no API keys.
- Hardware: MacBook Pro 14", M5, 32GB unified memory, 1TB SSD.
- Models come from Hugging Face and run offline after a one-time download.
  - Pin each model to an exact revision (commit).
  - Safetensors files only (no pickle .bin files).
  - `trust_remote_code=False`.
- The model cache stays in `~/.cache/huggingface`, outside the project folder, so
  model files cannot be committed by accident.
- Python 3.12 virtual environment in `.venv` inside the project folder.
  Python 3.12.16 and ffmpeg 9.0.2 are installed through Homebrew.
  (The system Python is 3.14.6; we avoid it because ML libraries may lag behind.)
- Processing is batch, after the session. It is not real time. A few minutes
  per clip is acceptable.
- Hosted VLM Run was reviewed and not chosen. Its gateway FAQ says it does not
  store or train on inputs, but its privacy policy is a generic template, the
  gateway is marked beta, and no terms page was found. Local avoids the question.

- Comparison approach (decided 2026-11-05): build a personal dataset over sessions and
  measure drift against the owner's own baseline (early vs late in a session, session
  vs session). No elite-runner benchmarks as the core feature; public norms may be shown
  later as clearly labeled context only. First sport/metrics focus: running fatigue and
  injury-risk flags (final metrics still to be designed).

## Security rules

- No secrets or API keys in the repo. `.env` is gitignored.
- No personal videos or results committed. `.gitignore` excludes video files,
  `data/`, `outputs/` and model weight files.
- Add a secret scanner as a pre-commit hook (for example gitleaks) before the
  first push to GitHub.
- Validate any uploaded or user-supplied file (type, size, readable by ffmpeg)
  before processing it.
- Pin all dependencies to exact versions in `requirements.txt`.
- Only load models we chose deliberately; never run code shipped inside a model.

## Open questions and unverified items

These come from memory or are guesses and must be confirmed.

- Whether SAM 3.1 is available as open weights that run locally. Unverified.
  If not, basketball ball/rim detection needs a different model.
- ViTPose likely needs a separate person detector in front of it (for example
  RT-DETR). Unverified; confirm from the Hugging Face model docs.
- Exact Hugging Face model ids. `usyd-community/vitpose-plus-large` is the name
  the reference repo uses; confirm it exists on huggingface.co before download.
- Real speed and accuracy on the M5 using PyTorch's MPS backend. Not measured.
  Download size of the pose model is also not checked.
- Which metrics best capture fatigue and injury-risk signals per sport.
  Not yet designed.
- Order of sports: guess is running first, then basketball, then cycling.

## Roadmap

Done:
- [x] Python 3.12.16 installed (Homebrew)
- [x] ffmpeg 9.0.2 installed (Homebrew)

- [x] `.gitignore` and `.venv` (Python 3.12) created
- [x] Pinned libraries installed (torch 2.15.0, transformers 5.22.1, scipy needed too)
- [x] Models downloaded, pinned: usyd-community/vitpose-plus-large (e211df37...) and
      PekingU/rtdetr_r50vd_coco_o365 (457857ce...), both Apache-2.0, safetensors
- [x] Spike on a living-room jog clip (2026-11-05): detector found the person in 211/211
      frames, ~164 ms/frame on M5 via MPS, skeleton visually on the body. Joints flicker
      slightly (needs temporal smoothing). Visual check only, not yet verified against
      ground truth. Spike code was throwaway and is not kept.

Next:
- [ ] Test with a proper basketball clip (also needs ball/hoop detection; SAM 3.1
      local availability still unverified)
- [ ] Decide metrics per sport, add smoothing, verify against hand-counted ground truth
- [ ] Finish brainstorming: written design document, reviewed and approved
      by the owner before any real building starts
- [ ] Implementation plan, then build, starting with one sport
- [ ] Create the GitHub repo (private first), add the secret scanner, choose a
      license, then open-source when ready
