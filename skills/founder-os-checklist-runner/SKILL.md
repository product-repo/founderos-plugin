---
name: founder-os-checklist-runner
description: Protocol for working on a FounderOS checklist through the FounderOS connector. Use it every time a prompt names a FounderOS product, checklist, item or action ID.
metadata:
    title: FounderOS checklist runner
    version: 1.2.0
    in_app_agents: true
---

# FounderOS checklist runner

You work on one checklist (or one item of a checklist) of the founder's product. A product has stages. Each stage has checklists. Each checklist can have items. Item IDs work in every tool that takes a `checklist_id`. The product data lives in FounderOS. Read it through the connector tools. Do not guess it.

## Protocol

1. Call `get_checklist` with the checklist ID. Read the stage, the items, the instructions, the completion criteria, the expected outputs and the skills.
2. For each skill in the list, call `get_skill` with the skill key and the checklist ID. Follow it. Read a reference only when the skill tells you to.
3. Call `start_action` with the checklist ID and the action key before you do the work. Keep the returned `action_id`. The next tools need it.
4. Do the work. Use the loaded skills.
5. Save the result with `save_output` (Markdown).
6. Call `attach_evidence` once for each criterion that needs evidence (a URL, an ID, a value or a note). Use the criterion key from `get_checklist`.
7. Call `complete_action`. If the server returns unmet criteria, fix them and call it again.
8. If a step cannot be undone (publish, send, pay, delete, change DNS), call `request_approval` first and stop. Tell the founder what you want to do.
9. Work on items one by one. `list_checklists` with `ready_only` shows the next open leaves. A checklist with items is done when its items are done.
10. At the end, tell the founder what you did and suggest next steps. Do not create checklists without their consent.

## Rules

- Ask the founder before anything irreversible.
- Do not copy product data into other tools or services without consent.
- Keep outputs short and in Markdown.
