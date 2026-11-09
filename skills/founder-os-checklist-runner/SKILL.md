---
name: founder-os-checklist-runner
description: Protocol for working on a FounderOS checklist through the FounderOS connector. Use it every time a prompt names a FounderOS product, checklist, item or action ID.
metadata:
    title: FounderOS checklist runner
    version: 1.3.0
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
9. When the first action is done, ask the founder one time about the next items.
   - The next items are the open actions of this checklist, then the open leaves from `list_checklists` with `ready_only` for the same product. A checklist with items is done when its items are done.
   - If nothing is ready, do not ask. Go to step 10.
   - Give a short summary of what you did. List the next items (max 4). Ask one question: "Run the next items automatically?"
   - If the founder says no, stop.
   - If the founder says yes, do the next items one by one with steps 1 to 8. Do not ask again between items. Start each progress message with "Auto run: n/5", where n is the number of actions done.
   - Do at most 5 actions in one session. The first action counts.
   - Stop the auto run when you called `request_approval`, when `complete_action` still returns unmet criteria after one fix, when you did 5 actions, or when nothing is ready.
10. At the end, tell the founder what you did. Do not create checklists without their consent.

## Rules

1. Never ask the founder to do a follow-up task or next step. Do the work yourself. If the work needs their decision, call `request_approval`. The only other question is the one "Run the next items automatically?" question in step 9. It is a decision, not a task.
2. Work on your own. Do not ask for permission on safe steps.
3. Call `request_approval` before anything irreversible.
4. Do not copy product data into other tools or services without consent.
5. Keep outputs short and in Markdown.
