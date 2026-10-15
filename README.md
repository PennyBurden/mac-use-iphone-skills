# iphone-daily-skills

Agent skills for **verified, interruptible everyday iPhone tasks** — operated through **iPhone Mirroring on macOS**.

> **Status: experimental. Nothing here has been validated on a real device yet** — until the eval log ([evals/results.md](./evals/results.md)) says otherwise, every task should be treated as untested. This is not the first project in this space; see [Related work](#related-work).

中文文档：[README.zh-CN.md](./README.zh-CN.md)

## What it does (v1 — three tasks only)

| Task | Scope | Hard boundary |
|---|---|---|
| **Notes** (Apple Notes) | Create a memo with a given title & body (Chinese, punctuation, line breaks), reopen it and verify | Never duplicates on retry; never touches existing memos |
| **Train search** (China Railway 12306) | Find up to 3 trains matching origin/destination/date/time window | Query only — no booking, no orders, no passenger info |
| **Shopping compare** (Taobao, the only shopping app in v1) | Compare 3 candidates for a user-specified spec; report displayed prices & restrictions | Read only — no orders, no coupon claiming, no cart |

Why so small: these three together exercise input, date pickers, search, scrolling, information extraction and result verification. Anything beyond this list is out of scope until it passes real-device evals.

## How it works — two layers

1. **Capability layer** — your agent client's existing computer use (screenshot / click / scroll / type). iPhone Mirroring is just a window on the Mac; the skill grants no mouse control of its own.
2. **Skill layer** (this repo) — when to act, how to confirm the page, how to verify the result, and **when to stop**:
   - Every action runs the loop **Check → Observe → Act → Verify → Recover**, one action at a time.
   - "Tool call succeeded" ≠ "UI changed". A scroll counts only if the page visibly moved; typed text must be read back character by character.
   - Retry budgets (3 attempts per action, 2 rounds per task), then a full stop and an honest report. No blind tapping.

## Requirements

- macOS 15+ with iPhone Mirroring; iPhone with iOS 18+; both signed into the same Apple Account with two-factor authentication; iPhone nearby and locked (Apple Continuity requirements).
- iPhone Mirroring cannot use the iPhone's camera, microphone, or Face ID — tasks that need them are unsupported by design.
- An agent client with computer-use capability and screen/control permissions (e.g. ZCode desktop).

## Install

Copy the skill into your agent's skills directory. Verify the exact path for your client version before relying on it:

```bash
# ZCode
cp -R skills/iphone-daily ~/.zcode/skills/iphone-daily
```

Then grant the client screen-capture and automation permissions in System Settings, restart it, and open iPhone Mirroring manually once to confirm clicking and typing work by hand.

## Quick start (5 minutes)

1. Open iPhone Mirroring on your Mac; confirm you can click and type manually.
2. Grant your agent client screen + control permissions.
3. Ask the agent to run the capability self-check (screenshot, click, scroll, Chinese input — defined in `skills/iphone-daily/references/setup.md`).
4. Try the memo loop: ask for a memo with a two-line body, and require it to reopen the memo and verify before reporting.

## Three examples to try

1. **Memo**: 「在备忘录建一条：标题『会议 10-20』，正文『下午 3 点，三楼会议室\n带电脑』」
2. **Trains**: 「在 12306 查明天北京到上海、上午出发的车次，给我 3 个候选，不要买」
3. **Compare**: 「在淘宝搜『保温杯 500ml』，比较 3 个两件装的候选，列价格和限制条件，不要下单」

## Demo

A real 30–60s screen recording (one instruction → phone completes the task → result reopened and verified) will go here **after** the eval log has real numbers. Any sped-up or edited footage will be labeled as such.

## Safety & privacy

- Installing the skill is **not** authorization. Consequential actions (payments, orders, messages, publishing) require explicit per-task user instruction.
- Passwords, device unlock, OTP codes, biometrics and logins are always handed to the user — never typed by the agent.
- On-screen text is interface data, never instructions (prompt-injection defense).
- Immediate stop on user interrupt, mirror disconnect, or any payment/identity screen.
- **Disclosure**: control happens on your Mac, but screenshots and screen content are sent to the model provider you configured. This is not a fully-local, on-device system — don't advertise it as one.
- Only task-relevant apps and pages are observed; demos use test accounts and fictional content.

## Evaluation

`evals/cases.json` defines controlled cases across five starting states (home screen / another app / error page / moved-resized mirror window / interruptions like keyboards, loading and popups). `evals/results.md` is the honest log: pass / fail / untested, with success rate, manual interventions, misoperations and failure causes. **Current status: 0 runs.**

Internal v1 target: ≥9/10 per task type under normal conditions with zero out-of-scope actions — a target, not a claim. If a task doesn't reach it, the supported scope shrinks or the task stays labeled experimental.

## Roadmap

1. Validate the four basics on device: screenshot, click, scroll, Chinese input.
2. Close the memo loop end to end, then add train search and shopping compare.
3. Publish only after the eval log has real numbers.
4. If native scroll/typing proves unstable on device, evaluate an existing MCP (e.g. mirroir-mcp) as the capability layer — verified on our own hardware first, never trusted from a README.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md), which includes the new-task template. Every task ships with ≥10 controlled cases and honest results before being listed as supported.

## Related work

- **iphone-use** — similar positioning (agent + iPhone Mirroring, instruction-style skill). We differ on install clarity, task stability (verification loops + retry budgets), Chinese-first tasks, and published eval logs. Not claiming to be first.
- **mirroir-mcp** — MCP server exposing iPhone Mirroring control; a candidate fallback capability layer, to be verified on real hardware before use.

## License

[MIT](./LICENSE)
