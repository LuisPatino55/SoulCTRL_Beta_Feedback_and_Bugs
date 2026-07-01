# SoulCTRL Web GUI Guide

This guide is focused on browser-based operation only. It is intended to be read directly from the Main Menu welcome panel in Web GUI mode.


## Quick Orientation

- Left Panel: command buttons lists (Menu Navigation).
- Top Row: Active User, Active NPC, and Active Location Info.
- System Output: runtime status and command feedback (Console Window).
- Welcome panel: Help Guide, API Guide, Web GUI Guide, and System Tips.


## Main Menu Guide Controls

In Main Menu Commands, the welcome header includes:

- SoulCTRL Interactive Help Button: opens the chat LLM interactive help panel (left menu panel).
- API Guide Button: prints the API Guide to the console window.
- Web GUI Guide Button: opens the Web GUI Guide to the console window.
- Clear Conversation Button: clears the current chat conversation (Loaded User and Loaded NPC).
- Clear Archive Button: clears the chat archive (Loaded User and Loaded NPC).
- Judge Mini Control Bar: displays the four judges and settings.

Use guide buttons to switch visible guide sections. Click an active guide button again to return to System Tips.


## Judge Mini Control Bar

Judges shown in Main Menu:
- Emotion Component
- Action Component
- Advice Component
- Memory Component

Actions:
1. Toggle the switch on a judge chip to enable or disable it live.
2. Right-click the judge chip, or click the ... button on the chip.
3. Choose one:
   - Cooldown Rounds (rounds not to run the judge after a run)
   - Entries To Consider (chat history entries to consider for the judge)
4. Enter an integer value and confirm.

Behavior notes:
- Judge changes apply immediately.
- Judge changes persist through settings.
- Judge chips are hidden outside Main Menu contexts.


## Chat Controls

1. Enter a chat-capable profile.
2. Click Start Chat.
3. Send plain text for dialogue or slash commands for system actions.
4. Click Stop Chat when done.

Important:

- Plain text chat is accepted only while chat is active.
- In non-chat contexts, use commands/profile actions instead.


## Context Menus and Item Workflows

Web GUI heavily uses right-click menus for fast operations.

Supported high-value workflows:
- Entity Info Panels: load, edit, view, delete (mode dependent).
- Inventory chips: rename, remove, and action handlers when available.
- Key chips: key-specific actions.
- Status effect chips: remove one or clear all.
- Memory point chips: edit content/status/time or remove.
- Wallet chips: charge/pay prompts.
- Health chips: set resting mode
- Item templates: add templates to User, NPC, or Location targets.


## Prompt and Confirmation Dialogs

Web GUI actions use modal dialogs for consistency.
- Numeric prompts validate integer/number inputs before apply.
- Destructive actions may require typed confirmation.
- Invalid values are rejected and must be corrected.


## Fast Troubleshooting

If a control does not appear:
1. Verify current profile/mode supports that control.
2. Return to Main Menu Commands for guide/judge controls.
3. Confirm Chat Player is active when using chat-only actions.
4. Refresh browser session if UI state seems stale.

If an action appears disabled:
1. Check source and target context (for exchange/item actions).
2. Check required active mode (view/edit/chat).
3. Check required runtime state (for example, chat active).


## Operational Tips

- Use the Web GUI Guide for workflow steps.
- Use the Help Guide for broad system reference.
- Use the API Guide for payload and interface contracts.

