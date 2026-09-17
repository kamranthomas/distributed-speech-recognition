# Distributed Speech Recognition System

A university project (P5, 5th semester, Electronics and Computer Engineering, Aalborg University Esbjerg, 2017) that builds a neural-network-based speech recognizer and runs it as a client-server system over a network.

Originally developed under the team name "Mavericks" by **Stefan Bîrs**, **Tiberiu-Ioan Szatmari**, and **Kamran Thomas Alimagham**.

## What it does

- Trains recurrent neural networks (LSTM and bidirectional LSTM, with fully-connected layers) in TensorFlow 1.3 to transcribe speech to text, using CTC (Connectionist Temporal Classification) loss.
- Preprocesses raw audio into MFCC (Mel-Frequency Cepstral Coefficient) features before feeding it to the network.
- Trains and evaluates on the LibriSpeech / OpenSLR corpus (up to ~1000 hours of speech).
- Splits the system into a **server** (runs the trained model on a GPU machine) and a **client** (captures/sends audio, receives the transcription), connected over a TCP socket through an OpenVPN tunnel — so recognition can be requested from a separate, lower-powered machine.
- Compares three network topologies (simple LSTM, LSTM+FC, BiRNN+FC) and picks the best-performing one based on training/validation/test error rate.

## Repository layout

- `src/` — the canonical source code, cited directly by the report: `rnn.py` (network + training), `tf_train_ctc.py` (CTC training loop), `serverPy.py` / `client.py` (the client-server pair), `modelLoadPyV2.py` (the inference entry point `serverPy.py` shells out to), `FlacToWavPy.py` / `FileNameParserPy.py` / `txtFileMaker.py` (data-mining/preprocessing scripts).
- `report/` — the full LaTeX project report (introduction, speech processing theory, machine learning theory, model development/comparison, network framework, implementation, discussion, conclusion) and its compiled PDF.
- `archive/` — earlier iterations superseded by `src/`: `LSTM_Model/` (an earlier modular training-framework package) and `network-test/` (an earlier client/server protocol draft).
- `python_GoogleEngine/` — an alternate approach explored: transcription via Google's off-the-shelf Speech API instead of the team's own model.
- `MatLab/` — FFT/signal-processing exploration in MATLAB.
- `data/` — small sample audio clips used for manual testing.
- `figs&diagrams/`, `Links/` — supporting figures and reference notes gathered during the project.

## Background / credit

Early experimentation started from the simple LSTM reference model in Georgi Rubashkin's Silicon Valley Data Science RNN tutorial, and from ["How to Make a Simple Tensorflow Speech Recognizer"](https://youtu.be/u9FPqkuoEJ8) by Sirajology (code originally by [pannous](https://github.com/pannous)) — both cited in the report. That tutorial code isn't vendored in this repo; the team's own models (built on top of, and eventually well beyond, those starting points) live in `AAU_P5_report/code/`.

## Status

This is a snapshot of a 2017 coursework project, kept here for reference — it isn't maintained or expected to run against current TensorFlow/CUDA versions.
