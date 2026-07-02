---
name: pi-handoff
description: Hand the current pi conversation off to a fresh pi agent spawned in a new herdr pane, seeded with a handoff summary as its first prompt.
disable-model-invocation: true
---

Hand the current conversation off so a fresh pi agent can continue the work. Write a handoff summary and launch a new pi seeded with it as the first prompt — spawned in a new herdr pane beside you.

If invoked with arguments (`/skill:pi-handoff <focus>`), treat them as a description of what the next session focuses on and tailor the summary accordingly.

## 1. Guard: confirm you are inside herdr

Check `HERDR_ENV`. If it is not `1`, say you are not running inside a herdr-managed pane and stop — there is nowhere to spawn the handoff.

Done when: `HERDR_ENV=1` is confirmed.

## 2. Write the handoff summary

Follow the `/handoff` skill for what the summary contains and how to write it, with two overrides:

- **It's a transient seed, not a deliverable.** Don't commit it or leave it as a repo artifact. Step 4 writes it to a temp file only to hand it over reliably. Write it addressed to the fresh agent.
- **End with a checkpoint directive:** instruct the agent to confirm scope in a few lines, then stop and wait for the go-ahead before executing. Keep it tight — the goal in one sentence, the immediate first action, and only the decisions or assumptions it actually needs resolved. Do not re-list what's already done or restate the summary back. It must not touch code or run commands until the user approves.

Done when: a fresh agent reading only this summary understands the goal, knows which skills to invoke, and is told to restate scope and wait — and it carries no secrets.

## 3. Spawn a fresh pi in a new pane

Split your current pane, keeping your own focus, and read the new pane id from the response:

```bash
NEW_PANE=$(herdr pane split <your-pane-id> --direction right --no-focus | python3 -c 'import sys,json; print(json.load(sys.stdin)["result"]["pane"]["pane_id"])')
```

Use `herdr pane list` to find your own pane id if needed.

Done when: `NEW_PANE` holds the new pane id.

## 4. Seed the fresh pi with the summary

Do not inline the summary as a quoted argument — a long summary trips shell quoting and often never submits. Write the summary to a temp file and hand pi the file:

```bash
# 1. write the step-2 summary to a temp file, e.g.
TMP=/tmp/pi-handoff-<name>.md   # <name> = your descriptive slug
# ...write the summary into $TMP...

# 2. seed pi with the file (@file includes its contents as the first message)
herdr pane run $NEW_PANE "pi -n '<descriptive name>' @$TMP"
```

`@<file>` includes the file's contents as pi's first message with no escaping and no giant paste. Set `-n`/`--name` to a short descriptive name so you can tell handoff agents apart in pi's footer.

If a previous attempt left the pane stuck at a prompt, reset it before retrying: send Ctrl-C twice with `herdr pane send-keys $NEW_PANE`, then re-run.

Done when: pi launched in the pane with the summary file as its prompt.

## 5. Confirm and report

Wait for the fresh pi to pick up the prompt, then report:

```bash
herdr wait agent-status $NEW_PANE --status working --timeout 20000
```

Reaching `working` proves pi launched and accepted the seeded prompt. If it times out, read the pane (`herdr pane read $NEW_PANE --source recent`) and report the failure — leave the temp file in place for a retry. On success, delete the temp file (`rm -f $TMP`), then tell the user the new pane id and that the agent will restate scope and wait for approval at the checkpoint; they focus it with `herdr pane focus $NEW_PANE` to review and give the go-ahead. Do not focus it yourself.

Done when: the user has the pane id and its confirmed state (launched-and-awaiting-approval, or failed).
