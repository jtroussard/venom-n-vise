# GDD: Venom & Vise

| | |
|---|---|
| **Course** | Multi-Agent AI for Game Development |
| **Instructor** | Joshua Burdick |
| **Assignment** | #1 – First Draft of GDD |
| **Target Architecture** | Unreal Engine 5 / Godot 4 + Claude 3.5 Haiku API |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Game Mechanics](#2-game-mechanics-player-facing-actions--core-loop)
3. [AI Architecture](#3-ai-architecture-gameplay-facing-agent-behaviors)
4. [Technical Strategy](#4-technical-strategy-pipeline-budget--constraints)

---

## 1. Executive Summary

**Venom & Vise** is a high-tension, first-person sub-aquatic harvesting and resource management game. Players dive into a bounded coral amphitheater off the Florida coast to cull invasive lionfish under strict oxygen constraints, while managing an escalating negotiation with a protected (and moody) 7-foot green moray eel named **"Slippy"**.

### Core Gameplay Loop

1. **Dive** into the reef to spear invasive lionfish in sub-aquatic first-person 3D.
2. **Manage** Slippy's predatory appetite via sacrificial tributes to maintain harvesting safety and coax cooperation.
3. **Surface** to the boat and triage the catch on an aluminum prep bench under a 30-second thermal spoilage timer.
4. **Settle** sales and regulatory compliance at the harbor dock between **Mac the Fishmonger** and **Officer Vance** of the Florida Fish and Wildlife Conservation Commission (FWC).

### Win / Loss Conditions

- **Win Condition:** Fulfill commercial quotas, maximize fillet revenue at dockside, and safely surface without permit revocation.
- **Loss Condition:** Diver blackout from oxygen starvation, lethal bites from an enraged apex predator, or commercial permit revocation/arrest by the FWC for severely injuring or killing protected native wildlife.

### The "One Agent, One Wow" (Slide 9)

**The Slippy Tactical Intent Agent.** Rather than functioning as a deterministic damage hazard or static timer bar, Slippy operates as an expressive, cognitive co-hunter and extortionist. The agent processes dive telemetry, strike history, and catch quality to dynamically toggle between flushing hidden trophy lionfish from deep caverns or ambushing the diver to poach speared fish right off the shaft.

---

## 2. Game Mechanics (Player-Facing Actions & Core Loop)

### Phase 1: The Reef Descent & Quarry Spearing

#### Locomotion & Camera Control

Sub-aquatic first-person controller.

| Input | Action |
|---|---|
| `W` `A` `S` `D` | Horizontal thrust |
| Mouse | Aim (pitch / yaw) |
| `Space` | Ascend |
| `Ctrl` / `C` | Descend |

Camera roll is locked to zero to eliminate disorientation and nausea while preserving 3D verticality in a 30-meter circular reef basin.

#### Spear Physics & Strike Orientation

The player wields a pneumatic pole spear firing a high-velocity raycast. Catch quality is determined deterministically via vector math combining impact position along the mesh axis with a relative velocity dot product (`V_spear · V_fish`):

| Zone | Strike Type | Angle | Mesh Region | Result | Market Value | Slippy Reaction |
|---|---|---|---|---|---|---|
| **A** | Head Strike | < 45° | Front 30% | Spines fold back flat | **100%** | Optimal tribute — swallows cleanly |
| **B** | Flank Strike | 45°–120° | Mid 40% | Spines remain partially flared | **90%** | Feeding causes hesitation |
| **C** | Tail / Reverse Strike | > 120° | Rear 30% | Severely mangled fillet, flared venomous spines | **75%** | Rejected — immediate aggression penalty |

#### Predatory Tribute & Transit Buffer

Slippy resides at a fixed cavern coordinate (`Den_Origin`). Feeding Slippy buys safety equal to his base digestion window **plus** the physical transit time he takes to swim back to his den. Players can deliberately lure Slippy away from his cavern before offering a tribute to buy extra unharassed harvesting seconds.

#### Defensive Prodding & The FWC Morality Rule

Secondary fire delivers a blunt-end prod to shove an inquisitive eel away. Prodding grants **2 seconds of clearance** but increments Slippy's hidden **grudge counter**. Repeated heavy prods incur active FWC citations, while killing the protected moray triggers instant license revocation and a hard **Game Over**.

### Phase 2: Boat Deck Prep Bench Triage

#### The Spoilage Clock

Upon surfacing, the catch spills onto an aluminum prep bench with a **30-second thermal spoilage timer** ticking down. Unsorted fish lose **5% market value every 5 seconds** past zero.

#### Tactical Triage Actions

- **Sort to Ice:** Dragging clean, high-grade fish into the ice cooler locks in market value.
- **Venom Spine Reflex Check:** Handling Zone B or Zone C fish requires timing a reflex click. Failing inflicts a venomous sting, causing screen distortion and a mandatory **\$75 deduction** for onboard antivenom.
- **Evidence Discard Chute:** Fish bearing visible eel bite marks or blunt trauma can be dropped down the chum chute. This destroys incriminating evidence of wildlife harassment before inspection, sacrificing meat bounty in exchange for regulatory safety.

#### The "Slide All to Ice" Rapid Override

Players opting for aggressive speedruns or confident in their clean spearing can press a single master toggle to dump the entire bench directly into the cooler instantly. This skips the triage timer entirely but carries all uninspected liabilities (venom stings, mangled evidence) directly to dock evaluation.

### Phase 3: The Harbor Dock Settlement

#### Dual Faction Dynamics

The player interfaces with two competing authorities via structured dialogue wheels and skill checks (fully gamepad compatible):

- **Mac (The Fishmonger):** Evaluates meat volume and fillet integrity. High reputation with Mac unlocks discounted dive gear and covert disposal of questionable catches.
- **Officer Vance (The FWC Warden):** Inspects the cooler for illegal harvest indicators, predator bite marks, and signs of native fauna harassment.

#### Resolution Encounters

If suspicious carcasses were iced, Vance initiates an inquiry. Players navigate structured dialogue branches:

- `[Blame Strong Currents]`
- `[Pay Conservation Fee]`
- `[Rely on Mac's Alibi]`

Success depends on player dive skill stats, fishmonger standing, and how much incriminating evidence was discarded during triage.

### 48-Hour Non-AI Baseline Verification (Slide 8 Scope Rule)

If all generative models and network APIs are completely severed, two developers can build and balance a fully playable greybox of this game in 48 hours:

- **Mechanics** run on standard engine swimming pawns, raycast shooting, and a 2D drag-and-drop inventory table.
- **Slippy** runs on a deterministic 3-state Finite State Machine (`PATROL → FEED → RETREAT`) driven by a simple 0–100 Agitation counter.
- **The dock** runs on basic arithmetic tally formulas and hardcoded dialogue trees.

The game is mechanically functional, balanceable, and fun before introducing any AI components.

---

## 3. AI Architecture (Gameplay-Facing Agent Behaviors)

The AI layer operates exclusively at semantic, tactical, and narrative seams where hardcoded branching trees would be too brittle, leaving real-time physics and collision entirely under engine authority.

### Agent 1: Slippy Tactical Intent Agent (The "One Wow" Player-Facing Agent)

**Development Role:** Operates as an autonomous behavioral engine that evaluates relational history, hunger levels, and diver respect to dictate predator posture and cooperation.

**Gameplay Effect:** When fed or encountering the diver, the agent selects an operational stance for the next 30–45 second window:

| Stance | Trigger | Behavior |
|---|---|---|
| **Cooperative Flusher** | Fed a pristine Zone A catch with zero prods | Actively darts into tight rock crevices, flushing massive trophy lionfish into open water for the player to spear. |
| **Opportunistic Pirate** | Fed poor scraps or previously jabbed | Stalks behind the diver's blind spot, lunging to steal speared fish directly off the shaft before they can be stowed. |
| **Territorial Blocker** | — | Occludes access to high-value coral caves, issuing deep acoustic hissing and false lunges. |

### Agent 2: Dockside Compliance & Market Agent (Supporting Studio Dev Crew)

**Development Role:** Operates as a forensic narrative adjudicator that evaluates the physical state of the sorted cooler against environmental regulations and market supply.

**Gameplay Effect:** Evaluates aggregate damage telemetry (number of bite marks, crushed spines, missing fish discarded) to generate Officer Vance's inspection strictness and dynamic dialogue responses, while shifting dockside commodity bounties based on haul size.

---

## 4. Technical Strategy (Pipeline, Budget, & Constraints)

### 4.1 Agent Roles & Data Contracts

#### Slippy Tactical Intent Agent

**Input Schema**

```json
{
  "dive_clock_remaining_sec": 85,
  "grudge_count": 1,
  "offered_fish": {
    "length_in": 16,
    "hit_zone": "HEAD_STRIKE",
    "spine_state": "FOLDED_SAFE"
  },
  "recent_diver_actions": ["PRODDED_EEL_ONCE", "RETRIEVED_SPEARED_FISH"]
}
```

**Output Schema**

```json
{
  "behavioral_stance": "COOPERATIVE_FLUSHER",
  "target_coral_crevice_id": "CAVE_NORTH_02",
  "retreat_duration_sec": 40,
  "acoustic_vocalization": "CONTENT_LOW_PURR",
  "aggression_scalar": 0.15
}
```

#### Dockside Compliance & Market Agent

**Input Schema**

```json
{
  "iced_fish_summary": {
    "total_count": 14,
    "head_strikes": 11,
    "eel_bite_marks": 3,
    "discarded_chum_count": 2
  },
  "diver_telemetry": {
    "eel_prods_delivered": 1,
    "mac_reputation_level": 3
  },
  "selected_player_dialogue": "BLAME_STRONG_CURRENTS"
}
```

**Output Schema**

```json
{
  "inspection_verdict": "CLEAR_WITH_SUSPICION",
  "fwc_fine_amount": 0,
  "market_payout_credits": 340,
  "vance_dialogue": "Those tears look an awful lot like moray teeth, kid. Mac vouched for the surge today, so you get a pass, but I'm watching your boat.",
  "mac_loyalty_delta": 1
}
```

### 4.2 Token Budget & Cost Projections

**Model target:** Claude 3.5 Haiku (\$0.25 / 1M input tokens, \$1.25 / 1M output tokens)

| Agent | Input Tokens | Output Tokens | Tokens / Call | Calls / Run | Tokens / Run |
|---|---|---|---|---|---|
| Slippy Tactical Agent | ~300 | ~75 | ~375 | 4 (avg.) | ~1,500 |
| Dockside Compliance Agent | ~450 | ~100 | ~550 | 1 | ~550 |

**Session Totals**

- **Tokens Per Dive Session:** ~2,050 tokens (1,650 input + 400 output)
- **Cost Per Session:**

  ```
  (1,650 / 1,000,000 × $0.25) + (400 / 1,000,000 × $1.25)
  = $0.00041 + $0.00050
  ≈ $0.00091 / run
  ```

- **Monthly Cost at 1,000 Sessions:** ≈ **\$0.91 / month**

### 4.3 Technical Requirements & Operational Constraints

#### Rate Limits

Anthropic Tier 1 allows 60 requests/minute. A 5-minute dive generates an average of 5 total API calls (≤ 1.0 req/min), consuming **< 2%** of standard rate quotas.

#### Latency Masking Architecture

- **Slippy In-Game Calls:** 1.2–1.8s generation latency is masked behind the physical spearfishing animation (the eel biting, twisting, and pulling the lionfish off the spear tip). The engine state machine only switches to the new AI stance once the physical ingestion animation completes.
- **Dockside Audit Calls:** 1.0–1.5s latency is masked behind the harbor travel transition screen and cooler weigh-in audio sequence.

#### Context Windows & Fallback Resilience

All agent interactions use compact, single-turn structured schemas requiring **< 1K** context window overhead. If an API request times out or returns malformed JSON, the game engine falls back to deterministic local tables without interrupting gameplay:

- **Slippy** reverts to hardcoded FSM retreat timers.
- **The dock** applies standard arithmetic value formulas.
