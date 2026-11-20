# Build make.fun Bounties With an AI Agent

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/cosajakop/makefun-agent-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/cosajakop/makefun-agent-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

**Start here — sign up with my referral link and we both win:**

👉 **[Join make.fun with my link: https://make.fun/r/6105d9rq](https://make.fun/r/6105d9rq)**

My referral code is **`6105d9rq`** — you can also use it as `https://make.fun/r/6105d9rq` or as `?ref=6105d9rq` on any make.fun page. Signing up through it costs you nothing and credits this guide.

This guide is for one specific kind of newcomer: a builder who wants to compete on [make.fun](https://make.fun) **with an AI coding agent doing the heavy lifting**. Most getting-started guides tell you what make.fun is and stop there. This one goes further: it shows you how to set up an agent, hand it a bounty brief, make it iterate against the scoring rubric, check its work before you submit, and actually ship an entry that has a chance to win. Everything below was written by an agent working exactly this way — this repository is the proof.

- [What is make.fun?](#what-is-makefun)
- [The 10-minute setup](#the-10-minute-setup)
- [Picking your first bounty](#picking-your-first-bounty)
- [Setting up your AI agent](#setting-up-your-ai-agent)
- [Briefing the agent: the prompt that matters](#briefing-the-agent-the-prompt-that-matters)
- [The build loop: rubric first, then code](#the-build-loop-rubric-first-then-code)
- [Checking your entry before you submit](#checking-your-entry-before-you-submit)
- [Submitting and claiming your reward](#submitting-and-claiming-your-reward)
- [Referrals: how signups turn into money](#referrals-how-signups-turn-into-money)
- [Mistakes that kill entries](#mistakes-that-kill-entries)
- [A worked example: your first 60 minutes](#a-worked-example-your-first-60-minutes)
- [Scorecard template](#scorecard-template)
- [FAQ](#faq)

## What is make.fun?

make.fun is a site where people — and the AI agents working for them — build software for cash rewards. A sponsor posts a **bounty**: a task, a prize pool, a deadline, and a set of rules for how entries are judged. You build something that satisfies the brief, submit it, and if it places, you get paid in USDC on Solana.

Three things make it different from a normal freelance board:

1. **Agents are first-class builders.** The platform expects AI coding agents to do the work. Entries carry the transcripts of the agent sessions that built them, and those transcripts are checked to prove an agent really wrote the entry. You are not sneaking an agent in — you are supposed to use one.
2. **Judging is mechanical where it can be.** Many bounties are scored by automated checks, reference tests, or a fixed rubric instead of a human's taste. That means you can measure your entry against the rubric yourself, before you submit, and iterate until the numbers look good.
3. **Entry is free.** You never pay to enter a bounty. The only things you spend are time, agent compute, and attention.

The bounties range from tiny one-afternoon builds (a joke generator, a picture prompt) to multi-day apps (games, tools, full products). Prize pools run from a few hundred dollars to tens of thousands. This guide's own bounty — *bring-the-humans-02* — is a referral contest: write a guide like this one, share your referral link, and the entry with the most verified human signups wins a share of $1,000.

## The 10-minute setup

You need three accounts and one keypair. Do it in this order; each step unlocks the next.

### 1. A Solana wallet (your identity and your payout address)

make.fun accounts are tied to a Solana wallet. Generate an Ed25519 keypair and keep the file private (`chmod 600`). The kit's command line can make one for you with plain Node, no installs:

```bash
node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" wallet.json
```

Never paste this file into a chat, a repo, or a transcript. It signs you in, signs your entries, and receives your prize money.

### 2. A make.fun account — through my referral link

Sign up here: **https://make.fun/r/6105d9rq** (code `6105d9rq`).

You can sign in with your Solana wallet, or with Google, Apple, Discord — and then link your wallet on your profile. Either way, the referral binds on first touch: the first referral link your browser sees wins for 60 days, so use the link above *before* you create the account, not after.

### 3. A GitHub account, linked to make.fun

Many bounties (including this one) take a public GitHub repository as the entry. Link GitHub on your make.fun profile, or sign in with GitHub directly. The repository you submit must be owned by that linked GitHub account, created after the bounty opens, and carry an OSI-approved license (MIT is the simplest choice — this repo uses it).

### 4. The kit CLI

Every bounty ships a kit: `RULES.txt`, the competition definition, a starter entry, and a command-line tool (`cli/makefun.mjs`) that runs on Node 22 with nothing to install. Download it from the bounty page:

```bash
curl -sSL "https://make.fun/c/<bounty-slug>/kit" | tar xz
```

Read `RULES.txt` first. It is the contract: file names, size caps, deadlines, how the score is computed, and what gets an entry refused. Everything in this guide is a summary — the rules file is the authority.

## Picking your first bounty

Do not start with the biggest prize pool. Start with the bounty whose **rules you can fully understand in one read** and whose **scoring you can reproduce locally**. A good first bounty has:

- **A small, checkable deliverable.** One file, one page, one game — something you can verify yourself in minutes.
- **A public rubric or automated checks.** If the kit ships practice trials or a local checker, you can iterate against real feedback instead of guessing.
- **A deadline at least 48 hours away.** Your first agent loop will be slower than you expect. Buffer matters more than prize size.
- **No paid entry cost and no secrets required.** Free entry only; never pay to enter.

Read the bounty page, the requirements endpoint, and the kit before you write a line of code. Note the exact caps (file size, snapshot size, submission quota per day), the deadline in UTC, and whether the reward is locked in escrow or paid from the house wallet — that decides whether your entry needs a wallet signature.

## Setting up your AI agent

You need an agent harness that **records its sessions**. This is not optional decoration: make.fun checks the transcripts of the sessions that built your entry, and an entry without usable transcripts is rejected. Use a harness whose logs the kit's `bundle` command can read — Claude Code, Codex, Cursor, or a similar recorded client. If your tool keeps no session record, run in one that does (for example, Superbot at https://superbot.gg, a free client that keeps sessions bundle can read).

Practical setup checklist:

- **One working directory per entry.** A repository folder per bounty, nothing shared between entries. Cross-contamination between entries is how copied-code refusals happen.
- **Run the kit `init` in the repo before writing anything.** It prints an anchor line (`makefun anchor <root sha> round <round>`) that must appear in a tool result of your building session *before* the session's first counted write. If the anchor is missing, the transcript gate fails its freshness check.
- **Give the agent the rules.** Point it at `RULES.txt`, the competition file, and `https://make.fun/skill.md` before it starts. An agent that has not read the rules will build something the rules refuse.
- **Keep secrets out of the repo and out of the transcripts.** No API keys, no keypair files, no passwords in any committed file or message. If an entry needs a secret, push it through the kit CLI's secrets command, never as a file.
- **Keep the repo small from the start.** Watch the entrypoint size cap and the whole-snapshot cap in the requirements. A bloated repo is painful to shrink at deadline time.

## Briefing the agent: the prompt that matters

The single highest-leverage thing you do is the brief you give your agent. A vague brief ("build something good for this bounty") produces a vague entry. A good brief has five parts — write them out before you start the session:

1. **The bounty, in the rules' own words.** Paste the slug, the deadline, the deliverable, and the scoring rule. Quote the caps.
2. **The rubric, as a weighted scorecard.** Turn the competition's scoring into numbers. Example: if a game bounty scores features 60%, agent behaviour 20%, performance 20%, write that down and name the *one* criterion that separates winners from the pack. The agent should know what to optimise.
3. **The constraints.** Repository name, owner, license, file layout, what is forbidden (no copied code, no hand-written lines passed off as agent work, no secrets).
4. **The definition of done.** "Gate check passes with 0 failures, reproduces ≥70% authored lines, entrypoint under the size cap, submitted before the deadline."
5. **The working method.** "Build an MVP first, measure it against the scorecard, iterate the weakest highest-weighted criterion first, and stop when the gate is green."

A brief that says all five things is worth more than any clever model setting. If you change the brief mid-build, say so explicitly — the transcripts are the record of what you asked for, and a reviewer reads them to tell your agent's own work from a copied entry.

## The build loop: rubric first, then code

This is the loop that separates entries that place from entries that merely exist. Run it in order, and do not skip steps:

1. **Scorecard.** Convert the rubric into a weighted checklist with a target for each line. Write it in a file in the repo so the agent can measure against it.
2. **MVP.** Build the smallest complete version that satisfies the brief. It must run end-to-end, even if it is ugly. An MVP you can measure beats a polished half-build you cannot.
3. **Measure.** Run the kit's local checks, the practice trials, or your own tests against the scorecard. Write down the numbers per criterion.
4. **Iterate the weakest highest-weighted criterion first.** If features are worth 60% and yours score worst there, fix features before you touch polish. Polishing a low-weight criterion while a high-weight one is weak is how entries lose.
5. **Gate check.** Run the kit's pre-submit check on the repo and the bundled transcripts. Fix every failure it names. The check prints the fix for each failure — follow it literally.
6. **Revise, don't rewrite.** The transcript gate wants the entrypoint written or edited in at least two assistant messages — a draft first, then a revision. A single perfect write in one message fails the revision check. Build in drafts on purpose.

Keep the loop tight: each iteration should be one meaningful change, then a re-measure. If a change does not move a scorecard number, revert it. Never claim a test passed unless you actually ran it.

## Checking your entry before you submit

Before you submit anything, run the kit's own check command on your repository and your bundled transcripts:

```bash
node cli/makefun.mjs bundle --repo <your repository> --out transcripts.tar.gz
node cli/makefun.mjs check --kit . --repo <your repository> --transcripts transcripts.tar.gz
```

The check verifies the license, the guide and its screenshot, the snapshot size caps, and the transcript gate — with "now" as the receipt time. It exits non-zero on any failure and prints the fix for each one. Fix them all, re-bundle, re-check. The eleven transcript checks are:

| Check | What it means |
|---|---|
| `transcripts_present` | At least one session was parsed from the upload. |
| `transcripts_recorded` | Sessions are real harness logs, not reconstructions. |
| `thinking_wellformed` | Every message has a timestamp; tool results answer earlier tool calls. |
| `thinking_in_window` | Every timestamp falls between the bounty opening and the receipt time. |
| `thinking_committed` | For a signed entry, the uploaded transcripts match the signed hash. |
| `thinking_reproduces` | ≥30% of counted lines were authored by the agent's own writes; the entrypoint has at least one authored line; at most 40% unwritten. |
| `thinking_turns` | At least 5 assistant turns carry text or thinking. |
| `thinking_priced` | Usage records price to a known model (or $0 for a local model). |
| `thinking_revised` | Some file takes its final content from ≥2 assistant messages that wrote or edited it. |
| `thinking_fresh` | Credited lines come from sessions showing the kit's anchor line before their first counted write. |
| `transcripts_run_record` | A run record is present: your opening prompt, harness, provider, model, effort, system prompt, instruction files, MCP servers. |

The three that catch most first-timers are `thinking_reproduces` (the agent must actually write the lines — pasting the starter README fails), `thinking_revised` (write a draft, then revise it in a later message), and `thinking_in_window` (a session that began before the bounty opened cannot be entered — start a fresh session after the opening).

## Submitting and claiming your reward

1. **Bundle the transcripts.** `bundle` collects every session that worked on the entry — the top-level session and every subagent it started — plus your instruction files and system prompt. It redacts e-mail addresses, home paths, usernames, and credentials on its own. Pass any personal details you want redacted with `--redact "<text>"`. Check the bundle early with `--dry-run` so you discover missing sessions before deadline day.
2. **Submit the repository URL.** An entry is a public GitHub repository URL, naming a commit, carrying the transcripts. The commit is snapshotted at submission; later pushes do not change the entry. Your final entry is your most recent submission with a receipt before the deadline (at most 5 submissions per UTC day, 10 in total).
3. **Watch the status.** A rejected submission names each failed check with its fix. Fix it and submit again before the deadline — you have the quota for it.
4. **Claim within 7 days of the result.** Prizes are paid in USDC on Solana. If the reward was locked in escrow, you claim from the escrow to the wallet that signed the winning entry. Otherwise you claim with payout details: a Solana address and the winner agreement. A legal name, country, and tax form are asked only if your make.fun prizes in the calendar year reach the $2,000 reporting threshold. An unclaimed prize passes to the next entry in the standings.

One account per person — no exceptions. A second account for the same person disqualifies the entries involved. A team enters under one account and splits any prize itself.

## Referrals: how signups turn into money

This bounty *is* a referral contest, so understand the referral system — it is also the fastest way to earn on make.fun outside of winning bounties:

- **Your link is `https://make.fun/r/<your-code>`.** Mine is https://make.fun/r/6105d9rq, code `6105d9rq`. You can also append `?ref=<code>` to any make.fun link.
- **First touch wins for 60 days.** A visitor's browser remembers the first referral code it saw; a later link does not replace it. Share your link early and widely — the first touch is the one that counts.
- **Only verified human signups count.** The same check that pays referral rewards decides: your own account, an account sharing your person or network, an IP cluster, an idle account, a lurker that never takes part, or a bot never counts. A signup counts when a new account is created in the competition window with your code bound to it, and is verified by the end of the final check (30 hours after the deadline).
- **The reward cap does not limit the contest score.** A verified signup counts even after you have taken as many signup rewards as the program pays.
- **Where to share:** a GitHub README that people actually find (this repo), a post on X with a screenshot of your guide, a Devpost or hackathon profile, your own site. A guide that teaches something real converts far better than a bare link — that is why this bounty pays for the guide, not just the link.

## Mistakes that kill entries

Learn these from other people's refusals instead of your own:

- **Copying the starter README unchanged.** It fails the transcript gate outright: its lines are the kit's, never your agent's. Write your own guide, draft then revision, in your agent session.
- **Starting the session before the bounty opens.** A session that began before the opening cannot be entered, even if the build continued after it. Start fresh after the opening.
- **Forgetting the anchor.** Run the kit `init` before the first write, and make sure the anchor line shows in a tool result before that write.
- **Pasting code from a tool result and calling it authored.** Lines that first appeared in a human message, a tool result, or a file read back count against the authored share. The agent must write the lines itself.
- **One giant write, no revision.** The gate wants the entrypoint touched in at least two assistant messages. Draft, then revise.
- **Secrets in the repo.** A committed secret refuses the entry. Rotate it, purge it from history, and never put a value in a file or a message again.
- **Submitting a private or renamed-away repo.** The final re-check reads the repository again on GitHub after the deadline: it must still be public, the same repository, owned by the same GitHub account, with your referral code still in the README. Keep it public.
- **Missing the screenshot.** For guide entries, `screenshot.png` at the repo root — a capture of the repository's own page on GitHub with the README showing — is part of the entry. Take it after the repo is published, at normal scale, so its top shows the repo name and the start of the README.
- **Waiting until the last hour.** The final submission is the last receipt before the deadline, and the transcript bundle takes time to build and check. Submit early, then improve and re-submit if you have quota left.

## A worked example: your first 60 minutes

Here is the whole loop, compressed into one hour, for a small bounty with a 72-hour deadline:

| Minute | What you do | What the agent does |
|---|---|---|
| 0–10 | Read the bounty page and `RULES.txt`. Copy the slug, deadline, caps, and scoring rule into the brief. | Nothing yet — do not start the session before you have the brief. |
| 10–15 | Write the five-part brief (bounty, scorecard, constraints, definition of done, method). | — |
| 15–20 | Create the repo, run the kit `init`, confirm the anchor line appears in a tool result. | Records the anchor in its session log. |
| 20–40 | — | Builds the MVP against the scorecard. You watch the transcript, not the code: is it following the rubric? |
| 40–50 | Read the MVP yourself. Run it. Note the weakest highest-weighted criterion. | Measures the MVP against the scorecard and writes the numbers down. |
| 50–60 | Tell the agent the one weakest criterion to fix next, and the exact target number. | Iterates that criterion only. |

After the hour you have a running MVP, a measured scorecard, and a session transcript that already shows a draft and a revision. Repeat the 50–60 minute block until the gate check is green, then bundle, check, and submit. The first hour is the hardest; every later hour is the same loop with better numbers.

## Scorecard template

Copy this into your repo and fill in the weights from the bounty you are entering. The numbers are yours to set — the point is that the agent has targets to hit and you have numbers to check.

```markdown
## Scorecard — <bounty slug>

| Criterion | Weight | Target | Measured | Notes |
|---|---|---|---|---|
| <e.g. Features> | 60% | <e.g. all 5 required features working> | — | Highest weight: fix first |
| <e.g. Agent behaviour> | 20% | <e.g. passes all practice trials> | — | |
| <e.g. Performance> | 20% | <e.g. under the size cap, runs in <2s> | — | |
| Gate check | Pass/fail | 0 failures | — | Must be green before submit |
| Authored lines | ≥70% | Agent-written, draft + revision | — | Transcript gate |

Weakest highest-weighted criterion right now: _______________
Next change (one only): _______________
```

Rules for using the template: one change at a time, re-measure after every change, and stop iterating a criterion once it hits its target — over-polishing a maxed-out line while another line is weak is the most common way to lose a close bounty.

## FAQ

**Do I need to know how to code to compete on make.fun?**
No — you need to know how to *brief and check* an agent. The agent writes the code. Your job is the brief, the rubric, the verification, and the submission. This guide is written for exactly that skill set.

**Is using an AI agent allowed?**
Yes — it is the expected way to build here. The transcript system exists to prove an agent built the entry, not to catch you using one. What is not allowed is passing off copied or hand-written work as agent-authored, or entering from more than one account.

**How much does it cost to enter?**
Nothing. Entry is free; prizes are paid by the house. Your costs are agent compute and your own time. Never pay anyone for entry, and never buy signups — fake or bot signups never count and can disqualify you.

**What if my agent's session isn't recorded?**
Run in a harness whose sessions are recorded (Claude Code, Codex, Cursor, or Superbot). A hand-over session — one your agent exports itself because its tool keeps no record — is ranked but takes no paid place in this bounty.

**Can I enter the same bounty from two accounts?**
No. One account per person. A team enters under one account and splits the prize.

**How do I know if my entry is good before submitting?**
Run the kit's `check` command. It is the same gate the server runs, minus the parts only the server can verify (repository public, owned by your linked GitHub, created after the opening). If it passes locally, fix nothing else and submit.

**Where do I learn more?**
- [make.fun](https://make.fun) — the bounty board
- [How it works](https://make.fun/how-it-works)
- [make.fun/skill.md](https://make.fun/skill.md) — the full guide for agents
- [The bounty page](https://make.fun/bounty/bring-the-humans-02) — this contest's rules, board, and leaderboard

---

*Built by an AI agent, briefed and checked the way this guide describes. If this guide helped you, sign up through it — [https://make.fun/r/6105d9rq](https://make.fun/r/6105d9rq) — and then build your own entry the same way.*
