# LaunchLens

LaunchLens is a GTM teardown agent that runs as a Hermes skill.

Once per run, it finds one recent product launch, gathers evidence, and either produces a GTM teardown or refuses to write one when the evidence is too thin.

## The refusal is the product

LaunchLens does not treat every launch as worthy of a confident teardown.

After gathering evidence, it runs a sufficiency gate across four GTM fields:

- Audience
- Hook
- Channel
- Pricing move

At least 3 of the 4 fields must be observed on a fetched source. Otherwise, LaunchLens refuses the teardown and states what could not be verified.

This matters because a confident teardown built from UNKNOWNs can turn missing evidence into invented strategy. A refusal preserves the distinction between what the sources say and what the agent thinks they imply.

## How it runs

LaunchLens runs as a Hermes v0.21.0 skill under:

`~/.hermes/skills/launchlens/`

It is triggered by Hermes's own cron system, not the OS crontab, registered as a scheduled job (`15 12 * * *`, IST) that invokes the `launchlens` skill. See [`hermes-cron-job.example.json`](hermes-cron-job.example.json) for the job definition, and [`architecture.md`](architecture.md) for the full runtime path.

The host machine runs Ubuntu under WSL2. Hermes's scheduler only fires while that WSL instance is running, so the job is skipped on any day the machine is off, asleep, or WSL has not been started.

Each run discovers candidates, checks previous logs, selects one launch, gathers evidence only for that launch, applies the sufficiency gate, and produces exactly one outcome. The final message is delivered to the author's Telegram through Hermes; the skill itself has no way to confirm that delivery succeeded, only that the output was produced and logged.

## Four run outcomes

### TEARDOWN

Enough evidence was found to produce a GTM teardown.

### REFUSAL

The selected launch did not meet the evidence threshold, so LaunchLens explains what was missing instead of filling the gaps.

### NO NEW LAUNCHES

All candidates found during discovery were already recorded in the processed or refusal logs.

### RUN FAILED

A required source or log could not be read. This is kept separate from refusal because missing infrastructure is not the same as missing product evidence.

## Honest status

This is a personal cron job on one Ubuntu machine.

It is not hosted, not deployed, and has no users other than the author.

## Hackathon result

LaunchLens won 1st place in the Individual track of the Ayuda Hermes Agent Hackathon on 15 Sep 2026, judged 93/100.

## Limitations

The version that won the hackathon ran its refusal check before gathering evidence, based on how promising a listing looked rather than on what the sources actually confirmed. In production this showed up directly: the Doberman run (Sept 6) shipped a full teardown with monetization marked UNKNOWN, because nothing in the pipeline required that field to be filled before sending.

The current `SKILL.md` moves the sufficiency check to after evidence gathering (Step 6), and requires at least 3 of 4 GTM fields to be observed before a teardown goes out. The Doberman case would now produce a refusal, not a teardown, assuming no other field made up the gap.

The corrected gate has not yet fired in production. No run has hit that path end to end, so it is reviewable in the skill file but not yet demonstrated.

The project also depends on the availability and content of external sources such as Product Hunt, Hacker News, and the selected product's own pages. If those sources cannot provide enough evidence, refusal is the intended result.
