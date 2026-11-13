---
title: Speech inference
description: Serving speech recognition and text-to-speech models, where real-time conversational latency requirements are especially strict.
roadmap: inference-engineering
node: speech-inference
group: scaling-and-beyond-llms
video:
  provider: youtube
  id: "N7b1PJc7SFc"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1475
level: advanced
prerequisites: [continuous-batching, embedding-and-multimodal-inference]
takeaways:
  - A chained voice pipeline — speech-to-text, then a language model, then text-to-speech — has to run all three stages within a latency budget tight enough to feel like a real conversation, commonly under a few hundred milliseconds total.
  - Streaming every stage, rather than waiting for each one to fully finish before starting the next, is what makes a conversational latency budget achievable at all with a chained architecture.
  - Native speech-to-speech models process audio in and audio out directly, skipping the intermediate text representation entirely, which removes two full stage transitions from the latency budget at the cost of a newer, less proven technology.
check_questions:
  - q: Why does a chained voice pipeline need every stage to stream rather than wait for full completion?
    a: Waiting for speech-to-text to fully finish transcribing before the language model even starts, and waiting for the language model to fully finish before text-to-speech starts, would stack each stage's full latency sequentially. Streaming lets each stage begin working on partial output from the previous one, which is the only way to meet a tight conversational latency budget with three separate stages.
  - q: What specific latency cost does a native speech-to-speech model eliminate that a chained pipeline cannot avoid?
    a: It removes the transitions between stages — converting audio to text and then text back to audio — since the model processes audio directly on both ends. A chained pipeline pays for those conversions and for the handoffs between three separate models; a native model has neither.
deeper:
  - label: "AI Engineer — Building Effective Voice Agents (OpenAI)"
    url: https://www.youtube.com/watch?v=-OXiljTJxQU
related_formulas: []
---

## A latency budget tighter than almost anything else in this roadmap

Every lesson so far has treated latency as something to minimise within
whatever a product reasonably tolerates. Real-time voice conversation sets a
specific, unforgiving bar: a response that takes much longer than a few
hundred milliseconds to begin breaks the feeling of a natural conversation,
in a way a slightly slow chat response simply does not. Serving speech
models well means hitting this bar consistently, which shapes the
architecture choices covered in this lesson more than almost any other
constraint in this roadmap.

## The chained pipeline, and why every stage has to stream

The traditional architecture chains three separate models: **automatic
speech recognition (ASR)** converts spoken audio into text, a **language
model** processes that text and produces a response, and **text-to-speech
(TTS)** converts the response back into spoken audio. Run naively — wait for
ASR to fully finish before starting the language model, wait for the
language model to fully finish before starting TTS — the latencies of all
three stages stack sequentially, which makes hitting a sub-second total
budget essentially impossible. The fix is streaming every stage: ASR starts
emitting partial transcriptions before the speaker has finished talking, the
language model can begin generating a response from an early partial
transcript, and TTS begins synthesising audio from the first part of the
response before the rest has even been generated. This overlapping, rather
than sequential, execution is what makes a tight conversational budget
achievable with a three-stage chain at all.

## Native speech-to-speech: removing the stages entirely

An alternative architecture processes audio directly end to end, with no
intermediate text representation at all — the model takes spoken audio as
input and produces spoken audio as output, skipping the ASR and TTS
conversion stages and the handoffs between three separate models entirely.
This removes real latency cost by construction, since there are no stage
transitions left to pay for, but it is a newer approach with less production
track record than the well-established chained architecture, and it trades
away the ability to easily inspect or modify the intermediate text
representation, which a chained pipeline gets for free as a natural byproduct
of its architecture. The deeper link covers this exact contrast directly,
comparing the chained approach against a native speech-to-speech
implementation built on a realtime API.

## A rough budget, spent three ways

Picture a target of 500 milliseconds from the speaker finishing a sentence
to the first word of the response being audible. Without streaming, each
stage pays its own full latency in sequence: perhaps 150 milliseconds for
ASR to finalize a transcript, 200 milliseconds for the language model to
produce its first useful chunk of text, and 150 milliseconds for TTS to turn
that chunk into audio — already at the limit before anything has gone
wrong. Streaming changes what each number means: ASR's 150 milliseconds
becomes the lag before a word is confirmed, not a wait before anything
starts; the language model can begin reading a half-finished transcript
instead of waiting for the full one; and TTS can start speaking the first
few words of a response while the model is still generating the rest. The
budget becomes a ceiling on each stage's lag rather than a sum of three full
completions.

## Where this sits relative to the rest of this roadmap

Every technique covered earlier still applies to each individual model in
either architecture — a speech-to-text model still benefits from
[quantization](/roadmaps/inference-engineering/quantization) and a
well-chosen [inference engine](/roadmaps/inference-engineering/inference-engines), and a chained pipeline's overall latency is still bounded by
[time to first token](/roadmaps/inference-engineering/time-to-first-token) at each individual stage. What is specific to this lesson is the
orchestration across stages and the unusually tight end-to-end budget that
forces streaming everywhere, rather than any single model's serving
mechanics being fundamentally different from what the rest of this roadmap
covers.
