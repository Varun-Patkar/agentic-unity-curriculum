# Story Bible — *Hearthfall*

> **Status: DRAFT SCAFFOLD.** This is a structure with your name on the blanks, not a finished story. You said you want to think the narrative through properly — good, you should. What follows is designed so that you can replace every name, place, and line of dialogue without touching a single system. The *shape* is what the code depends on.
>
> **Working title:** `Hearthfall`. One find-and-replace to change it.

---

## 1. The one-line pitch

*A peasant leaves a starving valley to find coin and glory. He finds both. The game is the road home.*

## 2. The theme

The thing worth exploring, in your words: **the most successful people are often the richest, and often they lost themselves getting there — and it worked.**

So the game must refuse the easy moral. Being ruthless in Vaskirk **works**. It pays, visibly, in coin the player watches accumulate. Nobody punishes you. The city rewards you exactly as it promised. The bill arrives somewhere else entirely, addressed to people who never met the man who signed for it.

Three rules that keep this honest:

1. **No moral meter. No karma UI. Ever.** The moment the player sees a red bar, the theme dies — it becomes a puzzle to optimise instead of a life to live.
2. **Every corrupt choice is argued for by someone sympathetic and correct.** If the player can dismiss the tempter as a villain, he has not been tempted. Vance must be *right*.
3. **Consequences are delayed, displaced, and diegetic.** Never "your reputation decreased." Always: a letter that mentions food less than the last one did.

## 3. The three endings

Ending is determined by two hidden ledgers (`Coin`, `Conscience`) **plus one final choice**. That last part matters: in two of three endings the player chooses. In the third, the choice has already been made, in a counting-house, months ago, in good handwriting.

### Ending A — "Set the Sword Down"
**Conscience: high · Coin: modest but sufficient · Final choice: stay**

You turned down the profitable things. You came home poorer than you could have been — but with real coin, enough to buy the valley out of debt and put a roof on. The family is whole. You hang the sword above the hearth and you are a farmer again, and the game does not treat that as a small thing.

*Tone: quiet, earned, warm. Not saccharine. He is going to be poor and tired for the rest of his life and it is the right trade.*

### Ending B — "The Tithe" · **the gut-punch**
**Conscience: low · Coin: maximum · Final choice: removed**

You did the arithmetic. Every step was defensible, most of it was legal, and all of it paid. The Assize was honest bookkeeping. The name you gave was a criminal's. The grain contract was a fair price agreed by willing parties.

You ride home rich. The levy on the Weald went up by a third — you saw the figure yourself, you helped produce it, you took a share of it. Your father sold the oxen. Then the field. Then he stopped writing.

The final scene offers you no choice at all. That is the punch: this game has asked you what you want to do for ten hours, and here, at the only moment it matters, it doesn't ask. Because you already answered.

*Tone: cold, procedural, quiet. **No score screen. No villain reveal. No music sting.** Let him sit in the room. The most brutal thing the game can do here is show him the ledger in his own hand.*

### Ending C — "The Peasant Knight"
**Conscience: middling · Coin: high · Final choice: go**

You bent when bending paid and held when holding mattered. You didn't ruin anyone, but you didn't save anyone either — you took the situational route every time and it worked out. You come home with real money, you put it in your sister's hands, you eat one meal at your father's table, and in the morning you are on the road again.

Not a hero, not a monster. A man who found out what he's for and it isn't this valley. The family is fine. They will see him maybe twice more before he dies.

*Tone: bittersweet, restless, honest. This is probably the most common ending, and it should feel like the most human one.*

---

## 4. The hidden ledgers

Both live in `Hearthfall.Core`. Neither is ever shown.

**`Coin`** — actual currency, plus a `LifetimeEarned` counter that never decreases. What matters for the ending is what you *took*, not what you still have.

**`Conscience`** — not a score, a **ledger of decisions**. `List<ConsciencePoint>`, each with the choice ID, the weight, and the flag it set. This matters for two reasons:
- It lets the epilogue *cite specific choices back at the player*: "you named Tam Ferrier in the third week of Lent." A single number could never do that. This is the whole reason Ending B lands.
- It makes the system testable in a console app with no Unity involved.

**`Roots`** — a small set of flags about the family and the valley, mutated by delayed consequences: `FatherHealth`, `ValleyLevy`, `TollgateOpen`, `HarvestSold`, `EnidsLastLetterTone`.

Ending selection reads all three at the final gate. Nothing else does.

## 5. The Three Ledgers — the mechanism

The spine of the whole design. Three optional, profitable, *individually reasonable* jobs in Vaskirk, each of which sets a flag that resolves back home in Act III. Each has an honest alternative that pays less and costs nothing later.

| # | The job | Who asks | Why it's reasonable | What it does to the Weald |
|---|---|---|---|---|
| 1 | **The Assize** — help re-survey the Weald's true yield for the levy | Steward Vance | The old rolls are decades out of date and genuinely wrong. Accuracy is not a crime. You get a surveyor's share. | The levy on Hearthfall rises by a third. Your father cannot pay it. |
| 2 | **The Name** — identify who is running grain past the tollgate | Vance, on the Sheriff's behalf | Tam Ferrier *is* a smuggler. He is not a good man. There is a real bounty and you have a real need. | The tollgate route closes. Cheap grain stops reaching the valley. |
| 3 | **The Forward Contract** — sell the Weald's next harvest to the Grain Hall now, at a set price | Iselde | A guaranteed price protects farmers from a bad market. Genuinely. Sometimes it even helps them. You take the broker's fee. | The harvest is sold before it is grown. The winter is hungry. |

**Take all three → Ending B is fully earned and entirely the player's own reasoning.**
**Take one or two → the world is damaged but survivable. Ending C territory.**
**Take none and you will be visibly, painfully poorer for most of Act II** — and that must genuinely sting, or Ending A means nothing.

### The design rule that makes this work
> The player must never be able to say *"the game tricked me."* Every consequence must be foreseeable in hindsight and invisible in the moment. If a player replays and thinks *"oh — it told me, I just wasn't listening"*, the design succeeded.

## 6. Enid's letters — the feedback channel

Three letters from your sister arrive during Act II. They are the entire consequence-feedback system, and they cost nothing to build because they are text.

They **never accuse**. They never mention the levy, the tollgate, or the contract. They simply change:

- **Letter 1** — long, warm, gossip about neighbours, asks what the city looks like.
- **Letter 2** — shorter. Practical. Asks when you're coming back. Mentions the price of things, once, then apologises for mentioning it.
- **Letter 3** — four lines. Does not mention food at all. Signs off differently.

The tone of each is selected by how many Ledgers you've taken. Clean run: the letters stay warm and get funnier. Full-corruption run: letter 3 is four lines and one of them is a lie.

*This is the cheapest, highest-impact narrative system in the whole game. Build it well.*

## 7. Cast

> Placeholder names, drawn Anglo-Saxon/Norman to sit in a KCD register. Replace freely — the **role** is what matters.

| Character | Role | The thing that makes them memorable |
|---|---|---|
| **(player-named)**, default **Wat** | The peasant | Not chosen, not special, not secretly a lord's son. Emphatically ordinary. |
| **Osric** | Father | Proud past the point of sense. Will not write to ask for help, and that is what kills him. |
| **Enid** | Sister | Runs the farm in practice. Writes the letters. Sharper than everyone in the valley including you. |
| **Cob** | Younger brother | Wants to follow you. In one ending he does. |
| **Aldric Vance** | Steward of Vaskirk — **the tempter** | Decent, tired, competent, and *right*. Believes accurate records are a public good. He is not lying to you once. |
| **Sgt. Brannoc "Bran"** | Mercenary, teaches you the sword | Situational morality worn openly and without shame. He is Ending C, walking around. He is also the only one who's honest with you. |
| **Iselde of the Grain Hall** | Broker | Transactional, funny, zero illusions about what she does. Likes you. Would ruin you and say so first. |
| **Father Corvin** | Priest | The conscience voice — and a hypocrite, so the player can't just outsource his morality to him. |
| **Tam Ferrier** | Smuggler | Genuinely a criminal. Genuinely the reason your valley eats. Both true at once. |

## 8. Places

| Place | Function | Note for the art-direction milestone (M13) |
|---|---|---|
| **Hearthfall** | Home hamlet in the Weald. Village hub, Act I and Act III. | Same geometry both acts. **Only the light, weather, and props change.** Cheapest, most devastating storytelling in the game. |
| **The Wealdrun** | The road. Travel, ambush, the tollgate. | Long, cold, empty. Where the two combat encounters live. |
| **Vaskirk** | The city. Act II. Counting-house, Grain Hall, barracks, cathedral steps. | Warmer, denser, richer light than the valley. It should look *better*. It should look like it's worth it. |
| **Greyhold** | The keep / dungeon. The "adventure" you came for. | The one place that looks like a fantasy game. Deliberately. It's the lie you were sold. |

## 9. Three acts, mapped to the build

| Act | Content | Built in |
|---|---|---|
| **I — The Leaving** | Hearthfall. Establish family as people, not backstory. The valley is in trouble but survivable. You choose to go. | M04–M06 (2D) · M14 (3D) |
| **II — Vaskirk** | The city. Bran teaches you to fight. Vance offers the Assize. Iselde offers the contract. Greyhold pays out. Enid's letters arrive. | M05–M09 (2D) · M12–M14 (3D) |
| **III — The Road Home** | The Wealdrun in winter. Hearthfall again, changed by exactly what you did. The final gate. The three endings. | M08–M09 (2D) · M14 (3D) |

## 10. Scope guardrails

**In:** one hub, one road, one keep · 4–6 major branching choices · 3 endings · light/heavy/dodge/stamina/lock-on · 2–3 enemy types · save/load with choice history · dialogue UI with portraits · quest log · a handful of Kokoro-voiced lines · full game shell.

**Out, deliberately, and do not relitigate this in December:** open world · full voice acting · crafting · inventory economy · survival stats · mounts · day/night simulation · romance · a fourth ending · "just one more area."

## 11. Your homework

Fill these in as you go — buffer days are for this. The systems don't care what you write, only that something is written by the day the milestone needs it.

- [ ] **Rename the game**, or decide `Hearthfall` stays. (by M02)
- [ ] **The father's specific pride** — what exact thing will he not do, and why? This is Ending B's load-bearing beam. (by M05)
- [ ] **Vance's best argument** — write the actual paragraph where he explains the Assize. If it doesn't convince *you*, rewrite it. (by M05)
- [ ] **The four lines of Enid's third letter**, corrupt version. (by M06)
- [ ] **Ending B's final image.** What is the last thing on screen? (by M08)
- [ ] **The other 3–4 minor choices** that aren't the Three Ledgers — smaller, more personal, less costly. (by M06)
- [ ] **Ten lines of Bran's dialogue.** If he's funny, the whole middle of the game works. (by M07)
