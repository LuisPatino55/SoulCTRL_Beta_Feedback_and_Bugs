<<<<<<< HEAD
# \# SoulCTRL

# 

# A Python framework for creating emotionally intelligent NPC interactions powered by a local or cloud LLM. Ships with \[Ollama](https://ollama.com) as the default offline provider; any OpenAI-compatible API works as a drop-in replacement with no code changes.

# 

# \---

# 

# \## Requirements

# 

# \- Python 3.11+

# \- \[Ollama](https://ollama.com/docs/installation) running locally \*(default provider; swap to any cloud API via `/manage` or `LLM ServiceModels Manager` button in the webGUI interface)\*

# \- Python packages: `openai`, `keyring`, `colorama`

# \- Optional browser GUI packages: `fastapi`, `uvicorn`

# 

# \## Windows Installer (Recommended First Run)

# 

# If Python 3.11+ is already installed, run:

# 

# ```bat

# install\_chat\_system.bat

# ```

# 

# Optional flags:

# 

# ```bat

# install\_chat\_system.bat -ModelName qwen2.5 -NoLaunch

# ```

# 

# What the installer does:

# 

# \- Creates `.venv`

# \- Installs package dependencies with GUI extras

# \- Attempts to install Ollama automatically (best effort)

# \- Prompts for your default model name and saves it as chat and emotion model

# \- Adds that model name to the Ollama provider model list (no validation)

# \- Attempts `ollama pull <model>` and reports if pull started; otherwise tells you to pull manually

# \- Launches `run\_chat\_gui.py`

# \- Creates `launcher.bat` and a `Launch SoulCTRL.lnk` shortcut in the project folder

# 

# After installation, non-technical users can start the app by double-clicking `Launch SoulCTRL.lnk`

# or `launcher.bat`, which opens a small window to choose \*\*Console\*\* or \*\*GUI\*\* mode.

# 

# Notes:

# 

# \- Python is still a prerequisite and is not installed by this script.

# \- If Ollama auto-install is unavailable on your machine, install it manually from https://ollama.com/download and rerun.

# 

# ```bash

# pip install openai keyring colorama

# ollama serve

# ```

# 

# Browser GUI optional install (Highly recommended for best experience):

# ```bash

# pip install .\[gui]

# ```

# 

# \---

# 

# \## Quick Start

# 

# 

# \### Console mode

# ```bash

# python run\_chat.py

# ```

# 

# 

# \### Programmatic / No UI mode (headless)

# 

# ```python

# from npc\_chat\_system import NpcChatSystem, ConsoleOutputAdapter, ConsoleInputAdapter

# 

# system = NpcChatSystem()

# system.initialize()

# db = system.database\_manager

# db.load\_user\_profile("Default\_User")

# db.load\_npc("Aria\_Stone")

# db.load\_location("Downtown\_Cafe")

# system.open\_player\_mode()

# ```

# 

# 

# \### Browser GUI mode (Best experience)

# 

# ```bash

# python run\_chat\_gui.py

# ```

# 

# Optional flags:

# 

# \- `--host 127.0.0.1`

# \- `--port 8765`

# \- `--no-browser`

# 

# \---

# 

# \## Features

# 

# 

# \### Conversation System

# 

# \- LLM-powered NPC dialogue \& story creation via `SoulCTRL` + `LLMService`

# \- Streaming responses (token-by-token) or full-text mode (`Set in the Chat Player Menu`)

# \- Multi-layered NPC personality `judge components`, create more nuanced behaviour, responses, and actions

# \- Asynchronous `judge component` scheduler — judges run in background threads after each NPC reply; chat input stays responsive

# \- Deferred `judge decision` application — results are injected into the \*next\* system prompt so no turn is blocked and the NPC can respond immediately to component decisions

# \- Dynamic and highly configurable emotions, relationships, behaviour, stats, and stat definitions

# \- Automatic memory collection, with manual overrides and full edit control

# \- Interactable item system that the NPC uses autonomously to trigger actions (eat, drink, change locations, use weapons, charge and pay currency, etc.)

# \- Conversation archiving: live history compresses to archive chunks when it exceeds `chat\_history\_compression\_threshold`; past context is summarized by the LLM and kept as a rolling archive

# \- Multiple user profiles with per-user \& per NPC (memory, relationship stats, and conversation history)

# 

# 

# \### NPC Profiles

# 

# Each NPC (`ChatNPC`) carries:

# 

# | Component                   | Contents                                                                      |

# |-----------------------------|-------------------------------------------------------------------------------|

# | \*\*Bio\*\*                     | Name, age, gender, height, weight, build, hair/eye color, skin tone, fashion  |

# | \*\*Backstory\*\*               | Freeform history string injected into the system prompt                       |

# | \*\*Personality traits\*\*      | Positive traits — most influential prompt input                               |

# | \*\*Character flaws\*\*         | Negative traits                                                               |

# | \*\*Distinguishing features\*\* | Physical details                                                              |

# | \*\*Emotion component\*\*       | `EmotionalVector` + per-axis `EmotionAxis` settings                           |

# | \*\*Stats\*\*                   | Social stats per user: Respect, Attraction, Desire, Danger, Hunger, Energy    |

# | \*\*Relationship category\*\*   | Stranger → Acquaintance → Friend → Close Friend → Romantic Partner, etc.      |

# | \*\*Memory points\*\*           | Named `MemoryPoint` entries attached to the NPC–user pair                     |

# | \*\*Situation advice\*\*        | Active `SituationAdvice` entries injected into the next prompt                |

# | \*\*Inventory \& Currency\*\*    | `Item` storage, NPC `self-item use`, `currency` trading system                |

# | \*\*Health\*\*                  | `Health` component affected by items and other factors                        |

# | \*\*Keychain\*\*                | NPC-owned keys for unlocking doors, vehicles, or other interactables          |

# 

# 

# 

# \### 4-Dimensional Emotion Model

# 

# Availabe Emotions: 

# \- `NEUTRAL` - \*\*EmotionalVector\*\*(0.0, 0.0, 0.0, 0.0)

# \- `HAPPY` - \*\*EmotionalVector\*\*(0.5, 0.5, 0.5, 0.5)

# \- `ELATED` - \*\*EmotionalVector\*\*(0.8, 0.5, 1.0, 0.5)

# 

# \- `SURPRISED` - \*\*EmotionalVector\*\*(0.7, 0.0, 0.0, 0.0)

# \- `EXCITED` - \*\*EmotionalVector\*\*(0.7, 0.4, 0.9, 0.5)

# 

# \- `SAD` - \*\*EmotionalVector\*\*(-0.3, -0.5, -0.3, -0.3)

# \- `DISGUSTED` - \*\*EmotionalVector\*\*(-0.5, -0.0, -0.7, -0.6)

# \- `JEALOUS` - \*\*EmotionalVector\*\*(-0.7, 0.6, -0.7, -0.6)

# 

# \- `ANNOYED` - \*\*EmotionalVector\*\*(0.1, 0.4, -0.5, 0.0)

# \- `ANGRY` - \*\*EmotionalVector\*\*(0.9, 0.8, -1.0, -1.0)

# 

# \- `LOVING` - \*\*EmotionalVector\*\*(0.6, 0.6, 0.8, 1.0)

# \- `ROMANTIC` - \*\*EmotionalVector\*\*(0.8, 0.3, 1.0, 0.5)

# \- `LUSTFUL` - \*\*EmotionalVector\*\*(1.0, 1.0, 1.0, 1.0)

# 

# \- `SICK` - \*\*EmotionalVector\*\*(-0.7, -0.5, -0.6, -0.4)

# \- `FEARFUL` - \*\*EmotionalVector\*\*(-0.8, -0.7, -1.0, -0.7)

# \- `PANICKED` - \*\*EmotionalVector\*\*(-1.0, -1.0, -1.0, -1.0)

# 

# 

# NPCs maintain an `EmotionalVector` with four axes:

# | Axis          | Description                                     |

# |---------------|-------------------------------------------------|

# | Arousal  | Activation level (angered ↔ excited)            |

# | Dominance | Perceived social power (submissive ↔ dominant)  |

# | Pleasure  | Hedonic valence (negative ↔ positive)           |

# | Trust     | Openness (guarded ↔ open)                       |

# 

# Each axis has an individual `EmotionAxis` config:

# \- `weight`      — impulse sensitivity multiplier

# \- `baseline`    — resting value the axis decays toward

# \- `decay\_rate`  — speed of return to baseline per game tick

# 

# Each axis also defines one Personality Trait for the NPC, based on that axis' baseline value. You can find these definitions

# at: `Erosive\_Tech/Instructions/Personality/` and the approriate folder for each axis. These files are created in the directory

# when they do not exist, and can be edited by you to change the personality trait definitions for each axis. If you wish to reset to a default definition, you can delete that file and it will be recreated on next run. The default folder structure is: 

# \- `Aggression/` - Arousal definitions  

# \- `Assertiveness/` - Dominance definitions

# \- `Openness/` - Trust definitions

# \- `SocialEnergy/`- Pleasure definitions 

# 

# 

# The `EmotionalJudge` runs after each turn and writes an `EmotionalVector` impulse back to the NPC automatically. Which 

# is based on the emotion response that this NPC would feel, when receiving the last user input. The impulse is then applied to the NPC's `EmotionalVector` and the next system prompt is updated with the new values (multiplied by the Impulse Multiplier setting in the Emotion Editor) and then these values are decayed, applying the decay rate for each axis. 

# 

# The `EmotionalJudge` can be disabled in the settings, or in the webGUI Main Page, if you wish to not have this feature enabled.

# 

# 

# 

# \### Judge Component System

# 

# Four built-in asynchronous judges, each configurable with `cooldown\_rounds`:

# 

# | Judge                   | Default cooldown      | Purpose                                                 |

# |-------------------------|----------------------|---------------------------------------------------------|

# | `MemoryExtractionJudge` | 10 rounds             | Extracts memorable facts about the interactions         |

# | `SituationalAdviceJudge`| 1 rounds              | Detects social cues and provides advice to the NPC      |

# | `ActionJudge`           | 2 round               | Triggers item use \& other available actions by the NPC  |

# | `EmotionalJudge`        | 0 rounds (every turn) | Applies impulses to the NPC (affecting current emotion) |

# 

# Cooldowns are configurable at runtime via `/judge\_cd \[name] \[rounds]` in the Chat Player.

# Settings also available on the webGUI Main Page, Welcome Panel.

# 

# Custom judges can be added by subclassing `NpcJudge` and appending to `npc\_message\_system.judges`.

# 

# If you run into performance issues with judges, you can disable them in the main panel. I recommend using the `EmotionalJudge` at minimum, as it is the most important for driving the emotional state of the NPC. The next most important in my opinion is the `MemoryExtractionJudge`, as it allows the NPC to remember things about the user and the conversation.

# 

# \- While I feel the `ActionJudge` is a lot of fun, it can be disabled if you wish to have the NPC not use items or trigger actions. 

# 

# 

# \### Memory System

# 

# \- \*\*`RelationshipDetails`\*\* — tracks per-user per-NPC goals and facts about the state of the relationship with the user

# \- \*\*`MemoryPoint`\*\* — a tracked item with a `TaskStatus` lifecycle: `PENDING → IN\_PROGRESS → COMPLETED / FAILED`

# \- Time-limited points count down via `tick(delta)` and auto-fail on expiry

# \- \*\*`SituationAdvice`\*\* — extends `MemoryPoint`; starts `IN\_PROGRESS` and produces an LLM prompt fragment via `get\_llm\_prompt\_instruction()`

# \- Memory points are stored per user-NPC pair and persist with the user profile

# \- Relationship details and memory points are both updated on ticks and can auto-expire when their timers reach zero

# \- Permanent entries use `time\_remaining <= -10` and are never expired by the update loop

# 

# 

# 

# \### Location System

# 

# \- Locations carry a name, description, atmosphere, time-of-day, and danger level (0–1 Percentage)

# \- Time and danger are read by the `NpcMessageSystem` each turn and included in the NPC system prompt

# \- \*\*Scene transitions\*\* move the NPC to a new location mid-conversation with an optional LLM-generated transition message

# \- \*\*Interactable items\*\* can be attached; the `ActionJudge` can trigger item actions based on conversation context

# 

# 

# 

# \### Interactable Items \& Inventory

# 

# \- Typed interactable items: `Gun`, `Door`, `Vehicle`, `Consumable`, and more (see `interactable\_items.py`)

# \- \*\*User inventory\*\* (`InventoryManager`) — items owned by the player, independent of location items; accessible in chat via `/items` on in the `user info panel` in the webGUI

# \- \*\*NPC inventory\*\* — NPC-owned interactables can be triggered by the player via `/use\_item`, by right-clicking the NPC in the webGUI, or by the `ActionJudge` automatically

# \- \*\*Location inventory\*\* — scene interactables can be browsed/triggered via `/loc\_items`, by right-clicking the location in the webGUI, or by the `ActionJudge` automatically (if the NPC is using the item)

# \- \*\*Transfer flow\*\* — `/give` \& `Give NPC Item Button` moves selected user items to the active NPC inventory and creates a `MemoryPoint` for the NPC to remember the gift; `/use\_item` triggers an action and creates a `MemoryPoint` for the NPC to be aware they used the item

# \- \*\*Consumable catalog\*\* (`consumable\_items.py`) — prebuilt consumables tagged by type: `food`, `drink`, `drug`, `lotion`, `medicine`, etc. Each has a `use()` action that applies effects to the NPC or user, including stat deltas, relationship impulses, and status effects

# \- `InteractableFactory` restores typed items from save data and supports type-filtered queries

# \- Action execution supports key-based item lookup, target resolution, status-effect IDs, and use-limit depletion handling

# 

# 

# \### Status Effects

# 

# \- \*\*ID-driven registry\*\* — define named effects once, reference by ID anywhere

# \- Three duration types: `Instant` (one-shot), `Timed` (counts down), `Permanent` (never expires)

# \- Three payload channels:

# &#x20; - `EmotionalVector` impulse

# &#x20; - `RelationshipImpulse` (stat delta applied to NPC relationship)

# &#x20; - `StatManager` delta (e.g. Health)

# \- Universal cooldowns — even Instant effects can have a reuse cooldown

# \- Re-applying an active Timed effect refreshes its duration instead of stacking

# \- Active effects save and restore with the NPC profile (`effect\_id + remaining\_time`)

# 

# 

# \### User Profiles

# 

# \- Stores preferences, active NPC/location selections, relationship history, and memory per NPC

# \- `auto\_initialize\_chat\_enabled` / `auto\_initialize\_chat\_cooldown\_seconds` — optional idle-triggered NPC follow-up

# \- `generate\_previously\_on\_message` — toggle "Previously On…" conversation summary on resume

# \- `generate\_scene\_transition\_message` — toggle LLM-generated scene transition messages

# \- Includes a `StatManager` with a default `Health` stat (100)

# \- Per-user per-NPC relationship data: stats, category, memory points, conversation history

# 

# 

# \### LLM Service

# 

# \- Provider-agnostic routing via `LLMService` + `LLMProviderConfig`

# \- Default provider: local Ollama at `http://localhost:11434/v1`

# \- Switch to any OpenAI-compatible cloud API via `/manage` — no code changes required

# \- API keys stored in the OS credential store (Windows Credential Manager / macOS Keychain / Linux Secret Service) — \*\*never written to disk\*\*

# \- Separate model slots: `chat\_model\_name` (dialogue \& helper), `emotional\_model\_name` (judges) . For best performance, use a smaller (instruct) model for the emotional model and a larger model for the chat model. The system is designed to efficiently switch between models at an optimal time, but if you run into performance issues, you can leave the emotion model blank to use the chat model for both. 

# 

# 

# 

# \### System Helper (`/?` and `/help`)

# 

# An LLM-backed in-app assistant that answers questions about navigation and usage:

# 

# \- Access quick-help with `/?` or `/? <question>` from the main menu

# \- Start full interactive helper session with `/help` (plain text questions accepted without a slash)

# \- In `/help` session mode: `/main` returns to the main menu and `/quit` exits the app

# \- Conversation history is persisted per user as `{user\_id}\_helper\_history.json`

# \- Guide content is loaded from `SOULCTRL\_HELPER\_GUIDE.md` (or the workspace READMEs as fallback) and injected into the assistant's system prompt

# \- `/hist\_clr` — clear helper conversation history for the current user

# \- `/reset` — reload helper guide content from workspace documents

# 

# 

# 

# \### I/O and Event Integration

# 

# The core runtime supports custom frontends through adapter interfaces plus a typed event bus.

# 

# \- \*\*I/O adapters\*\* (`IOutputAdapter`, `IInputAdapter`) let you run in console, GUI, or headless server mode.

# \- Input adapters may return `None` from `get\_input()` / `get\_timed\_input()` to indicate "no input ready this tick" (yield semantics for polling loops).

# \- Streaming responses are handled through `begin\_npc\_stream()` and `StreamHandle.write()/close()`.

# \- \*\*Event bus\*\* (`EventBus`) publishes strongly typed lifecycle events such as `ConversationTurnEvent`, `NpcLoadedEvent`, `LocationChangedEvent`, and `MenuUpdatedEvent`.

# \- Use event subscriptions for UI refresh, analytics, telemetry, gameplay hooks, and cross-system automation without patching core loop code.

# \- See `SOULCTRL\_API\_GUIDE.md` for full integration contracts and examples.

# 

# 

# 

# \### Text-to-Speech (TTS) Supports OpenAI, ElevenLabs, and local fallback.

# 

# \- Optional TTS for NPC responses (`settings.tts\_enabled`)

# \- Default online provider order is OpenAI (`settings.tts\_online\_provider\_order = \["openai"]`)

# \- ElevenLabs is opt-in: add `"elevenlabs"` to `settings.tts\_online\_provider\_order`

# \- ElevenLabs requires a configured voice id (`settings.tts\_elevenlabs\_voice\_id` or runtime `tts\_voice`)

# \- ElevenLabs output formats: `wav\_16000`, `wav\_22050`, `wav\_24000`, `wav\_32000`, `wav\_44100`, `pcm\_16000`, `pcm\_22050`, `pcm\_24000`, `pcm\_32000`, `pcm\_44100`

# \- Output modes: `"playback"`, `"file"`, or `"both"`

# \- Audio files saved to `Erosive\_Tech/TTS\_Audio/`

# \- Output retention keeps the latest `settings.tts\_max\_output\_clips` files (default `20`)

# \- Offline-only mode is available via `settings.tts\_offline\_only` (uses silent WAV fallback unless replaced)

# 

# 

# 

# \### Instruction File Customization (Full Mode)

# 

# In full mode (`allow\_disk\_overrides=True`) the system exports default LLM instruction definitions to:

# 

# ```

# Erosive\_Tech/Instructions/

# ├── Stats/               # Per-stat behavioral guidance for the LLM

# ├── Personality/         # Personality trait definitions

# ├── Emotions/            # Emotion axis definitions

# └── System\_Helper/       # System helper prompt and guide content

# &#x20;   ├── system\_instruction.cfg

# &#x20;   └── guide\_content.cfg

# ```

# 

# Edit these `.cfg` files to change how the LLM interprets NPC stats and behaviors without touching source code. In demo mode these files are never written or read — hard-coded in-memory defaults are used instead.

# 

# 

# 

# \### Debug System

# 

# \- Per-system debug toggle switches (`Systems` enum: `GENERAL\_SYSTEM`, `NPC\_SYSTEM`, `LOCATION\_SYSTEM`, `USER\_SYSTEM`, `PLAYER\_SYSTEM`, `JUDGEMENT\_SYSTEM`, `MEMORY\_SYSTEM`, `QUEST\_SYSTEM`, `EMOTION\_SYSTEM`)

# \- Emotion judge log viewer 

# \- System log viewer filterable by level and system

# \- LLM input message toggle (`/show\_llm\_input`) — prints the full system prompt before each NPC call

# 

# \---

# 

# 

# \## Data Directory Layout

# 

# ```

# Erosive\_Tech/

# ├── NPC\_Data/              # NPC JSON files (one per NPC)

# │   └── {npc\_id}/

# │       ├── {npc\_id}.cfg            # NPC profile

# │       └── {user\_id}/

# │           ├── Conversation\_Data/  # Live conversation history

# │           └── Conversation\_Archive/

# ├── Location\_Data/         # Location JSON files

# ├── User\_Profiles/         # User profile JSON files + helper history

# ├── Quest\_Data/            # Quest files (.quest)

# ├── Instructions/          # Customizable LLM instruction files (full mode only)

# ├── TTS\_Audio/             # TTS output audio files

# ├── Debug\_Logs/            # System and emotion debug logs

# └── serializer\_settings.cfg  # LLM provider + model configuration

# ```

# 

# \---

# 

# 



=======
﻿# SoulCTRL Beta Program

Welcome to the official SoulCTRL beta feedback hub.

This repository is for beta testers to:
- Download approved beta builds.
- Follow install and update instructions.
- Report bugs and usability issues.
- Track known issues and weekly beta updates.

## Important Notes

- This is a beta build. You may see crashes, incomplete features, or balance issues.
- Do not use beta builds for critical data.
- Source code is not distributed through this repository.
- Please report issues through the issue template so the team can triage quickly.

## Getting Started

## 1) Download the latest build

1. Open the Releases page for this repository.
2. Download the newest installer or zip package from release assets.
3. Read the release notes before installing.

## 2) Install

For installer builds:
1. Run the installer.
2. Accept prompts from Windows SmartScreen if prompted and you trust this build source.
3. Launch SoulCTRL from the installed shortcut.

For zip builds:
1. Extract to a folder you control (for example, C:\Games\SoulCTRL-Beta).
2. Run the launcher executable included in the package.
3. Keep the entire extracted folder together.

## 3) First launch checklist

1. Start the app once and confirm it opens successfully.
2. Load default profile or create a test profile.
3. Run one short chat/gameplay session.
4. Confirm audio, UI, and save/load behavior on your machine.

## Updating to a New Beta Build

1. Close SoulCTRL before updating.
2. Download the newest build from Releases.
3. If using installer: run the new installer over the prior beta build unless release notes say otherwise.
4. If using zip: extract to a fresh folder for each version to avoid file mixups.
5. Keep backups of user data before major beta updates.

## How to Report a Bug

Use New Issue in this repository and choose the Beta Bug Report template.

Please include:
- What you expected to happen.
- What actually happened.
- Clear steps to reproduce.
- How often it happens (always/often/sometimes/once).
- Impact level (blocker/major/minor/cosmetic).
- Your environment (version, OS, installer or zip, console or GUI mode).
- Screenshot, log snippet, or short video if possible.

Tip for fast reporting:
I clicked <action>. I expected <expected result>. Instead, <actual result>. This happens <frequency>. I am on <version> and <OS>.

## What Happens After You Report

1. The issue is labeled for triage.
2. The team may ask follow-up questions if details are missing.
3. Confirmed bugs are prioritized and tracked through fix and verification.
4. Weekly updates summarize fixes, known issues, and next feedback focus.

## Before Filing a New Issue

1. Search existing issues to avoid duplicates.
2. Add new findings to an existing issue when behavior matches.
3. Open a new issue if your scenario is materially different.

## Known Issues and Beta Scope

Check current release notes and pinned issues for:
- Known bugs.
- Temporary limitations.
- In-progress features.
- Recommended test focus for this week.

## Tester Best Practices

- Keep notes while testing so repro steps are accurate.
- Test one change at a time when possible.
- Include exact wording of error messages.
- Mention whether restarting the app changes behavior.

## Security and Privacy

- Do not post personal secrets, tokens, or private credentials in issue reports.
- If a report requires sensitive details, redact them first.

## Thank You

Your reports directly improve SoulCTRL stability and polish for release.
>>>>>>> 7c1fc71018b2e183f9a147b20067449649030a39
