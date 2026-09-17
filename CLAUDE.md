# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

This is a frozen 2017 university coursework snapshot (P5, 5th semester, Electronics and Computer
Engineering, Aalborg University Esbjerg), kept for reference. It is **not maintained and not
expected to run** against current TensorFlow/CUDA versions — it targets TensorFlow 1.3's
`tf.contrib` API. There are no build, lint, or test commands: no package manifest, no test suite,
no CI. Treat changes here as archival/documentation work unless the user explicitly asks to get
the code running again (which would require pinning TF 1.3 and Python 3.5/3.6, and rewriting the
hardcoded Windows paths described below).

## Repository layout

- `src/` — the canonical, final code (cited directly by the report via `\lstinputlisting`).
- `report/` — the LaTeX report deliverable (`sections/`, `setup/`, `bib/`, `figures/`) and its
  compiled PDF. Several `.tex` files embed listings straight from `src/` (see Architecture below);
  moving or renaming files in `src/` will break those references unless updated to match.
- `archive/` — earlier iterations kept for historical interest, superseded by `src/`:
  `LSTM_Model/` (an earlier modular training-framework package that `src/rnn.py` and
  `src/tf_train_ctc.py` were distilled from) and `network-test/` (an earlier client/server draft
  with a different protocol than the final `src/client.py`/`src/serverPy.py`).
- `python_GoogleEngine/` — a distinct alternate approach the team explored: transcription via
  Google's off-the-shelf Speech API instead of the team's own model. Not part of the final
  pipeline.
- `MatLab/` — FFT/signal-processing exploration in MATLAB, not part of the final Python pipeline.
- `data/` — sample WAV/FLAC clips for manual testing.
- `figs&diagrams/`, `Links/` — supporting figures and reference notes.

## Architecture

The system is an offline training pipeline that produces a model, served on demand by a
client/server pair over a raw TCP socket (intended to run through an OpenVPN tunnel so a
low-powered client machine can request transcription from a GPU server):

1. **Preprocessing** (`src/`): `FlacToWavPy.py` converts the LibriSpeech/OpenSLR corpus from
   `.flac` to `.wav`; `FileNameParserPy.py` / `txtFileMaker.py` build the filename/transcript
   manifests used for training.
2. **Model architectures** (`src/rnn.py`): defines candidate networks — `SimpleLSTM`, `LSTM`, and
   three `BiRNN` variants (`BiRNN`, `BiRNN_V2`, `BiRNN_V3`) — each reading its hyperparameters from
   a `.ini` config section via `ConfigParser`. Layer layout originates from Mozilla DeepSpeech's
   `DeepSpeech.py`.
3. **Training** (`src/tf_train_ctc.py`): the `Tf_train_ctc` class loads audio, computes MFCC
   features, builds the graph using one of the `rnn.py` architecture functions, and trains with
   CTC (Connectionist Temporal Classification) loss.
4. **Inference entry point** (`src/modelLoadPyV2.py`): imports `tf_train_ctc.Tf_train_ctc` and
   calls `run_model()` against a trained model in inference mode. This is the script `serverPy.py`
   shells out to.
5. **Server** (`src/serverPy.py`): binds a TCP socket, accepts one client connection, receives a
   streamed `.wav` file to disk, launches `modelLoadPyV2.py` as a `subprocess.Popen`, and pipes
   the resulting transcript back to the client over the same socket. Has hardcoded Windows paths
   (`C:/Users/Mavericks/...`) from the original dev machine, including the old absolute path to
   `modelLoadPyV2.py` — that subprocess call was not repaired when the file moved to `src/`, since
   the repo isn't runnable as-is regardless.
6. **Client** (`src/client.py`): thin end — connects to the server, streams a local `.wav` file,
   and prints the transcription received back.

`rnn.py` and `tf_train_ctc.py` are the shared core that both the training path and
`modelLoadPyV2.py`'s inference path depend on — a change to one needs to stay consistent with the
other call site. `report/sections/implementation.tex` and `report/sections/model_development.tex`
embed excerpts of `src/serverPy.py`, `src/FlacToWavPy.py`, `src/txtFileMaker.py`, and `src/rnn.py`
by line range (`\lstinputlisting{../src/...}`) — editing those files' line numbers will shift what
the report displays without a corresponding line-range update in the `.tex`.

## Credit

Early experimentation started from a Silicon Valley Data Science RNN tutorial and Sirajology's
"How to Make a Simple Tensorflow Speech Recognizer" (code originally by pannous); neither is
vendored in this repo. The team's own models, built on top of and eventually well beyond those
starting points, live in `src/`. Originally developed under the team name "Mavericks" by Stefan
Bîrs, Tiberiu-Ioan Szatmari, and Kamran Thomas Alimagham.
