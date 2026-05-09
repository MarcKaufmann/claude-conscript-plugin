# Claude Conscript Plugin — Usage Guide

## Setup

1. Clone this repo:
   ```
   git clone https://github.com/MarcKaufmann/claude-conscript-plugin
   ```

2. Launch Claude Code with the plugin:
   ```
   cd ~/projects/your-study-project
   claude --plugin-dir ~/projects/claude-conscript-plugin
   ```

3. The plugin gives Claude 6 skills:
   - **create-study** — generate a new study (interactive)
   - **study-review** — QA an existing study
   - **upload** — deploy a study to a congame server
   - **conscript-coding** — framework reference (loaded automatically)
   - **racket-coding** — Racket language reference (loaded automatically)
   - **examples** — find and use example studies (loaded automatically)
   - **coding** — coding conventions (loaded automatically)

## First Time: Project Initialization

The first time you use `/create-study` in a project, Claude will ask
setup questions to learn your project's patterns. This creates a
`study-config.md` file that future invocations use automatically.

### Example session

```
You:  /create-study

Claude: I don't see a study-config.md in this project. Let me set up
        your project first.

        Do you already have a .md file describing the type of studies
        you build here?

You:  No.

Claude: OK, let me ask a few questions.

        1. What research area are these studies for?

You:  Bayesian belief updating. Participants observe signals and
      update their beliefs about an uncertain state.

Claude: 2. What does a typical study look like?

You:  5-10 rounds. Each round: show a signal, elicit a belief
      (probability 0-100). No treatment arms usually, but sometimes
      we vary the signal informativeness.

Claude: 3. Any standard requirements?

You:  Consent step at the start. Flat payment of £5 plus £2 bonus
      based on accuracy. No specific CSS.

Claude: 4. Naming conventions for study IDs?

You:  Descriptive names like "urn-beliefs", "signal-updating".

Claude: 5. Where is your congame repo?

You:  ~/projects/congame

Claude: Any other repos with example studies?

You:  ~/projects/many-designs has some, but the main one is this
      repo itself — there's a study in studies/urn-beliefs.rkt

Claude: Here's the config I'll save. [shows study-config.md]
        Look good?

You:  Yes.

Claude: Saved. You're ready to create studies. Run /create-study
        again with a description.
```

## Creating a Study

After initialization, use `/create-study` with a description:

### Example: Simple belief-updating study

```
You:  /create-study A study where participants see draws from one of
      two urns (60/40 vs 40/60 red/white) and report their belief
      about which urn after each draw. 5 rounds.

Claude: [reads study-config.md, finds urn-beliefs.rkt as reference]
        [generates a .rkt file with consent, instructions, 5-round
         loop, belief elicitation form, reveal step, bot models]

        Here's the study. [shows code]
        Want me to change anything?

You:  Add a treatment arm where participants see 2 draws per round
      instead of 1.

Claude: [updates with treatment assignment, conditional branching,
         modified draw logic]

You:  Looks good. Upload it.

Claude: [runs /upload]
```

### Example: Study with treatment arms

```
You:  /create-study Participants play a dictator game. Treatment:
      they either know or don't know the recipient's income level.
      Measure: amount shared out of £10.

Claude: [generates study with treatment assignment, conditional
         instructions, dictator game form, bot models for both
         generous and selfish types]
```

## Reviewing a Study

Use `/study-review` to QA an existing study:

```
You:  /study-review studies/urn-beliefs.rkt

Claude: [reads the file, asks about study design if unclear]
        [runs through checklist: structure, variables, treatment,
         flow, forms, matchmaking, bots, content]
        [reports pass/fail for each item]
        [offers to fix issues]
```

## Uploading a Study

Use `/upload` to deploy:

```
You:  /upload urn-beliefs

Claude: [finds studies/urn-beliefs.rkt, picks -with-admin variant]
        [runs: raco congame upload urn-beliefs-with-admin studies/urn-beliefs.rkt]
        Test at: http://127.0.0.1:5100/_anon-login/urn-beliefs-with-admin
```

## Tips

- **Always initialize first.** The `/create-study` skill works much
  better after initialization because it knows your project's patterns.
- **Point Claude to examples.** During initialization, list all repos
  with relevant studies. More examples = better generated code.
- **Iterate.** Generated studies are a starting point. Ask Claude to
  modify specific parts rather than regenerating from scratch.
- **Review before uploading.** Run `/study-review` on generated studies
  to catch issues.
- **Bot models matter.** Every study needs a `-with-admin` variant
  with bot models for testing. `/create-study` generates these
  automatically.

## TODO

- [ ] Add BSD license to this plugin (matching congame's license)
