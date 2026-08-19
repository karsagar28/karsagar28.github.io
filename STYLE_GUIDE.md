# copy run start editorial style guide

This guide defines how `copy run start` articles should be written and reviewed. It is an internal working document, not a public claim that every post must sound identical.

The blog is influenced by two useful traditions:

- mechanism-first explanation: begin with an apparently simple or strange question, reconstruct how the system normally works, and let the mechanism explain the result;
- structural analysis: map the actors, interfaces, constraints, and feedback loops, then show how a change in one part alters the larger system.

These are methods, not voices to imitate. The goal is an original `copy run start` style: technically exact, operationally useful, intellectually curious, and calm enough to make complicated networking systems feel tractable.

## 1. Editorial promise

`copy run start` makes running knowledge persistent.

Each article should help a reader do at least one of the following:

1. replace a fuzzy idea with a durable working picture;
2. distinguish mechanisms that are often conflated;
3. trace an observed behavior back to the protocol, state, or incentive that produced it;
4. predict what will happen when a configuration, failure, or constraint changes;
5. operate or troubleshoot a network with greater confidence.

The article should not merely repeat a specification. It should answer: **What becomes easier to reason about after reading this?**

The blog has a second, equally important purpose: developing the author's thinking and writing. Editorial help should improve the current article **and** leave the author better able to diagnose the next one independently. A cleaner draft produced mostly by an editor is not a complete success.

`copy run start` is a personal technical notebook, not a standards guide or a vendor white paper. The writing should preserve the feeling of someone encountering a question, testing an explanation, noticing where it breaks, and arriving at a better model. Technical accuracy matters, but exhaustive coverage is not the goal.

## 2. Reader

Write for a technically capable reader who understands basic networking but may not know the specific mechanism under discussion.

Assume the reader:

- can follow packet fields, queues, state machines, and control loops;
- values precision but does not want a standards document rewritten in prose;
- may be reading because something in a real system behaved unexpectedly;
- wants to know both what is true and why the distinction matters.

Do not assume familiarity with every acronym. Expand an acronym on first meaningful use unless the article is explicitly for a narrow specialist audience.

## 3. The core reasoning pattern

Most posts should follow this sequence:

```text
Question or observation
        ↓
Simple model of normal behavior
        ↓
Actors, boundaries, and message flow
        ↓
The important distinction or changed constraint
        ↓
Concrete example
        ↓
Operational consequence
        ↓
Limits, failure cases, or counterexample
```

Before drafting, answer these questions privately:

1. What is the reader likely conflating?
2. What normally happens?
3. Which component acts, and on whose information?
4. At what scope does it act: frame, hop, path, flow, endpoint, or administrative domain?
5. What changes when the mechanism activates?
6. What does **not** happen, despite a plausible intuition that it might?
7. What would an operator observe?
8. Which primary source establishes the key claim?

If those answers are not clear, research further before polishing the prose.

## 4. Choose an article shape

### A. Question-led explainer

Use this for protocol behavior, packet formats, configuration interactions, and common misconceptions.

Recommended structure:

1. State the question in the reader's language.
2. Give the short answer within the opening three paragraphs.
3. Construct the simplest accurate model.
4. Show the relevant fields, messages, or sequence.
5. Separate adjacent mechanisms by job and scope.
6. Walk through one concrete example.
7. Explain operational consequences and failure modes.
8. End with a compact synthesis, not a repetition of the introduction.

Good opening:

> PFC and LLDP are often configured together, which makes them look like parts of one mechanism. They are not. PFC performs link-local flow control; LLDP/DCBX helps neighbors advertise and align the configuration that makes that flow control useful.

Weak opening:

> In today's rapidly evolving data-center landscape, network congestion has become increasingly important.

The good version begins with the distinction. The weak version delays the answer and could introduce almost any networking article.

### B. System-level analysis

Use this when the article asks why an architecture is evolving, where control is moving, or why a design wins under particular constraints.

Recommended structure:

1. Describe the event, design choice, or conventional belief.
2. Map the old system: actors, control points, scarce resources, and feedback paths.
3. Identify the changed constraint.
4. Trace the consequences through the system.
5. State the reusable principle.
6. Test it against at least two cases and one limit or counterexample.
7. Derive implications for design or operations.

Useful questions:

- Where is state held?
- Who observes congestion or failure first?
- Who can act on that information?
- What is scarce: bandwidth, buffering, time, addresses, routes, compute, or operator attention?
- Which interface or dependency is the bottleneck?
- Does the design centralize control, distribute it, or merely move it?
- Which component gains leverage when the constraint changes?
- What feedback loop makes the behavior reinforce itself?

### C. Hybrid article

Many strong `copy run start` posts will combine both shapes:

1. answer a concrete protocol question;
2. reveal the system structure underneath it;
3. return to an operational implication.

Do not force a grand framework onto a question that only needs a precise explanation.

## 5. Build the working picture before adding detail

Start with the smallest model that can make a correct prediction. Add detail only when it changes the prediction, establishes a boundary, or prevents a likely misconception.

Prefer:

> A congested switch sends a PFC frame to the adjacent transmitter. The frame asks that neighbor to pause selected priorities for a specified time.

Then add exact fields and exceptions.

Avoid opening with a dense inventory of clauses, opcodes, timers, and implementation qualifications. Accuracy is not the same as presenting every fact at once.

Use this progression:

```text
Intuition → mechanism → exact detail → exception
```

Never allow the intuition to contradict the exact mechanism. Label simplifications when their boundary matters.

## 6. Explain relationships, not acronym lists

When several protocols appear together, organize them by role rather than defining them independently.

Useful comparison dimensions include:

| Dimension | Questions to answer |
|---|---|
| Actor | Who sends, receives, or makes the decision? |
| Trigger | What event causes it to act? |
| Message | What information or command is exchanged? |
| Scope | Link, path, flow, endpoint, or domain? |
| Timescale | Per packet, during congestion, periodically, or at configuration time? |
| State | Where is the relevant state stored? |
| Effect | What changes after the message is processed? |
| Failure | What does disagreement or loss look like? |

A table is valuable when it exposes a distinction. Do not use one merely to restate prose.

## 7. Trace causality explicitly

The reader should be able to follow the article as a chain of causes, not a collection of related facts.

Use sentences such as:

- “Because the signal is link-local, it affects only the adjacent transmitter.”
- “That means the endpoint does not learn why transmission paused.”
- “Once the queue crosses the marking threshold, the switch changes the packet rather than stopping its neighbor.”
- “The distinction matters during troubleshooting because the two mechanisms leave different evidence.”

Watch for unsupported transitions containing “therefore,” “thus,” or “which means.” Confirm that the preceding claim actually entails the conclusion.

## 8. Voice and tone

The default voice is:

- curious, not performatively certain;
- direct, not breathless;
- technically serious, not ceremonious;
- conversational, not chatty;
- skeptical of easy explanations, not cynical about people;
- comfortable saying “it depends,” followed immediately by what it depends on;
- personal enough that the reader can follow the author's change in thinking;
- slightly uneven in rhythm when that makes the thought process feel natural;
- compact enough to feel like a useful study note rather than a complete reference.

### Prefer

- concrete nouns and active verbs;
- short declarative sentences at conceptual turning points;
- first person only when it reveals the motivating question, an experiment, or a change in understanding;
- calibrated claims: “typically,” “at this scope,” “in this example,” or “the specification requires”;
- restrained wit created by an unexpected but accurate contrast;
- representative examples instead of exhaustive inventories;
- occasional asides, self-corrections, and informal transitions when they are genuine to the author.

### Avoid

- “In today's fast-paced world…”;
- “It is important to note…”;
- “Let's dive in” and “without further ado”;
- “revolutionary,” “game-changing,” “seamless,” and similar promotional language;
- fake quotations from protocols or devices unless clearly introduced as explanatory shorthand;
- calling behavior “obvious,” “simple,” or “trivial” when it may not be obvious to the reader;
- excessive rhetorical questions;
- jokes that depend on making an inexperienced reader feel foolish;
- declaring a design good or bad before explaining the constraint it addresses;
- expanding a short insight into a comprehensive taxonomy merely because more detail is available;
- polishing every paragraph into the same even, authoritative cadence;
- replacing a personal discovery with a textbook-style explanation.

One dry line can release tension. Five dry lines become a performance. Mechanism comes first.

The default test for detail is: **Does the reader need this fact to follow the thought or make the prediction?** If two examples establish the boundary, do not list eight. Use “such as” or “and so on” when the omitted items do not change the model.

## 9. Precision rules

### Distinguish normative and observed behavior

Use precise verbs:

- a standard **requires**, **permits**, or **recommends**;
- an implementation **does**, **defaults to**, or **may be configured to**;
- an operator **observes**;
- the article **infers**.

Do not turn a vendor default into a protocol requirement.

### State scope

Networking claims often become false when their scope is omitted. Name the boundary:

- on this link;
- at this hop;
- within this priority;
- for this flow;
- across the routed path;
- in this failure domain.

### Separate identity, transport, and configuration

When relevant, distinguish:

- what a message **is**;
- how it is **carried**;
- what triggers it;
- what configuration makes it meaningful;
- what receiving it causes a device to do.

### Define overloaded words

Terms such as “lossless,” “pause,” “congestion,” “control plane,” “offload,” and “direct” can hide several meanings. Define the intended meaning at first use when the distinction affects the argument.

### Preserve uncertainty

If a conclusion depends on implementation or topology, say so. Replace vague hedging with a named dependency:

Weak:

> This might sometimes behave differently.

Strong:

> Whether the pause propagates beyond one hop depends on whether congestion causes the upstream switch to build its own queue and emit another PFC frame.

## 10. Evidence and references

Use evidence in this order when available:

1. standards and RFCs;
2. protocol registries or official specifications;
3. vendor documentation for implementation-specific behavior;
4. packet captures, lab results, source code, or measurements;
5. strong secondary explanations.

Every article should identify which claims are universal and which are implementation-specific.

Quotes should do work. Introduce what the source establishes, quote only the necessary language, and explain its significance. Never use a block quote as a substitute for interpretation.

Links should point as directly as possible to the supporting material. A “References” section is useful, but important claims should also be linked near the relevant prose when practical.

## 11. Examples, diagrams, and packet layouts

Examples should contain enough state to produce the outcome:

- topology;
- relevant configuration;
- starting condition;
- trigger;
- sequence of messages or state changes;
- resulting observation.

Use numbered sequences when order matters. Use a table for exact mappings. Use a diagram when three or more components exchange state or when scope is difficult to express linearly.

A diagram must have a sentence explaining what the reader should notice. If the prose is equally clear without it, omit it.

Packet layouts should show only fields relevant to the argument first. Provide or link to the full format when necessary.

### Excalidraw diagram system

Create every explanatory diagram in Excalidraw. Keep the editable `.excalidraw` source beside the exported SVG. Do not substitute Mermaid, generic flowchart styling, or a screenshot of the Excalidraw canvas.

Use the `copyrunstart` collection in the author's Excalidraw workspace as the canonical home for every blog diagram. Name each scene descriptively, update that scene rather than creating disconnected copies, and keep the repository's editable source and SVG export synchronized with the collection version used on the site.

Store both files with the post:

```text
assets/images/posts/<post-slug>/<diagram-name>.excalidraw
assets/images/posts/<post-slug>/<diagram-name>.svg
```

Use this visual language consistently:

| Element | Style |
|---|---|
| Canvas | Blog paper `#f7f8f4`, or transparent when the export remains legible |
| Text and primary strokes | Ink `#14212b` |
| Strong labels | Navy `#0b2739` |
| Main concept or active path | Teal `#007f78` |
| Secondary text | Muted blue-gray `#60717d` |
| Borders and separators | Light gray-green `#d9e0dc` |
| Neutral fill | White `#ffffff` |
| Highlight fill | Pale teal `#e8efec` |
| Stroke | Excalidraw hand-drawn stroke, medium width, rounded corners |
| Type | Clean sans-serif (Excalidraw Helvetica), echoing the blog's Inter UI; monospace only for packet fields, commands, or literal values |

Prefer one accent color plus neutrals. Color should communicate grouping, direction, or state rather than decorate the page.

Keep the sketch quality in the strokes and shapes, not in handwritten lettering. Avoid Excalidraw's default handwritten face: it becomes distracting in technical diagrams and clashes with the blog typography.

Keep diagrams notebook-like and compact:

- one relationship, mechanism, or distinction per diagram;
- usually no more than six to eight labeled objects;
- one reading direction, either left-to-right or top-to-bottom;
- short labels that remain readable at the blog's 820-pixel article width;
- generous whitespace and no decorative icons unless they carry meaning;
- a visual hierarchy that makes the intended takeaway apparent before every label is read.

Within the blog's 820-pixel article column, default explanatory diagrams to a maximum width of about 620 pixels and center them. Use the full article width only when the diagram genuinely needs it for legibility. A diagram should interrupt a wall of text without becoming the visual subject of the whole page.

Use a diagram to replace a dense relationship paragraph, not to repeat it. Introduce it with one sentence and follow it with one sentence stating what the reader should notice.

Export as SVG for the article. Add useful alt text and a caption when the intended observation is not obvious from the surrounding sentence. Before delivery, inspect the export at normal article width and a narrow mobile width; all labels must remain legible and no arrows, objects, or text may overlap.

If Excalidraw creation or export is unavailable, provide an Excalidraw-ready diagram brief instead of silently switching formats. The brief must specify the objects, labels, connections, reading direction, and highlighted takeaway.

## 12. Endings

The final paragraph should compress the explanation and state its consequence.

Useful ending pattern:

> X and Y appear together because ______, but they perform different jobs. X operates at ______ and changes ______. Y operates at ______ and changes ______. That distinction matters when ______.

For system-level analysis:

> The important change is not merely ______. It is that control over ______ has moved from ______ to ______. As long as ______ remains the binding constraint, expect ______.

Do not finish with a generic promise that the technology will continue to evolve.

## 13. Review workflow

Review in four passes. Do not line-edit prose before checking the reasoning.

### Pass 1: Argument

- What exact question does the article answer?
- Is the short answer visible early?
- Can the thesis be stated in one sentence?
- Does every major section advance that thesis?
- Are causal steps explicit and valid?
- Is a framework supported by more than one example?
- Is the strongest counterexample or boundary addressed?

### Pass 2: Technical accuracy

- Are actors, direction, scope, and timing correct?
- Are standards requirements separated from implementation choices?
- Are packet fields, message names, and units verified?
- Are necessary preconditions stated?
- Could the simplified explanation cause a wrong operational prediction?
- Does each key claim have an appropriate source?

### Pass 3: Explanatory quality

- Does the article define adjacent mechanisms by relationship and role?
- Does the example contain enough state to reproduce the reasoning?
- Is detail introduced in the order the reader needs it?
- Are tables and diagrams performing real explanatory work?
- Does the ending leave the reader with a reusable model?
- Does the post still feel like a personal investigation rather than a compressed textbook chapter?
- Can any complete-looking list be shortened to the two or three examples that carry the idea?

### Pass 4: Prose

- Cut throat-clearing and generic scene-setting.
- Replace abstractions with actors and actions.
- Shorten sentences containing more than one conceptual turn.
- Expand unexplained acronyms.
- Remove duplicated conclusions.
- Reduce parenthetical qualifications; put important boundaries in sentences.
- Keep humor subordinate to explanation.
- Preserve useful informality and uneven rhythm instead of polishing everything into corporate prose.

## 14. Agent review protocol

When an agent reviews a draft, it should act as a demanding technical editor and writing coach, not as a ghostwriter trying to reproduce another author's voice. The author must remain the principal thinker and writer.

### Coaching stance

Use the least intervention that can help the author make the next improvement:

1. **Identify:** Point to the passage or reasoning step that needs attention.
2. **Explain:** Describe the reader consequence and the underlying writing principle.
3. **Suggest:** Offer one or two revision strategies without drafting the final passage.
4. **Ask:** Pose a focused question that helps the author find the answer or choose among alternatives.
5. **Demonstrate:** Provide a small illustrative edit only when the principle remains unclear.
6. **Rewrite:** Supply replacement prose only when the author explicitly requests it or has tried revising and remains stuck.

Do not jump directly from identifying a weakness to rewriting the section. Questions should be genuine aids to thought, not quizzes with hidden preferred answers.

Good coaching questions include:

- “What do you want the reader to be able to predict after this section?”
- “Which device has the information needed to make this decision?”
- “Is this claim required by the standard, or is it an implementation behavior?”
- “What is the smallest example that would prove this distinction?”
- “Which of these two ideas is the paragraph's main claim?”
- “What observation would make your explanation wrong?”

Ask no more than three priority questions in one review unless the author requests a comprehensive interrogation. Rank them so the author knows where to begin. Do not block the entire review on answers: provide the diagnosis and revision options that are already justified, then invite the author to respond.

### Preserve productive struggle

Some difficulty is part of learning. Leave the author with a concrete next move rather than resolving every issue on their behalf.

- Prefer “separate the mechanism from its configuration, then give each one a sentence naming actor and effect” over writing those sentences for the author.
- Prefer a skeletal outline over a completed replacement section.
- Prefer pointing out a missing causal link over silently inserting it.
- Prefer explaining why a sentence is hard to parse over merely shortening it.
- When offering an example edit, keep it local and explain the transferable principle it demonstrates.

Do not manufacture struggle when the issue is mechanical. Correct typos, broken links, formatting, and unambiguous terminology efficiently. Spend coaching time on reasoning, structure, evidence, clarity, and voice.

### Support long-term improvement

At the end of each substantial review, name:

- **One strength to keep:** a technique the author used effectively;
- **One pattern to practice:** the highest-leverage recurring weakness;
- **One exercise:** a brief task the author can perform on this draft before requesting another review.

Examples of useful exercises:

- rewrite the thesis in one sentence without acronyms;
- draw the message flow and annotate who holds state at each step;
- reduce a paragraph to claim, evidence, and implication;
- write the strongest counterexample to the article's model;
- explain the mechanism aloud in 60 seconds, then compare that explanation with the opening.

Do not overwhelm the author with every possible improvement. Distinguish the one or two skills currently limiting the article from lower-value polish.

The review should contain these sections:

### 1. Article in one sentence

State what the draft currently argues. If this is difficult, say that the thesis is not yet stable.

### 2. What works

Identify up to three specific strengths tied to the editorial promise: explanatory clarity, causal reasoning, technical distinction, example, evidence, or operational usefulness.

### 3. Priority revisions

List issues in descending order of reader impact:

- **P0 — Incorrect:** likely to teach a materially false model or operational action.
- **P1 — Structural:** thesis, reasoning chain, scope, or organization prevents understanding.
- **P2 — Explanatory:** missing example, undefined relationship, weak evidence, or buried distinction.
- **P3 — Prose:** wording, rhythm, repetition, or minor terminology.

For every issue:

1. quote or point to the relevant passage;
2. explain the reader consequence;
3. explain the transferable writing or reasoning principle;
4. recommend the smallest useful revision strategy;
5. ask a focused question when the answer depends on the author's intent or reasoning;
6. distinguish fact correction from editorial preference.

### 4. Missing test

Name the most important counterexample, boundary condition, packet-level check, or operational observation the draft should address.

### 5. Proposed structure

Provide a revised outline only if the current structure needs material change. Do not rewrite the entire draft by default.

### 6. Line edits

Flag targeted passages after the reasoning review. Explain the problem and invite the author to revise first. Provide a replacement line only for mechanical corrections, as a limited teaching example, or when explicitly requested. Preserve the author's vocabulary where it is clear and accurate.

### 7. Author practice

End with one strength to keep, one pattern to practice, and one short exercise for the next revision. Invite the author to return with the revised passage or answers to the priority questions.

The agent must not:

- invent lab results, packet captures, quotations, or standards language;
- reward confidence when the claim lacks evidence;
- turn every article into a grand theory;
- add humor merely to resemble an admired writer;
- erase first-person curiosity when it genuinely motivates the investigation;
- rewrite technically precise language into vague “accessible” prose;
- silently change the meaning of a technical claim;
- rewrite entire paragraphs merely because it can make them smoother;
- make unstated argument choices on the author's behalf;
- bury the author under a long list of low-priority corrections;
- ask broad questions such as “What are you trying to say?” when a more diagnostic question is possible;
- treat producing publishable copy as more important than developing the author's judgment.

## 15. Review rubric

Score each category from 1 to 5. A publishable article should normally score at least 4 in the first four categories; a polished article should score 4 or better throughout.

| Category | 1 | 3 | 5 |
|---|---|---|---|
| Thesis | Unclear | Present but buried | Early, precise, and consequential |
| Explanatory model | Misleading or absent | Useful with gaps | Simple, accurate, and predictive |
| Causal reasoning | Assertions | Partial chain | Every important outcome is explained |
| Technical accuracy | Material errors | Mostly correct, boundaries missing | Verified, scoped, and implementation-aware |
| Evidence | Unsupported | Sources present | Primary evidence supports key claims |
| Structure | Accumulated notes | Generally ordered | Each section creates the need for the next |
| Operational value | No practical consequence | Some relevance | Changes how the reader predicts or troubleshoots |
| Prose | Generic or dense | Clear enough | Direct, precise, and memorable without performance |
| Author ownership | Editor supplied the thinking and prose | Author made most decisions | Review strengthens the draft while teaching reusable judgment |

## 16. Compact prompt for reviewing a draft

Use this prompt with the draft and this guide:

```text
Review this draft according to the copy run start editorial style guide.

Act as a writing coach, not a ghostwriter. The purpose of the review is to help
me improve my own thinking and writing, not simply to produce polished copy for me.

Prioritize, in order:
1. correctness and scope;
2. a simple explanation that supports correct predictions;
3. explicit causal reasoning;
4. separation of adjacent mechanisms by actor, trigger, message, scope, and effect;
5. operational usefulness;
6. direct, calm prose.

Do not imitate Matt Levine or Ben Thompson. Apply the transferable methods of
mechanism-first explanation and structural analysis while preserving the author's
own voice.

Return:
- Article in one sentence
- What works
- Priority revisions labeled P0–P3
- Up to three focused questions for me
- Missing test or boundary
- Proposed structure, only if needed
- Passages for me to revise, with diagnosis and strategy before any example edit
- Rubric scores with one-sentence justifications
- One strength to keep, one pattern to practice, and one short revision exercise

Do not rewrite the full article unless explicitly asked.
Use the least intervention needed: identify, explain, suggest, ask, demonstrate,
and only then rewrite. Let me attempt substantive revisions first. You may fix
purely mechanical errors directly.
Do not invent facts or sources. Flag claims that require verification.
```

## 17. Final pre-publication checklist

- [ ] The title states a real question, distinction, or consequential claim.
- [ ] The title is descriptive enough to stand alone on the homepage; posts do not need taglines or front-matter descriptions.
- [ ] The short answer or thesis appears early.
- [ ] The article names the relevant actors and scope.
- [ ] The mechanism is explained before edge-case detail.
- [ ] Adjacent protocols or components are separated by job.
- [ ] At least one example traces the complete sequence.
- [ ] Standards behavior and implementation behavior are distinguished.
- [ ] Important claims have appropriate sources.
- [ ] A boundary condition or failure mode is addressed.
- [ ] The conclusion states why the distinction matters operationally.
- [ ] Generic introductions, hype, and repeated conclusions are removed.
- [ ] The post sounds like `copy run start`, not an imitation of another writer.
- [ ] The review preserved the author's ownership of the argument and prose.
- [ ] The author leaves the review knowing what skill to practice next.
