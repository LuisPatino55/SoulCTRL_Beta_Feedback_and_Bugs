# SoulCTRL

A Python framework for creating emotionally intelligent NPC interactions powered by a local or cloud LLM. Ships with [Ollama](https://ollama.com) as the default offline provider; any OpenAI-compatible API works as a drop-in replacement with no code changes.

---

## Requirements

- Python 3.12+
- [Ollama](https://ollama.com/docs/installation) running locally *(default provider; swap to any cloud API via `/manage` or `LLM ServiceModels Manager` button in the webGUI interface)*
- Python packages: `openai`, `keyring`, `colorama`
- Optional browser GUI packages: `fastapi`, `uvicorn`

## Windows Installer (Recommended First Run)

If Python 3.12+ is already installed, run:

```bat
install_chat_system.bat
```

Optional flags:

```bat
install_chat_system.bat -ModelName qwen2.5 -NoLaunch
```

SoulCTRL Installer [Download v0.1.0-beta.2](https://www.dropbox.com/scl/fo/fqy5i6z36uuplym34m5bq/AHicFlixUmfE41ZY58Q2o_I?rlkey=d117cw50bbkph8oymdqhlgnlz&st=ry4qrl4o&dl=0) Must have folder access to download.

Video walkthrough: [SoulCTRL Installer Walkthrough](https://youtu.be/lRgAWQ3jsT0)

Scenario Item Demo: [SoulCTRL 1st Date Scenario](https://youtu.be/HU9s1x1RNrg)


What the installer does:

- Creates `.venv`
- Installs package dependencies with GUI extras
- Attempts to install Ollama automatically (best effort)
- Prompts for your default model name and saves it as chat and emotion model
- Adds that model name to the Ollama provider model list (no validation)
- Attempts `ollama pull <model>` and reports if pull started; otherwise tells you to pull manually
- Launches `run_chat_gui.py`
- Creates `launcher.bat` and a `Launch SoulCTRL.lnk` shortcut in the project folder

After installation, non-technical users can start the app by double-clicking `Launch SoulCTRL.lnk`
or `launcher.bat`, which opens a small window to choose **Console** or **GUI** mode.

Notes:

- Python is still a prerequisite and is not installed by this script.
- If Ollama auto-install is unavailable on your machine, install it manually from https://ollama.com/download and rerun.

```bash
pip install openai keyring colorama
ollama serve
```

Browser GUI optional install (Highly recommended for best experience):
```bash
pip install .[gui]
```

---

## Quick Start


### Console mode
```bash
run_chat.bat
```


### Programmatic / No UI mode (headless)

```python
from npc_chat_system import NpcChatSystem, ConsoleOutputAdapter, ConsoleInputAdapter

system = NpcChatSystem()
system.initialize()
db = system.database_manager
db.load_user_profile("Default_User")
db.load_npc("Aria_Stone")
db.load_location("Downtown_Cafe")
system.open_player_mode()
```


### Browser GUI mode (Best experience)

```bash
run_chat_gui.bat
```

These launch wrappers run with the installer-created `.venv` so dependencies stay consistent.

Optional flags:

- `--host 127.0.0.1`
- `--port 8765`
- `--no-browser`

---

## Features


### Conversation System

- LLM-powered NPC dialogue & story creation via `SoulCTRL` + `LLMService`
- Streaming responses (token-by-token) or full-text mode (`Set in the Chat Player Menu`)
- Multi-layered NPC personality `judge components`, create more nuanced behaviour, responses, and actions
- Asynchronous `judge component` scheduler — judges run in background threads after each NPC reply; chat input stays responsive
- Deferred `judge decision` application — results are injected into the *next* system prompt so no turn is blocked and the NPC can respond immediately to component decisions
- Dynamic and highly configurable emotions, relationships, behaviour, stats, and stat definitions
- Automatic memory collection, with manual overrides and full edit control
- Interactable item system that the NPC uses autonomously to trigger actions (eat, drink, change locations, use weapons, charge and pay currency, etc.)
- Conversation archiving: live history compresses to archive chunks when it exceeds `chat_history_compression_threshold`; past context is summarized by the LLM and kept as a rolling archive
- Multiple user profiles with per-user & per NPC (memory, relationship stats, and conversation history)


### LLM Model Recommendations

* Context Window Size (Minimum: 32K - Recommended: 128K)
  - Recommended minimum context size (from Ollma settings) is 32K but you will need to be conservative with memory and conversation history to avoid hitting the context limit of your model. Use the biggest context you can run on your system for best results (developed with 128k). The system will automatically compress conversation history into a summary when it exceeds the `chat_history_compression_threshold` setting. You will see chat quality degrade if your context size is too small.

* Ollama Model Recommendations
  - For chat model, using a larger model is recommended for best quality (depending on your system capabilities). Choose a model that is abliterated or uncensored and is good at conversation / roleplay. 
    - Recommended chat models I tested: (Highest Resource Demand)
      - `DaddyLLAMA/magum_v2_123b_iq3_xxs:latest` (Ollama) 
      - `nchapman/l3.3-70b-euryale-v2.3:70b` (Ollama) 
    - Recommended chat models I tested: (Medium Resource Demand)
      - `huihui_ai/qwen3.5-abliterated:35b` (Ollama) 
      - `vaultbox/qwen3.5-uncensored:35b` (Ollama)
    - Recommended chat models I tested: (Lowest Resource Demand - These get kinda sketch)
      - `HammerAI/llama-3-lexi-uncensored:8b-q8_0` (Ollama) 
      - `HammerAI/mistral-nemo-uncensored:12b-q4_0` (Ollama)

  - For emotion model, using lighter and `instruction` models is recommended for best performance. The best emotion model I have tested so far that is light and works with 95% success is:
    `baytout3/Qwen3.5-Uncensored-HauhauCS-Aggressive:9b` (Ollama), but of course, you can do your own testing. 

* Model Switching
  - If switching between different chat and emotion models, is too taxing on your system, you can leave the emotion model blank and the chat model will be used for both. The system is designed to efficiently switch between models at an optimal time, but if you run into performance issues, you can leave the emotion model blank to use the chat model for both. However, you may see more `judge component` failures in the debug logs if not using a specific `instruction` model.



### NPC Profiles

Each NPC (`ChatNPC`) carries:

| Component                   | Contents                                                                      |
|-----------------------------|-------------------------------------------------------------------------------|
| **Bio**                     | Name, age, gender, height, weight, build, hair/eye color, skin tone, fashion  |
| **Backstory**               | Freeform history string injected into the system prompt                       |
| **Personality traits**      | Positive traits — most influential prompt input                               |
| **Character flaws**         | Negative traits                                                               |
| **Distinguishing features** | Physical details                                                              |
| **Emotion component**       | `EmotionalVector` + per-axis `EmotionAxis` settings                           |
| **Stats**                   | Social stats per user: Respect, Attraction, Desire, Danger, Hunger, Energy    |
| **Relationship category**   | Stranger → Acquaintance → Friend → Close Friend → Romantic Partner, etc.      |
| **Memory points**           | Named `MemoryPoint` entries attached to the NPC–user pair                     |
| **Situation advice**        | Active `SituationAdvice` entries injected into the next prompt                |
| **Inventory & Currency**    | `Item` storage, NPC `self-item use`, `currency` trading system                |
| **Health**                  | `Health` component affected by items and other factors                        |
| **Keychain**                | NPC-owned keys for unlocking doors, vehicles, or other interactables          |



### 4-Dimensional Emotion Model

Availabe Emotions: 
- `NEUTRAL` - **EmotionalVector**(0.0, 0.0, 0.0, 0.0)
- `HAPPY` - **EmotionalVector**(0.5, 0.5, 0.5, 0.5)
- `ELATED` - **EmotionalVector**(0.8, 0.5, 1.0, 0.5)

- `SURPRISED` - **EmotionalVector**(0.7, 0.0, 0.0, 0.0)
- `EXCITED` - **EmotionalVector**(0.7, 0.4, 0.9, 0.5)

- `SAD` - **EmotionalVector**(-0.3, -0.5, -0.3, -0.3)
- `DISGUSTED` - **EmotionalVector**(-0.5, -0.0, -0.7, -0.6)
- `JEALOUS` - **EmotionalVector**(-0.7, 0.6, -0.7, -0.6)

- `ANNOYED` - **EmotionalVector**(0.1, 0.4, -0.5, 0.0)
- `ANGRY` - **EmotionalVector**(0.9, 0.8, -1.0, -1.0)

- `LOVING` - **EmotionalVector**(0.6, 0.6, 0.8, 1.0)
- `ROMANTIC` - **EmotionalVector**(0.8, 0.3, 1.0, 0.5)
- `LUSTFUL` - **EmotionalVector**(1.0, 1.0, 1.0, 1.0)

- `SICK` - **EmotionalVector**(-0.7, -0.5, -0.6, -0.4)
- `FEARFUL` - **EmotionalVector**(-0.8, -0.7, -1.0, -0.7)
- `PANICKED` - **EmotionalVector**(-1.0, -1.0, -1.0, -1.0)


NPCs maintain an `EmotionalVector` with four axes:
| Axis          | Description                                     |
|---------------|-------------------------------------------------|
| Arousal  | Activation level (angered ↔ excited)            |
| Dominance | Perceived social power (submissive ↔ dominant)  |
| Pleasure  | Hedonic valence (negative ↔ positive)           |
| Trust     | Openness (guarded ↔ open)                       |

Each axis has an individual `EmotionAxis` config:
- `weight`      — impulse sensitivity multiplier
- `baseline`    — resting value the axis decays toward
- `decay_rate`  — speed of return to baseline per game tick

Each axis also defines one Personality Trait for the NPC, based on that axis' baseline value. You can find these definitions
at: `Erosive_Tech/Instructions/Personality/` and the approriate folder for each axis. These files are created in the directory
when they do not exist, and can be edited by you to change the personality trait definitions for each axis. If you wish to reset to a default definition, you can delete that file and it will be recreated on next run. The default folder structure is: 
- `Aggression/` - Arousal definitions  
- `Assertiveness/` - Dominance definitions
- `Openness/` - Trust definitions
- `SocialEnergy/`- Pleasure definitions 


The `EmotionalJudge` runs after each turn and writes an `EmotionalVector` impulse back to the NPC automatically. Which 
is based on the emotion response that this NPC would feel, when receiving the last user input. The impulse is then applied to the NPC's `EmotionalVector` and the next system prompt is updated with the new values (multiplied by the Impulse Multiplier setting in the Emotion Editor) and then these values are decayed, applying the decay rate for each axis. 

The `EmotionalJudge` can be disabled in the settings, or in the webGUI Main Page, if you wish to not have this feature enabled.



### Judge Component System

Four built-in asynchronous judges, each configurable with `cooldown_rounds`:

| Judge                   | Default cooldown      | Purpose                                                 |
|-------------------------|----------------------|---------------------------------------------------------|
| `MemoryExtractionJudge` | 10 rounds             | Extracts memorable facts about the interactions         |
| `SituationalAdviceJudge`| 1 rounds              | Detects social cues and provides advice to the NPC      |
| `ActionJudge`           | 2 round               | Triggers item use & other available actions by the NPC  |
| `EmotionalJudge`        | 0 rounds (every turn) | Applies impulses to the NPC (affecting current emotion) |

Cooldowns are configurable at runtime via `/judge_cd [name] [rounds]` in the Chat Player.
Settings also available on the webGUI Main Page, Welcome Panel.

Custom judges can be added by subclassing `NpcJudge` and appending to `npc_message_system.judges`.

If you run into performance issues with judges, you can disable them in the main panel. I recommend using the `EmotionalJudge` at minimum, as it is the most important for driving the emotional state of the NPC. The next most important in my opinion is the `MemoryExtractionJudge`, as it allows the NPC to remember things about the user and the conversation.

- While I feel the `ActionJudge` is a lot of fun, it can be disabled if you wish to have the NPC not use items or trigger actions. 


### Memory System

- **`RelationshipDetails`** — tracks per-user per-NPC goals and facts about the state of the relationship with the user
- **`MemoryPoint`** — a tracked item with a `TaskStatus` lifecycle: `PENDING → IN_PROGRESS → COMPLETED / FAILED`
- Time-limited points count down via `tick(delta)` and auto-fail on expiry
- **`SituationAdvice`** — extends `MemoryPoint`; starts `IN_PROGRESS` and produces an LLM prompt fragment via `get_llm_prompt_instruction()`
- Memory points are stored per user-NPC pair and persist with the user profile
- Relationship details and memory points are both updated on ticks and can auto-expire when their timers reach zero
- Permanent entries use `time_remaining <= -10` and are never expired by the update loop



### Location System

- Locations carry a name, description, atmosphere, time-of-day, and danger level (0–1 Percentage)
- Time and danger are read by the `NpcMessageSystem` each turn and included in the NPC system prompt
- **Scene transitions** move the NPC to a new location mid-conversation with an optional LLM-generated transition message
- **Interactable items** can be attached; the `ActionJudge` can trigger item actions based on conversation context



### Interactable Items & Inventory

- Typed interactable items: `Gun`, `Door`, `Vehicle`, `Consumable`, and more (see `interactable_items.py`)
- **User inventory** (`InventoryManager`) — items owned by the player, independent of location items; accessible in chat via `/items` on in the `user info panel` in the webGUI
- **NPC inventory** — NPC-owned interactables can be triggered by the player via `/use_item`, by right-clicking the NPC in the webGUI, or by the `ActionJudge` automatically
- **Location inventory** — scene interactables can be browsed/triggered via `/loc_items`, by right-clicking the location in the webGUI, or by the `ActionJudge` automatically (if the NPC is using the item)
- **Transfer flow** — `/give` & `Give NPC Item Button` moves selected user items to the active NPC inventory and creates a `MemoryPoint` for the NPC to remember the gift; `/use_item` triggers an action and creates a `MemoryPoint` for the NPC to be aware they used the item
- **Consumable catalog** (`consumable_items.py`) — prebuilt consumables tagged by type: `food`, `drink`, `drug`, `lotion`, `medicine`, etc. Each has a `use()` action that applies effects to the NPC or user, including stat deltas, relationship impulses, and status effects
- `InteractableFactory` restores typed items from save data and supports type-filtered queries
- Action execution supports key-based item lookup, target resolution, status-effect IDs, and use-limit depletion handling


### Status Effects

- **ID-driven registry** — define named effects once, reference by ID anywhere
- Three duration types: `Instant` (one-shot), `Timed` (counts down), `Permanent` (never expires)
- Three payload channels:
  - `EmotionalVector` impulse
  - `RelationshipImpulse` (stat delta applied to NPC relationship)
  - `StatManager` delta (e.g. Health)
- Universal cooldowns — even Instant effects can have a reuse cooldown
- Re-applying an active Timed effect refreshes its duration instead of stacking
- Active effects save and restore with the NPC profile (`effect_id + remaining_time`)


### User Profiles

- Stores preferences, active NPC/location selections, relationship history, and memory per NPC
- `auto_initialize_chat_enabled` / `auto_initialize_chat_cooldown_seconds` — optional idle-triggered NPC follow-up
- `generate_previously_on_message` — toggle "Previously On…" conversation summary on resume
- `generate_scene_transition_message` — toggle LLM-generated scene transition messages
- Includes a `StatManager` with a default `Health` stat (100)
- Per-user per-NPC relationship data: stats, category, memory points, conversation history


### LLM Service

- Provider-agnostic routing via `LLMService` + `LLMProviderConfig`
- Default provider: local Ollama at `http://localhost:11434/v1`
- Switch to any OpenAI-compatible cloud API via `/manage` — no code changes required
- API keys stored in the OS credential store (Windows Credential Manager / macOS Keychain / Linux Secret Service) — **never written to disk**
- Separate model slots: `chat_model_name` (dialogue & helper), `emotional_model_name` (judges) . For best performance, use a smaller (instruct) model for the emotional model and a larger model for the chat model. The system is designed to efficiently switch between models at an optimal time, but if you run into performance issues, you can leave the emotion model blank to use the chat model for both. 



### System Helper (`/?` and `/help`)

An LLM-backed in-app assistant that answers questions about navigation and usage:

- Access quick-help with `/?` or `/? <question>` from the main menu
- Start full interactive helper session with `/help` (plain text questions accepted without a slash)
- In `/help` session mode: `/main` returns to the main menu and `/quit` exits the app
- Conversation history is persisted per user as `{user_id}_helper_history.json`
- Guide content is loaded from `SOULCTRL_HELPER_GUIDE.md` (or the workspace READMEs as fallback) and injected into the assistant's system prompt
- `/hist_clr` — clear helper conversation history for the current user
- `/reset` — reload helper guide content from workspace documents



### I/O and Event Integration

The core runtime supports custom frontends through adapter interfaces plus a typed event bus.

- **I/O adapters** (`IOutputAdapter`, `IInputAdapter`) let you run in console, GUI, or headless server mode.
- Input adapters may return `None` from `get_input()` / `get_timed_input()` to indicate "no input ready this tick" (yield semantics for polling loops).
- Streaming responses are handled through `begin_npc_stream()` and `StreamHandle.write()/close()`.
- **Event bus** (`EventBus`) publishes strongly typed lifecycle events such as `ConversationTurnEvent`, `NpcLoadedEvent`, `LocationChangedEvent`, and `MenuUpdatedEvent`.
- Use event subscriptions for UI refresh, analytics, telemetry, gameplay hooks, and cross-system automation without patching core loop code.
- See `SOULCTRL_API_GUIDE.md` for full integration contracts and examples.



### Text-to-Speech (TTS) Supports OpenAI, ElevenLabs, and local fallback.

- Optional TTS for NPC responses (`settings.tts_enabled`)
- Default online provider order is OpenAI (`settings.tts_online_provider_order = ["openai"]`)
- ElevenLabs is opt-in: add `"elevenlabs"` to `settings.tts_online_provider_order`
- ElevenLabs requires a configured voice id (`settings.tts_elevenlabs_voice_id` or runtime `tts_voice`)
- ElevenLabs output formats: `wav_16000`, `wav_22050`, `wav_24000`, `wav_32000`, `wav_44100`, `pcm_16000`, `pcm_22050`, `pcm_24000`, `pcm_32000`, `pcm_44100`
- Output modes: `"playback"`, `"file"`, or `"both"`
- Audio files saved to `Erosive_Tech/TTS_Audio/`
- Output retention keeps the latest `settings.tts_max_output_clips` files (default `20`)
- Offline-only mode is available via `settings.tts_offline_only` (uses silent WAV fallback unless replaced)



### Instruction File Customization (Full Mode)

In full mode (`allow_disk_overrides=True`) the system exports default LLM instruction definitions to:

```
Erosive_Tech/Instructions/
├── Stats/               # Per-stat behavioral guidance for the LLM
├── Personality/         # Personality trait definitions
├── Emotions/            # Emotion axis definitions
└── System_Helper/       # System helper prompt and guide content
    ├── system_instruction.cfg
    └── guide_content.cfg
```

Edit these `.cfg` files to change how the LLM interprets NPC stats and behaviors without touching source code. In demo mode these files are never written or read — hard-coded in-memory defaults are used instead.



### Debug System

- Per-system debug toggle switches (`Systems` enum: `GENERAL_SYSTEM`, `NPC_SYSTEM`, `LOCATION_SYSTEM`, `USER_SYSTEM`, `PLAYER_SYSTEM`, `JUDGEMENT_SYSTEM`, `MEMORY_SYSTEM`, `QUEST_SYSTEM`, `EMOTION_SYSTEM`)
- Emotion judge log viewer 
- System log viewer filterable by level and system
- LLM input message toggle (`/show_llm_input`) — prints the full system prompt before each NPC call

---


## Data Directory Layout

```
Erosive_Tech/
├── NPC_Data/              # NPC JSON files (one per NPC)
│   └── {npc_id}/
│       ├── {npc_id}.cfg            # NPC profile
│       └── {user_id}/
│           ├── Conversation_Data/  # Live conversation history
│           └── Conversation_Archive/
├── Location_Data/         # Location JSON files
├── User_Profiles/         # User profile JSON files + helper history
├── Quest_Data/            # Quest files (.quest)
├── Instructions/          # Customizable LLM instruction files (full mode only)
├── TTS_Audio/             # TTS output audio files
├── Debug_Logs/            # System and emotion debug logs
└── serializer_settings.cfg  # LLM provider + model configuration
```

---


