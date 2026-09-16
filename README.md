# Last Stop, Hollowbrook — An Agentic Unity Curriculum

**112 days. One hour a day. From "I have never shipped a game" to a released 3D choice RPG with branching consequences.**

You are not going to watch tutorials. You are going to build things, break them, fix them, and commit them. An AI agent sits beside you as a pair — not as a teacher reading a script, and never as the person who writes your game for you.

---

## The deal

- **Start:** 10 September 2026 · **End:** 30 December 2026
- **1 hour per day.** That is the atomic unit. Weekends you *may* do two hours. You are never required to.
- **Every 7th day is a buffer day.** Catch up, polish, or genuinely rest. A missed day never cascades into a dead project.
- **16 milestones.** Each one ends with something that runs.
- **Two public releases.** A 2D vertical slice on itch.io at Day 70. The full 3D game at Day 112.

## Start here, right now

1. Read [AGENTS.md](AGENTS.md) — it is short, and it is the contract your AI agent follows.
2. Skim [CURRICULUM.md](CURRICULUM.md) — the whole 112 days on one page.
3. Open [milestones/M00-ground-zero.md](milestones/M00-ground-zero.md) and read Day 1.
4. Type `/session` in Copilot Chat. That is it. That is the whole ritual.

## The four commands

| Command | When you use it |
|---|---|
| `/session` | Every day. Runs today's hour. |
| `/unstuck` | The moment you have been stuck for more than 10 minutes. |
| `/review` | End of a milestone, or any time you want your code torn apart constructively. |
| `/catchup` | You missed days. Life happened. This re-plans without guilt. |

They live in [.github/prompts/](.github/prompts/). Any agent, any model — the prompts carry the context, not the conversation.

## What you are building

**Last Stop, Hollowbrook.** Alex Reed returns to an ordinary small town after their sibling disappears and discovers that Mayor Vale has kept it safe through a bargain with something beneath the old mine.

A modern supernatural-town choice RPG: sincere mystery, light combat, lasting consequences, and exactly three endings, including one first-playthrough secret for players who refuse to hear the Mayor out.

Full design in [reference/story-bible.md](reference/story-bible.md). It is *your* story — the bible is a scaffold with your name on the blanks.

## Repo map

```
README.md               you are here
AGENTS.md               the contract every AI agent obeys — read this
CURRICULUM.md           all 112 days on one page

.github/prompts/        /session /unstuck /review /catchup
milestones/             M00 .. M15 — the day briefs, seven days per file
reference/              glossary, editor map, asset pipeline, debugging, story bible
progress/
  STATE.md              where you are right now (agents read + write this)
  LOG.md                what you did each day (agents append to this)
```

## Where the code lives

**Not here.** This repo is curriculum. Your Unity projects live beside it:

```
D:\Projects\Unity Games\
  Unity Agentic Tutorial\     <- this repo (curriculum)
  Sandbox00\                  <- M00 throwaway
  ChickenChase\               <- M01 throwaway, your first finished game
  Hollowbrook\                <- M02 onward. The real one.
```

Each Unity project is its own git repo. The curriculum never contains game code, so you can wipe and restart a project without losing your plan.

## Rules that actually matter

1. **One hour. Then stop.** Even mid-task. Especially mid-task — you will restart faster tomorrow with a warm problem than a cold one.
2. **Commit every day, even a broken one.** `wip: day 23, dialogue advances but choices do not render` is a perfectly good commit message.
3. **Never let the agent write a system for you before you understand it.** Ask it to explain, then you type it. Copy-paste is how tutorial hell wins.
4. **Acceptance criteria are not vibes.** Every day brief has a checklist. Done means the checklist passes.
5. **When you are stuck, you are 10 minutes from `/unstuck`, not from quitting.** That is the entire point of this format.

---

## Using this yourself

This was built for one person, but nothing in it is personal. If you want to run it:

1. Fork or clone the repo. It contains no game code — only the plan.
2. Read [AGENTS.md](AGENTS.md). It is the file that makes an AI agent behave like a pair rather than a code vending machine. Most of the value is there.
3. Change the game. Swap [reference/story-bible.md](reference/story-bible.md) for your own premise and update the content briefs; the milestones teach systems, but their examples deliberately use Hollowbrook's cast and choices.
4. Reset [progress/STATE.md](progress/STATE.md) to Day 1 and shift the dates.

**Assumed background:** comfortable in C#, comfortable with git, zero Unity. If you're new to programming, the pacing will be brutal — the briefs skip every language concept on purpose.

**It is opinionated on purpose.** Engine-agnostic core from Day 22, the new Input System, URP, Cinemachine, no ScriptableObject-as-logic, no Addressables or DOTS. If you disagree with a call, the briefs are short enough to rewrite.

## Licence

MIT — see [LICENSE](LICENSE). Take it, fork it, teach with it.

---

*You have shipped software before. Games are software with a rendering budget and better feedback. You already have most of what you need.*

**Day 1 is [here](milestones/M00-ground-zero.md).**
