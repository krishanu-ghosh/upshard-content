---
title: Multimodal AI
description: How images, audio and video reach a language model, what changes in an application when the input is not text, and where the costs move.
roadmap: ai-engineer
node: multimodal-ai
group: production
video:
  provider: youtube
  id: "Vb6mZLF3mN8"
  start: 0
  end: null
creator:
  name: The ML Tech Lead!
  channel_url: https://www.youtube.com/@TheMLTechLead
duration_seconds: 1036
level: advanced
prerequisites: [choosing-a-model]
takeaways:
  - A multimodal model converts each input type into tokens in a shared representation space, so the language model works on the same kind of thing throughout.
  - Images, and especially video, consume tokens quickly, which makes resolution and frame rate direct cost decisions.
  - Generation and understanding are separate capabilities with separate models, so a product usually needs a pipeline, not one model.
check_questions:
  - q: How does an image reach a language model that only consumes tokens?
    a: An encoder splits it into patches and projects them into the same embedding space the text tokens occupy. From the language model's perspective, they're just more tokens in the sequence.
  - q: Why is video so much more expensive than it looks?
    a: Frames are sampled, and each one costs an image's worth of tokens. A few minutes at a modest sample rate can exhaust a context window, so sampling rate is the main cost lever.
deeper:
  - label: "IBM Technology — What is multimodal AI?"
    url: https://www.youtube.com/watch?v=J51oZYcNvP8
  - label: "Google for Developers — How AI video generation models work"
    url: https://www.youtube.com/watch?v=Y5zHHoLrvAE
related_formulas: [scaled-dot-product-attention]
---

## One representation space

The trick that makes multimodal models work is mechanically unglamorous:
convert every input type into vectors that live in the same space the text
tokens already occupy, then let the usual transformer machinery — covered in
[how LLMs work](/roadmaps/ai-engineer/how-llms-work) — run over all of them
together, with no need to know which modality any given vector came from.

An image gets cut into a grid of small patches, and a trained encoder
projects each patch into a vector the same size as a text token's embedding.
Audio gets processed into acoustic features over short time windows and
projected the same way. Video gets sampled at some rate into individual
frames and treated as a sequence of images with an added time dimension.
Once that projection has happened, the model attends across modalities
essentially for free — by the time self-attention runs, every position in
the sequence is just a vector of the same shape, whether it started out as a
word, an image patch, or a moment of audio.

## Concrete token costs, so the bill isn't a surprise

Vision tokens scale with resolution in a way that catches people off guard
the first time they see a real bill. A low-resolution image might cost a few
hundred tokens once converted to patches. A high-resolution one can cost
several thousand — easily as much as multiple pages of dense text — purely
because of how finely it was divided before reaching the encoder.

Video compounds this directly. Each sampled frame costs roughly as much as a
still image, so a short clip sampled at even one frame per second can burn
tens of thousands of tokens within a few minutes — often enough to exhaust a
large fraction of a context window on video alone, with little budget left
for anything else in the prompt. Downscaling images before upload and
sampling fewer frames per second are the two most direct levers here, worth
treating as tuning parameters rather than whatever a library's default
happens to be.

## The voice latency chain, and where it breaks

A conversational voice agent built the traditional way chains three stages:
speech-to-text converts spoken audio into text, that text runs through the
ordinary LLM pipeline exactly as if it had been typed, and text-to-speech
converts the response back into audio. Each stage adds its own latency on
top of the others, and the sum is what the user actually experiences as the
pause before a reply begins.

Newer **native audio-to-audio** models process spoken input and produce
spoken output directly, inside a single model, skipping the intermediate
text representation and its two conversion stages entirely. That meaningfully
cuts end-to-end latency for this use case — at the cost of being a
considerably newer, less battle-tested technology than the well-established
three-stage chain, with fewer provider options and less production track
record to lean on when something breaks.

## Vision understanding versus OCR

For pulling information out of documents — invoices, scanned forms, tables
embedded in images — there are genuinely two competing approaches, not one
obvious answer.

Traditional **OCR** (optical character recognition) converts an image
directly into plain text, with high precision on the exact characters it
recognizes but essentially no understanding of layout or meaning beyond that
raw character sequence. A vision-capable language model can instead reason
directly over the image — correctly understanding that a number sits
specifically inside the "total due" cell of a table — but may occasionally
misread an individual character with more confidence than a dedicated OCR
engine would show for that same character.

In practice, many production pipelines combine both: OCR extracts raw,
reliable text as a cross-check, while a vision model separately reasons over
structure and layout, and the two get reconciled before anything downstream
treats the extracted data as trustworthy.

## Multimodal RAG: embedding images for retrieval

The retrieval techniques covered earlier in
[chunking](/roadmaps/ai-engineer/chunking) extend directly to images once a
suitable embedding model exists for them. An image can be embedded into the
same kind of vector space a text chunk occupies, and a query — text or
another image — retrieves the most semantically similar images from an
indexed corpus, using the same
[similarity search](/roadmaps/ai-engineer/similarity-search) machinery
covered earlier. This reaches well beyond image search as a feature: a
product catalogue can be searched by photo instead of typed description, or
a large corpus of scanned documents can be made searchable without first
needing a perfect text extraction of every page.

## Why this remains the thinnest node in the roadmap

No single source covers vision understanding, speech recognition, speech
synthesis, and generation together in one coherent treatment — and speech
specifically has essentially no good general explainer available anywhere.
Evaluation is also measurably harder here than for text: comparing two
candidate transcriptions, or two descriptions of the same image, isn't
simple string equality the way checking many text outputs can be. Eval
design for a multimodal feature genuinely needs more deliberate thought than
anywhere else in this roadmap, often leaning on
[LLM-as-judge](/roadmaps/ai-engineer/llm-as-judge) adapted specifically to
compare meaning rather than exact wording.
