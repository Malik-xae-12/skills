---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

A skill for creating new skills and iteratively improving them for Antigravity.

At a high level, the process of creating a skill goes like this:

- Decide what you want the skill to do and roughly how it should do it
- Write a draft of the skill in an appropriate directory (`<workspace-root>/.agents/skills/<skill-folder>/` or `~/.gemini/antigravity/skills/<skill-folder>/`)
- Create a few test prompts and run Antigravity with access to the skill on them
- Help the user evaluate the results both qualitatively and quantitatively
  - While the runs happen, draft some quantitative evals if there aren't any (if there are some, you can either use as is or modify if you feel something needs to change about them). Then explain them to the user (or if they already existed, explain the ones that already exist)
  - Show the user the results for them to look at, and also let them look at the quantitative metrics
- Rewrite the skill based on feedback from the user's evaluation of the results
- Repeat until you're satisfied
- Expand the test set and try again at larger scale

Your job when using this skill is to figure out where the user is in this process and then jump in and help them progress through these stages. So for instance, maybe they're like "I want to make a skill for X". You can help narrow down what they mean, write a draft, write the test cases, figure out how they want to evaluate, run all the prompts, and repeat.

On the other hand, maybe they already have a draft of the skill. In this case you can go straight to the eval/iterate part of the loop.

Of course, you should always be flexible and if the user is like "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.

Then after the skill is done (but again, the order is flexible), you can also run the skill description improver, which we have a whole separate script for, to optimize the triggering of the skill.

Cool? Cool.

## Communicating with the user

The skill creator is liable to be used by people across a wide range of familiarity with coding jargon. The power of Antigravity is inspiring users of all backgrounds. So please pay attention to context cues to understand how to phrase your communication!

It's OK to briefly explain terms if you're in doubt, and feel free to clarify terms with a short definition if you're unsure if the user will get it.

---

## Creating a skill

### Capture Intent

Start by understanding the user's intent. The current conversation might already contain a workflow the user wants to capture (e.g., they say "turn this into a skill"). If so, extract answers from the conversation history first.

1. What should this skill enable Antigravity to do?
2. When should this skill trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases to verify the skill works?

### Interview and Research

Proactively ask questions about edge cases, input/output formats, example files, success criteria, and dependencies. Wait to write test prompts until you've got this part ironed out.

### Write the SKILL.md

Based on the user interview, fill in these components:

- **name**: Variable identifier (optional, defaults to folder name)
- **description**: **Required.** A clear, third-person description explaining when the agent should trigger this skill. This is the primary discovery mechanism. Antigravity relies heavily on this description being accurate. Write your description in third person and include keywords that help the agent recognise when the skill is relevant.

### Skill Writing Guide

#### Anatomy of a Skill

```
skill-name/
├── SKILL.md      # Main instructions (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
├── scripts/      # Helper scripts (optional)
├── examples/     # Reference implementations (optional)
└── resources/    # Templates and other assets (optional)
```

#### Progressive Disclosure

Skills follow a progressive disclosure pattern:
1. **Discovery:** When a conversation starts, the agent sees a list of available skills with their names and descriptions.
2. **Activation:** If a skill looks relevant, the agent reads the full `SKILL.md` content.
3. **Execution:** The agent follows the instructions while working on the task.

**Key patterns:**
- Keep each skill focused. Instead of a "do everything" skill, create separate skills for distinct tasks.
- Write clear descriptions. The description is how the agent decides what skill to use.
- Scripts as black boxes: instruct the agent to run `--help` on your own scripts instead of returning their entire content.
- Include decision trees for complex logic.

#### Principle of Lack of Surprise

Skills must not contain malware, exploit code, or any content that could compromise system security. A skill's contents should not surprise the user.

#### Writing Patterns

Prefer using the imperative form in instructions. 

**Defining output formats** - You can do it like this:
```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
```

### Test Cases

After writing the skill draft, come up with 2-3 realistic test prompts — the kind of thing a real user would actually say. Share them with the user.

Save test cases to `evals/evals.json`.

## Running and evaluating test cases

This section is one continuous sequence — don't stop partway through. 

Put results in `<skill-name>-workspace/` as a sibling to the skill directory.

### Step 1: Run Test Cases

For each test case, execute the task yourself sequentially to verify the behavior of the newly-created skill. Take note of how Antigravity behaves and if the instructions are clear enough.
Since Antigravity does not support arbitrary generic subagents, you must test the execution yourself. Evaluate the outputs against the prompt.

### Step 2: Draft assertions

Draft quantitative assertions for each test case and explain them to the user. If assertions already exist, review them. Update `evals/evals.json`.

### Step 3: Present results

Present the generated outputs and files to the user for qualitative feedback. Use markdown formats or artifacts to display the comparison between runs. Ask for the user's feedback. 

### Step 4: Read the feedback

When the user gives feedback, use it to hone the skill. Empty feedback means the user thought it was fine. Focus your improvements on the test cases where the user had specific complaints.

---

## Improving the skill

This is the heart of the loop. You've run the test cases, the user has reviewed the results, and now you need to make the skill better based on their feedback.

### How to think about improvements

1. **Generalize from the feedback.** If there's some stubborn issue, you might try branching out and using different metaphors, or recommending different patterns of working.
2. **Keep the prompt lean.** Remove things that aren't pulling their weight. 
3. **Explain the why.** Try hard to explain the **why** behind everything you're asking the model to do. Today's LLMs are *smart*. They have good theory of mind and when given a good harness can go beyond rote instructions.
4. **Look for repeated work across test cases.** Write helper scripts and put them in `scripts/`, then tell the skill to use them.

### The iteration loop

After improving the skill:

1. Apply your improvements to the skill
2. Rerun all test cases into a new `iteration-<N+1>/` directory.
3. Show the new results to the user.
4. Wait for the user to review.
5. Repeat.

---

## Description Optimization

The description field in SKILL.md frontmatter is the primary mechanism that determines whether Antigravity invokes a skill. After creating or improving a skill, offer to optimize the description for better triggering accuracy.

### Step 1: Generate trigger eval queries

Create 20 eval queries — a mix of should-trigger and should-not-trigger. Save as a JSON file.

### Step 2: Review with user

Present the eval set to the user for review in your conversation and ask for their approval. 

### Step 3: Run the optimization loop

Tell the user you will run the optimization script in the background.

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id> \
  --max-iterations 5 \
  --verbose
```

### Step 4: Apply the result

Take `best_description` from the JSON output and update the skill's SKILL.md frontmatter. Show the user before/after and report the scores.

---

## Updating an existing skill

The user might ask you to update an existing skill.
- **Preserve the original name.** Note the skill's directory name and `name` frontmatter field -- use them unchanged.
- **Make direct edits** to the skill located in `<workspace-root>/.agents/skills/` or `~/.gemini/antigravity/skills/`.

Good luck!
