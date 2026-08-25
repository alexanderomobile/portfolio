🇷🇺 [Русский](../ru/09-local-voice-assistant-smart-home.md) · 🇬🇧 [English](../en/09-local-voice-assistant-smart-home.md) · 🇪🇸 [Español](../es/09-local-voice-assistant-smart-home.md)

# Case Study: Local Voice Assistant & Smart Home

**Status:** ✅ Production

---

## Problem

A voice assistant and smart home running **entirely on owned hardware**. No speech transcript, no mail content, no calendar and no private messages ever leave the home perimeter — while matching commercial smart speakers feature for feature, and beating them on several scenarios.

One extra constraint: the assistant shares a single GPU with another project's production generative pipeline and **must yield it without failures**.

## Features

**Voice.** The wake word is detected on the device itself; then silence trimming, Russian speech recognition with punctuation, tool calling by the language model, and streaming synthesis sentence by sentence — the first sentence plays while the second is still being generated.

**Speaker recognition.** A voice fingerprint is matched against enrolled references in 40 ms and drives permissions: the owner gets everything, trusted household members get a limited set, an unknown voice gets no access to personal data. By design, voice never authorises irreversible actions.

**Personal data.** Calendar and tasks over CalDAV with phone sync and offline editing, reminders by voice and messenger, conversation memory, hybrid search across mail, notes and chats.

**Home.** A Zigbee mesh: dimmable tunable-white lighting, presence, climate, leak and opening sensors, smart sockets, scenes driven by presence and time of day. Controlled by voice, text and automations.

**Generation.** Images, music and video rendered on the local GPU from a spoken request, delivered to the messenger.

## Business value

Demonstrates that a private AI stack can be built on consumer hardware with no cloud subscriptions and no functional compromise: the same class of tasks as commercial assistants, with zero data egress and predictable cost of ownership. The architecture transfers directly to corporate environments where data is not allowed to leave the perimeter.

## Stack

**Platform.** Proxmox VE 9, nine LXC containers and a VM with GPU passthrough. Daily container snapshots and weekly VM snapshots, retention on a separate disk, automated backup audits and a guest-availability heartbeat.

**Language model.** llama.cpp on Vulkan, a 26B-class MoE model with QAT quantisation — 3.8B active parameters per token. A separate CPU-hosted service acts as the fallback path.

**Voice pipeline.** FastAPI, GigaAM (ASR), Silero (TTS and VAD), WeSpeaker in ONNX (voice fingerprint) — all CPU-bound, so it never competes with the GPU.

**Data.** Radicale (CalDAV), SQLite for state, Qdrant with BGE-M3 for hybrid retrieval — dense and sparse vectors fused with RRF, avoiding the morphology problems of classic BM25 in Russian.

**Infrastructure.** MariaDB, Dovecot and Postfix with inbound mail arriving through an edge worker and a tunnel, nginx and DNS on the gateway, Netdata in a parent/child topology, a WireGuard mesh with real certificates and not a single port exposed to the internet.

**Home.** Zigbee coordinator, Zigbee2MQTT, ESPHome satellites, local lighting API.

## Engineering decisions

**GPU arbitration.** Detecting a foreign render job by watching VRAM is already too late — by then the memory is taken and the job fails. A gatekeeper sits in front of the render queue: it evicts the language model, waits for memory to be released, and only then admits the job. The race is eliminated by construction rather than mitigated. A non-obvious follow-up was solved too: the generative engine does not release VRAM after finishing, so returning the model is preceded by an explicit weight unload.

**Egress restriction.** The assistant container may reach exactly three external destinations from an allow-list refreshed on a timer; everything else is dropped. Even a fully compromised container has nowhere to send data.

**Prompt-injection defence.** Text from mail and chats is fed to the model as data, never as instructions; tools are read-only; sending and deleting require explicit confirmation with the recipient shown; sensitive senders are never indexed at all — reading such a message is itself the leak.

**Channel separation.** Each channel keeps its own conversation memory: a spoken dialogue and a messenger thread never bleed into each other.

**Degrade, don't fail.** While the GPU is lent to rendering, the assistant openly announces the slow mode and keeps working on the CPU model.

## Functional blocks

Speech recognition · Voice fingerprint & permissions · Tool loop · Hybrid retrieval · Streaming synthesis · Calendar & reminders · Smart home · Media generation · GPU arbitration · Perimeter & monitoring

## Metrics

| metric | value |
|---|---|
| End-to-end voice turn | **1.3 s** (ASR 0.79 · model 0.22 · TTS 0.20) |
| Speaker recognition | 0.04 s; own voice 0.63–0.75 vs strangers 0.10–0.27 |
| Tool-call accuracy | **20 of 20** on the owner's typical phrases |
| Language model | 3400 tok/s prompt, 150 tok/s generation |
| Image / music generation | 14 s / 18 s |
| GPU handed back to the assistant | 112 s after the queue drains |

## Screenshots

![Architecture](../../assets/voice-assistant-smart-home-diagram.svg)

## Run

The infrastructure is described declaratively and deploys on any Proxmox node with a single 16 GB-class GPU. Configuration details are not published: the system serves a private perimeter.

---

[← Back to portfolio](https://github.com/alexanderomobile/portfolio/blob/main/README.en.md)
