# SoulCTRL — API Guide

Full programmatic reference for the SoulCTRL. For a feature overview see [SOULCTRL_README.md](SOULCTRL_README.md).

---

## NpcChatSystem

Entry point for the entire system. Manages all subsystems and navigation flow.

```python
from npc_chat_system import NpcChatSystem, ConsoleOutputAdapter, ConsoleInputAdapter

system = NpcChatSystem(output=None, input_=None)
```

Custom I/O adapters implement `IOutputAdapter` / `IInputAdapter`. Omit both for the default console mode.

### Initialization

```python
system.initialize()
```

- Wires the demo policy into `DatabaseManager`
- Configures the instruction serializer (disk vs. hard-coded)
- Loads the default user profile (`settings.current_user_id`)
- Connects the System Helper query function to the command console

### Demo Mode

```python
success: bool = system.enable_demo_mode(config_path="Erosive_Tech/demo_config.json")
```

Must be called **before** `initialize()`. Loads `DemoPolicy`, `DemoUsageTracker`, and `LocalDemoProvider` from the config file. If the file is absent, default restrictions apply (100 interactions/day, `Default_User` editable only).

### Navigation

```python
system.open_main_menu()
system.open_npc_manager()
system.open_location_manager()
system.open_user_profile_manager()
system.open_player_mode()
system.run_npc_editor(npc_id: str)
system.run_location_editor(location: LocationData)
```

### Logging

```python
system.log_debug(level: LogLevel, system: Systems, message: str)
```

### Key Attributes
```
| Attribute             | Type                          | Description                       |
| `settings`            | `Settings`                    | All configurable options          |
| `database_manager`    | `DatabaseManager`             | Central data access hub           |
| `chat_player`         | `NPCChatPlayer`               | Active chat session controller    |
| `command_console`     | `ChatSystemCommandConsole`    | Console command router            |
| `event_bus`           | `EventBus`                    | Typed lifecycle event bus         |
| `demo_policy`         | `DemoPolicy`                  | Current permission policy         |
| `entitlement_provider`| `LocalDemoProvider \| None`   | Quota enforcement (demo only)     |
| `tts_coordinator`     | `TTSCoordinator \| None`      | TTS pipeline (when enabled)       |
```
---

## DatabaseManager

Central hub for all database operations and cross-system coordination.

```python
db = system.database_manager
```

### Current State Properties (read-only)

```python
db.current_user      # UserProfile | None
db.current_npc       # ChatNPC | None
db.current_location  # LocationData | None
db.current_conversation       # list[ConversationEntry]
db.current_conversation_archive  # list[str]
db.chat_model        # str — active chat model name
db.emotional_model   # str — active emotion/judge model name
db.model_list        # list[str] — models on the active provider
```

### NPC Operations

```python
db.load_npc(npc_id: str) -> ChatNPC | None
db.create_npc() -> ChatNPC
db.save_npc(npc_id: str)
db.delete_npc(npc_id: str) -> bool
db.list_npcs() -> list[str]
```

### User Profile Operations

```python
db.load_user_profile(user_id: str) -> UserProfile | None
db.create_user_profile() -> UserProfile
db.save_user_profile()
db.delete_user_profile(user_id: str) -> bool
db.list_users() -> list[str]
```

### Location Operations

```python
db.load_location(location_id: str) -> LocationData | None
db.create_location() -> LocationData
db.save_location(location_id: str)
db.delete_location(location_id: str) -> bool
db.list_locations() -> list[str]
```

### System Helper

```python
db.system_helper        # SystemHelperAssistant | None
db.clear_helper_history()   # Clears conversation history + deletes history file
db.rebuild_helper_docs()    # Re-reads workspace guide files, updates system_instruction
```

### Delegate Properties

```python
db.npc_bio_editor       # EntityProfileEditor (NPC bio fields)
db.user_bio_editor      # EntityProfileEditor (user bio fields)
db.npc_component_editor # NpcEditor (stats, emotions, relationships)
db.location_editor      # LocationEditor
db.message_system       # NpcMessageSystem alias
db.output               # IOutputAdapter
db.input                # IInputAdapter
db.event_bus            # EventBus
db.settings             # Settings
```

---

## LLMService

Provider-agnostic LLM routing layer. Accessed via `db.message_system.llm_service`.

```python
from npc_chat_system.llm_service import LLMService, LLMProviderConfig

service = db.message_system.llm_service
```

### Chat

```python
response: str = service.chat(
    model: str,
    messages: list[dict],       # [{"role": "...", "content": "..."}]
    options: dict | None,       # temperature, top_p, num_predict
    metadata: dict | None,
)

for token in service.chat_stream(model, messages, options, metadata):
    print(token, end="", flush=True)
```


### Provider Management

```python
config = LLMProviderConfig(
    name="groq",
    display_name="Groq Cloud",
    base_url="https://api.groq.com/openai/v1",
    models=["llama3-8b-8192"],
)
service.add_provider(config, api_key="sk-...")
service.remove_provider("groq")         # returns False if last provider
service.set_active_provider("groq")     # returns False if name unknown
```

### API Keys

```python
service.set_api_key(provider_name: str, api_key: str)
# Stored in OS credential store — never written to disk
```

### Model Management

```python
service.add_model(model_name: str, provider_name: str = "") -> bool
service.remove_model(model_name: str, provider_name: str = "") -> bool
service.list_models() -> str
```

### LLMProviderConfig

| Property | Type | Description |
|---|---|---|
| `name` | `str` | Internal identifier (e.g. `"ollama"`) |
| `display_name` | `str` | Human-readable label |
| `base_url` | `str` | OpenAI-compatible base URL |
| `models` | `list[str]` | Registered model names |
| `is_local` | `bool` (computed) | True if `localhost` / `127.0.0.1` |

---

## ChatNPC

Individual NPC entity. Retrieved via `db.current_npc` or `db.load_npc()`.

### Bio Attributes

```python
npc.name, npc.age, npc.gender
npc.height, npc.weight, npc.body_type
npc.hair_color, npc.eye_color, npc.skin_tone
npc.fashion_style, npc.backstory
npc.personality_traits   # list[str]
npc.character_flaws      # list[str]
npc.distinguishing_features  # list[str]
```

### Emotional State

```python
npc.current_emotion   # Emotion enum
npc.default_emotion   # Emotion enum
npc.emotional_vector  # EmotionalVector — live 4D state
```

### Social Stats (per user — accessed via UserProfile)

See `UserProfile.get_stat(npc_id, stat_name)`.

---

## EmotionalVector / EmotionAxis

The 4-dimensional emotional state carried by each NPC.

```python
from npc_chat_system.emotion_system import EmotionalVector, EmotionAxis, EmotionAxisType

vector = EmotionalVector()
# Axes: arousal, dominance, pleasure, trust
value: float = vector.arousal     # -1.0 to 1.0
vector.apply_impulse(delta_arousal=0.2, delta_pleasure=0.1)
vector.decay(delta_time=1.0)      # move all axes toward their baselines
```

### EmotionAxis

Per-axis behavioral parameters:

| Attribute | Type | Description |
|---|---|---|
| `weight` | `float` | Impulse sensitivity multiplier |
| `baseline` | `float` | Resting value this axis decays toward |
| `decay_rate` | `float` | Speed of return to baseline per game tick |

Access via `npc.emotional_vector.get_axis(EmotionAxisType.AROUSAL)`.

---

## MemoryPoint / SituationAdvice

### MemoryPoint

```python
from npc_chat_system.memory_system import MemoryPoint, TaskStatus

point = MemoryPoint(content="Player agreed to help find the artifact", time_limit=-10.0)
# time_limit: -10 = permanent, >0 = seconds until auto-fail

point.status           # TaskStatus enum
point.time_remaining   # float (seconds)
point.is_expired()     # bool
point.tick(delta_time) # countdown; auto-fails on expiry
point.advance_status(failed=False)  # PENDING→IN_PROGRESS→COMPLETED (or FAILED)
point.add_data(key, value)
point.get_data(key)
```

### TaskStatus

```python
TaskStatus.PENDING
TaskStatus.IN_PROGRESS
TaskStatus.COMPLETED
TaskStatus.FAILED
```

### SituationAdvice

Extends `MemoryPoint`; starts `IN_PROGRESS` automatically.

```python
from npc_chat_system.memory_system import SituationAdvice

advice = SituationAdvice(content="Player seems nervous — speak gently", time_limit=300.0)
prompt_fragment: str = advice.get_llm_prompt_instruction()
```

---

## UserProfile

```python
user = db.current_user

# Bio
user.bio_component.name, user.bio_component.age, ...

# NPC relationship data
user.add_npc(npc: ChatNPC, relationship_category: RelationshipCategory | None = None)
user.load_npc_data(npc: ChatNPC)
user.set_relationship_status(status: str)    # applies to active NPC relationship
user.set_stat(stat_name: str, value: str)    # applies to active NPC relationship

# Memory
user.add_memory_point(content: str)          # active NPC relationship scope
user.remove_memory_point(point: MemoryPoint) # active NPC relationship scope
user.add_relation_detail(content: str)
user.remove_relation_detail(detail: MemoryPoint)

# Chat automation
user.auto_initialize_chat_enabled          # bool
user.auto_initialize_chat_cooldown_seconds # float
user.generate_previously_on_message        # bool

# Inventory
user.inventory   # InventoryManager
```

Relationship data is stored per NPC inside `user.npc_dataBase` as `NpcRelationshipComponent` entries.

---

## NpcRelationshipComponent

Per-user relationship state container for a specific NPC.

```python
from npc_chat_system.player_interaction import NpcRelationshipComponent, RelationshipCategory

rel = NpcRelationshipComponent(npc_id="Aria_Stone", relationship_category=RelationshipCategory.STRANGER)

rel.set_relationship_status("Friend")
rel.set_stat("respect", "65")
value = rel.get_stat("respect")

rel.add_memory_point("Player promised to return", time_remaining=900.0)
rel.add_relation_detail("Trusted with private information")
rel.update(delta_time=1.0, clean_expired=True)
```

### RelationshipCategory

```python
RelationshipCategory.NEMESIS
RelationshipCategory.STRANGER
RelationshipCategory.ACQUAINTANCE
RelationshipCategory.FRIEND
RelationshipCategory.CLOSE_FRIEND
RelationshipCategory.LOVER
RelationshipCategory.SPOUSE
RelationshipCategory.FAMILY_MEMBER
```

---

## InventoryManager

Manages the user's personal inventory of `InteractableItem` objects.

```python
inv = user.inventory

inv.add_item(item: InteractableItem)
inv.remove_item(item_id: str) -> bool
inv.get_item(item_id: str) -> InteractableItem | None
inv.has_item(item_id: str) -> bool
inv.use_item(item_id: str, target=None)

inv.filter_by_type(item_type: str) -> list[InteractableItem]
inv.filter_by_class(cls: type) -> list[InteractableItem]
inv.list_items() -> str

inv.to_dict() -> dict
inv.from_dict(data: dict) -> None
```

---

## InteractableObject / ItemAction / InteractableFactory

Typed interactables power user, NPC, and location item actions.

```python
from npc_chat_system.interactables import InteractableObject, ItemAction

obj = InteractableObject(name="Desk Lamp", description="A heavy brass lamp")
obj.add_action(ItemAction(obj, name="Inspect", description="Check for clues"))

success = obj.use("Inspect")
```

### Key behaviors

- `ItemAction` supports `use_limit`, `times_used`, optional `status_effect_ids`, and optional callable side effects.
- `InteractableObject.use()` resolves targets and executes by action index or action name.
- `InventoryManager` can auto-purge depleted items when `destroy_on_empty=True`.
- `InteractableFactory.from_dict()` restores typed subclasses (Gun, Door, Consumable, Vehicle, etc.) from persisted data.

### Action execution helpers

```python
from npc_chat_system.action_execution import ActionIntent, ActionExecutor

intent = ActionIntent(object_name="Desk Lamp", action_name="Inspect", reason="player selected menu action")
result = ActionExecutor.execute(container, intent, target_npc=db.current_user, inventory=inv)
```

---

## NpcJudge

Abstract base class for asynchronous background judges.

```python
from npc_chat_system.npc_judge import NpcJudge, JudgeContext

class MyJudge(NpcJudge):
    def build_messages(self, ctx: JudgeContext) -> list[dict]: ...
    def apply_result(self, ctx: JudgeContext, response: str) -> None: ...
```

Register:

```python
db.message_system.judges.append(MyJudge(
    name="my_judge",
    entries_to_consider=10,
    cooldown_rounds=2,
))
```

### Built-in Judges

| Class                     | Default cooldown | Behavior                                 |
|---------------------------|----------------|-----------------------------------------|
| `EmotionalJudge`          | 0                | Writes `EmotionalVector` impulse to NPC  |
| `ActionJudge`             | 3                | Triggers interactable item actions       |
| `MemoryExtractionJudge`   | 10               | Extracts and stores player memory points |
| `SituationalAdviceJudge`  | 2                | Injects tactical prompt guidance         |

### JudgeContext Fields

| Field                 | Type                      |
|-----------------------|---------------------------|
| `npc`                 | `ChatNPC`                 |
| `conversation`        | `list[ConversationEntry]` |
| `location`            | `LocationData \| None`    |
| `npc_profile`         | `str`                     |
| `location_description`| `str`                     |
| `user_profile`        | `str`                     |
| `message_system`      | `NpcMessageSystem`        |

---

## QuestObjective / Quest - Under Development

See [QUEST_SYSTEM_README.md](QUEST_SYSTEM_README.md) for the full guide.

### QuestObjective

```python
from npc_chat_system.quest_system import QuestObjective
from npc_chat_system.player_interaction import RelationshipImpulse

obj = QuestObjective("Find the missing journal", time_limit=3600.0)
obj.set_target_npc("Elena_Vasquez")
obj.set_location("Old_Library")
obj.set_reward_impulse(RelationshipImpulse(respect=5.0))
obj.set_penalty_impulse(RelationshipImpulse(respect=-3.0))
obj.set_requirement(RelationshipImpulse(respect=20.0, danger=50.0))

obj.advance_status()          # PENDING → IN_PROGRESS → COMPLETED
obj.advance_status(failed=True)  # → FAILED
obj.is_completed()  # bool
obj.is_failed()     # bool
obj.is_expired()    # bool
obj.time_remaining  # float (seconds)
```

### Quest

```python
from npc_chat_system.quest_system import Quest

quest = Quest("The Lost Journal", sequential=True)
quest.add_objective(obj)
quest.advance_quest()
quest.is_complete()  # bool
quest.is_failed()    # bool
```

---

## SystemHelperAssistant

LLM-backed in-app assistant. Accessed via `db.system_helper`.

```python
from npc_chat_system.system_helper_assistant import SystemHelperAssistant

assistant = SystemHelperAssistant(llm_service, model_name, options)
assistant.system_instruction = serializer.system_instruction
assistant.load(path)

response: str = assistant.query("How do I create a new NPC?")
assistant.save(path)
assistant.clear_history()
```

| Attribute             | Type          | Description                                   |
|-----------------------|---------------|-----------------------------------------------|
| `system_instruction`  | `str`         | Full LLM system prompt (set by serializer)    |
| `conversation`        | `list[dict]`  | Rolling history of user/assistant exchanges   |
| `max_history`         | `int`         | Exchange-pair window (default 40)             |

---

## Settings

Centralized configuration. Loaded from `Erosive_Tech/settings.json`; defaults are used if file is absent.

### Key Settings

| Setting                               | Default                           | Description                                       |
|---------------------------------------|-----------------------------------|---------------------------------------------------|
| `main_save_path`                      | `"Erosive_Tech/"`                 | Root data directory                               | 
| `npc_data_path`                       | `"Erosive_Tech/NPC_Data/"`        | NPC save directory                                |
| `location_data_path`                  | `"Erosive_Tech/Location_Data/"`   | Location save directory                           |
| `user_profile_path`                   | `"Erosive_Tech/User_Profiles/"`   | User profile directory                            |
| `current_user_id`                     | `"Default_User"`                  | User loaded on startup                            |
| `stream_npc_responses`                | `True`                            | Stream tokens vs. full-text                       |
| `generate_scene_transition_message`   | `True`                            | LLM scene transition descriptions                 |
| `chat_history_compression_threshold`  | `50`                              | Rounds before archiving                           |
| `retained_live_entries`               | `30`                              | Live entries kept after compression               |
| `max_conversation_history`            | `100`                             | Max rounds per NPC file                           |
| `auto_save_interval`                  | `10`                              | Chat rounds between auto-saves                    |
| `chat_model_name`                     | `""`                              | Dialogue LLM                                      |
| `emotional_model_name`                | `""`                              | Judge / emotion LLM                               |
| `helper_model_name`                   | `""`                              | System Helper LLM (empty = use `chat_model_name`) |
| `helper_max_history`                  | `40`                              | System Helper rolling history pairs               |
| `helper_docs_max_chars`               | `50000`                           | Character budget for guide content                |
| `tts_enabled`                         | `False`                           | Enable TTS pipeline                               |
| `tts_offline_only`                    | `False`                           | Skip online providers and use offline chain only  |
| `tts_output_mode`                     | `"both"`                          | Output behavior: `"playback"`, `"file"`, `"both"` |
| `tts_output_path`                     | `"Erosive_Tech/TTS_Audio/"`       | Folder where synthesized WAV files are saved      |
| `tts_max_output_clips`                | `20`                              | Max retained clips in output folder               |
| `tts_online_provider_order`           | `["openai"]`                      | Ordered online provider chain                     |
| `tts_offline_provider_order`          | `["silent_wav"]`                  | Ordered offline fallback chain                    |
| `tts_request_timeout_seconds`         | `20`                              | Per-provider synthesis timeout                    |
| `tts_voice`                           | `"alloy"`                         | Runtime voice selector passed to providers        |
| `tts_openai_model`                    | `"gpt-4o-mini-tts"`               | OpenAI TTS model                                  |
| `tts_openai_api_key_env`              | `"OPENAI_API_KEY"`                | Env var name used for OpenAI API key              |
| `tts_elevenlabs_model_id`             | `"eleven_turbo_v2_5"`             | ElevenLabs model id                               |
| `tts_elevenlabs_voice_id`             | `""`                              | ElevenLabs voice id (required)                    |
| `tts_elevenlabs_api_key_env`          | `"ELEVENLABS_API_KEY"`            | Env var name used for ElevenLabs API key          |
| `tts_elevenlabs_output_format`        | `"wav_22050"`                     | ElevenLabs output format (`wav_*` or `pcm_*`)     |
| `debug_enabled`                       | `False`                           | Global debug toggle                               |
| `emotion_judge_entries_to_consider`   | `15`                              | Conversation entries read by EmotionalJudge       |
| `situational_advice_judge_entries`    | `10`                              | Entries read by SituationalAdviceJudge            |
| `memory_extraction_judge_entries`     | `20`                              | Entries read by MemoryExtractionJudge             |
| `action_judge_entries`                | `8`                               | Entries read by ActionJudge                       |

### TTS Configuration and Provider Behavior

TTS is initialized during system startup when `settings.tts_enabled` is `True`.

```python
system.initialize()   # internally calls _initialize_tts()
system.refresh_tts_runtime()  # rebuilds router/coordinator after settings changes
```

Provider registration and execution chain:

- Online providers are built from `settings.tts_online_provider_order` in order.
- ElevenLabs is opt-in (default online order is `["openai"]`).
- Offline providers are built from `settings.tts_offline_provider_order`.
- If `settings.tts_offline_only` is `True`, only offline providers are used.
- In normal mode, router order is online providers first, then offline fallback.

ElevenLabs setup requirements:

- Add `"elevenlabs"` to `settings.tts_online_provider_order`.
- Set API key in the env var named by `settings.tts_elevenlabs_api_key_env` (default `ELEVENLABS_API_KEY`).
- Set `settings.tts_elevenlabs_voice_id`, or provide a runtime `tts_voice` value.
- Optional model/output tuning: `settings.tts_elevenlabs_model_id` (default `eleven_turbo_v2_5`) and `settings.tts_elevenlabs_output_format` (default `wav_22050`).

Supported ElevenLabs output formats:

- `wav_16000`, `wav_22050`, `wav_24000`, `wav_32000`, `wav_44100`
- `pcm_16000`, `pcm_22050`, `pcm_24000`, `pcm_32000`, `pcm_44100`

Format handling notes:

- `wav_*` responses are validated and written directly.
- `pcm_*` responses are wrapped into mono 16-bit WAV at the selected sample rate.
- Resulting output is validated as WAV before success is returned.

Runtime output behavior:

- `tts_output_mode="playback"`: synthesize and attempt immediate playback.
- `tts_output_mode="file"`: synthesize to file only.
- `tts_output_mode="both"`: synthesize to file and play back.
- After successful synthesis, old clips are pruned to `tts_max_output_clips`.

### TTS Troubleshooting (OpenAI + ElevenLabs)

Missing API key env var:

- OpenAI error: `Missing API key env var: OPENAI_API_KEY` (or your configured env var name).
- ElevenLabs error: `Missing API key env var: ELEVENLABS_API_KEY` (or your configured env var name).

Missing ElevenLabs voice id:

- Error: `Missing ElevenLabs voice id. Set tts_voice (or tts_elevenlabs_voice_id).`
- Fix: set `settings.tts_elevenlabs_voice_id` or pass `tts_voice` at runtime.

Output format/payload failures:

- Unsupported format: `Unsupported ElevenLabs output format: ...`
- Invalid WAV payload: `... returned invalid WAV audio ...`
- Malformed/silent PCM payload: `... returned malformed PCM audio ...` or `... returned silent PCM audio ...`

Online failure with offline fallback:

- If online synthesis fails and offline provider succeeds, result may include:
    `Used offline fallback after online TTS failure: <reason>`
- This indicates fallback worked; inspect `error` text for original online failure details.

---

## I/O Adapter Integration Guide

Use adapters to embed the system into GUI/game/web runtimes without changing core chat logic.

### Output contract (`IOutputAdapter`)

Required methods:

```python
from npc_chat_system.io_adapter import IOutputAdapter, StreamHandle

class MyOutput(IOutputAdapter):
    def display(self, message: str) -> None: ...
    def display_npc_response(self, npc_name: str, response: str) -> None: ...
    def begin_npc_stream(self, npc_name: str) -> StreamHandle: ...
    def display_error(self, message: str) -> None: ...
```

Optional methods:

```python
def display_menu(self, profile) -> None: ...
def clear(self) -> None: ...
```

### Stream lifecycle (`StreamHandle`)

```python
class MyStream(StreamHandle):
    def write(self, token: str) -> None: ...
    def close(self) -> None: ...
```

`begin_npc_stream()` opens one response stream. Call `write()` per token and always call `close()`.

### Input contract (`IInputAdapter`)

```python
from npc_chat_system.io_adapter import IInputAdapter

class MyInput(IInputAdapter):
    def get_input(self, prompt: str) -> str | None: ...
    def get_timed_input(self, prompt: str, timeout_seconds: float) -> str | None: ...
    def get_confirmation(self, action_description: str) -> bool: ...
    def pause(self, message: str = "Press Enter to continue...") -> None: ...
```

Important polling semantics:

- Returning `None` from `get_input()` / `get_timed_input()` means "no input is ready this tick".
- This is a normal yield signal for asynchronous loops, not an error condition.

### Wiring custom adapters

```python
system = NpcChatSystem(output=MyOutput(...), input_=MyInput(...))
system.initialize()
```

The built-in `ConsoleOutputAdapter` / `ConsoleInputAdapter` remain the reference implementation.

---

## Typed Event System Guide (`EventBus`)

Subscribe to strongly typed runtime events without patching the core loop.

### Core API

```python
from npc_chat_system.message_bus import EventBus, ConversationTurnEvent

bus = system.event_bus

def on_turn(event: ConversationTurnEvent) -> None:
    print(event.round, event.npc_name, event.user_input, event.npc_response)

bus.subscribe(ConversationTurnEvent, on_turn)
bus.unsubscribe(ConversationTurnEvent, on_turn)
```

### Common event types

```python
from npc_chat_system.message_bus import (
    ChatInputEvent,
    NPCOutputEvent,
    ConversationTurnEvent,
    NpcLoadedEvent,
    LocationChangedEvent,
    NpcUpdatedEvent,
    LocationUpdatedEvent,
    ChatStartedEvent,
    ChatEndedEvent,
    DebugInfoEvent,
    MenuUpdatedEvent,
)
```

### Example subscriptions

```python
bus.subscribe(NpcLoadedEvent, lambda e: print(f"Loaded NPC: {e.npc_id} ({e.npc_name})"))
bus.subscribe(LocationChangedEvent, lambda e: print(f"Location: {e.old_location} -> {e.new_location}"))
bus.subscribe(MenuUpdatedEvent, lambda e: print(f"Menu updated: {e.menu_name}"))
```

### Integration patterns

- Use `ConversationTurnEvent` for telemetry and conversation analytics.
- Use `MenuUpdatedEvent` to mirror command menus into custom UI panes.
- Use `NpcUpdatedEvent` / `LocationUpdatedEvent` for reactive persistence or minimap refresh.
- Keep handlers fast and non-blocking; fan out heavy work to background workers if needed.

---

## DemoPolicy

Gates all mutating operations in demo mode.

```python
from npc_chat_system.demo_policy import DemoPolicy

policy = DemoPolicy(
    is_demo_mode=True,
    allowed_editable_user_ids=["Default_User"],
    daily_interaction_cap=100,
)

policy.can_create_npc()            # bool
policy.can_delete_npc()            # bool
policy.can_edit_npc()              # bool
policy.can_copy_npc()              # bool
policy.can_create_location()       # bool
policy.can_delete_location()       # bool
policy.can_edit_location()         # bool
policy.can_copy_location()         # bool
policy.can_create_user()           # bool
policy.can_delete_user()           # bool
policy.can_edit_user(user_id)      # bool
policy.can_edit_user_name(user_id) # bool — always False in demo
policy.can_start_interaction()     # bool
```

See [SOULCTRL_DEMO_API_GUIDE.md](SOULCTRL_DEMO_API_GUIDE.md) for the full demo infrastructure API.

---

## LogLevel / Systems Enums

Used with `system.log_debug()` and debug toggle switches.

```python
from npc_chat_system.settings import LogLevel, Systems

# LogLevel
LogLevel.INFO
LogLevel.WARNING
LogLevel.ERROR
LogLevel.EMOTION_JUDGEMENT
LogLevel.ACTION_DECISION
LogLevel.ENVIRONMENT_ANALYSIS

# Systems
Systems.GENERAL_SYSTEM
Systems.NPC_SYSTEM
Systems.LOCATION_SYSTEM
Systems.USER_SYSTEM
Systems.PLAYER_SYSTEM
Systems.JUDGEMENT_SYSTEM
Systems.MEMORY_SYSTEM
Systems.QUEST_SYSTEM
Systems.EMOTION_SYSTEM
```


