# Boris Cherny on Cutting 80% of Claude Code's Prompt



> This guide summarizes the key concepts, models, and steps from the video.

## Why Claude Code Deletes Its Own System Prompt

Boris Cherny created Claude Code at Anthropic. In this Y Combinator talk, recorded one day after Opus 5 shipped, he explained that the Claude Code harness is never finished. The team is constantly adding and removing instructions, and every new model release triggers a large rewrite.

### The Harness Changes With Every Model

The reason is simple. Every model is different, so an instruction written for a model three months ago may not transfer to the next one at all.

- The team changes the system prompt on every model release.
- They change the set of tools available to the model.
- They change the prompts attached to each individual tool.
- They regularly unship tools and delete harness code entirely.

### What Changed With Opus 5

A large portion of the old system prompt existed to correct behaviors the model should have known but did not. Opus 5 does those things on its own, so the corrections became dead weight and the team removed more than 80% of the system prompt.

- Opus 5 scored 30% on Arc AGI 3. Previous best scores were in the low single digits to low teens.
- Combined with auto mode, it can run for days, weeks, or months without stopping.
- It no longer needs scaffolding commands to keep going, because it understands it needs to finish the task.
- Boris also reported that the model no longer appears to be prompt injectable.

## The Three Layers Behind Prompt Injection Resistance

Prompt injection is when the model reads an instruction from an untrusted source, such as a web page, and follows it. Boris described three layers that together made this very hard to demonstrate.

1. A well aligned model, the product of roughly three years of alignment research.
2. A prompt injection classifier run across all traffic, built on mechanistic interpretability work that watches which neurons activate when injection occurs.
3. The auto mode classifier, which adds a third independent check.

## What Is Left in the Harness

After all the deletion, Boris said almost all of the remaining Claude Code code is about safety, permissions, and static analysis, plus a body of interface code. Most of the rest has already been unshipped.

## The Ablation Method: How to Delete and Rebuild

Ablation is a research term. You remove something to measure its impact. Applied to prompts, it means deleting the entire system prompt and then bringing it back one line at a time to find out what each line is actually doing. Boris described it as an eval where you delete things to figure out the impact.

### The Advice for Everyday Users

Boris gave a direct recommendation to people who use Claude Code but do not build agentic products.

Roughly every six months, and especially on a major model release, delete your configuration and see what happens.

- Delete your `CLAUDE.md` file.
- Delete your skills.
- Delete your hooks.
- Then use the tool and observe what the model does without them.

### The Four Step Rebuild

Deleting is only the first half. The rebuild is what determines whether the new configuration is smaller or just different.

1. Delete the configuration.
2. Use it on real work, not hypothetical tests. Run the actual product or the actual codebase.
3. Watch for where it does well and where it stumbles.
4. Add an instruction back only after you see the model stumble on the same thing repeatedly.

Two reasons for the discipline in step four: you are a poor predictor of which instructions the model actually needs, and every line you keep is re-read by the model on every single use.

> **Core principle:** Do not guess what the model needs. Make it prove the need by failing at the same thing more than once.

## Ways to Test This Yourself

Boris named two mechanisms for running your own ablation.

- Pass a system prompt flag when launching Claude Code to substitute any system prompt you want.
- Set the environment variable `CLAUDE_CODE_SIMPLE=1`, a largely undocumented option that strips all system prompts, including the prompts attached to tools.

Anthropic uses the second option internally as its ablation baseline. Boris noted a counterintuitive finding: the model is slightly more intelligent without the prompts. The prompts are kept anyway, because they make Claude Code behave the way a person using it as a product would expect.

## What Happens to Evals

Evals are more durable than prompts, but not permanently so. Boris said an eval might survive one to three model generations before the models saturate it, at which point it gets thrown away and replaced.

- Keep appending to your eval set as you find new failures.
- Expect to retire evals once models max them out.
- Build new evals from where you actually watched the model struggle.

## Stop Over Specifying the Task

Boris identified over specification as one of the most common mistakes he sees, and said it is especially common among people who have been engineering for years or decades.

### The Failure Mode

People write instructions that dictate the exact sequence of moves: do this, then this, then this, then this.

That approach fit older models. It does not fit modern ones, and it prevents the model from finding a better route than the one you imagined.

Boris traced the habit to how software used to be built. You designed the system up front, wrote a large suite of unit tests, and treated re-architecture as a project measured in months or years. Models do not work like that.

### What to Write Instead

The replacement shape is higher level and much shorter.

- Describe the task.
- Describe the guardrails.
- Describe the exit criteria, meaning how the model knows it is finished.
- Let the model work, and check back later.

He also recommended aiming higher than feels comfortable. Give the model a task slightly harder than you think it can handle, because the ceiling moves with every release.

## Think of It as Organic, Not Mechanical

Boris described working with a model as closer to getting to know a living creature than configuring a system. Each generation behaves differently and has a slightly different personality, so you spend time learning it and then adjust the harness around what you learned. He said the appropriate level to pitch instructions is roughly how you would brief a coworker.

## Verification Is What Most People Get Wrong

Asked what skill matters now that prompt engineering has faded as a job title, Boris pointed at two things: giving Claude a task that seems slightly too hard, and making it possible for Claude to verify its own work along the way. He called verification the single most important thing that people do not get right.

### The Swift Rewrite Example

Boris wanted to know what the Claude desktop app, built in Electron, would feel like as a native application. He ran the experiment through Claude Tag, which is Claude running inside Slack.

1. He asked whether it had access to a macOS runner on GitHub. It said no, so he connected one, giving it the ability to start a Mac virtual machine.
2. He created an empty repository for the Swift rewrite and asked whether it had access. It said no, so he granted access.
3. He gave it one instruction: rewrite the Electron app in Swift, run the Electron app in the Mac virtual machine, screenshot it, compare it pixel by pixel to the Swift version, and do not stop until you are done.

At the time of the talk the task had been running for more than two weeks and was still going. Boris estimated it had spawned thousands to tens of thousands of agents. Claude also decided on its own to create a Slack channel and live blog its progress with screenshots every few minutes.

### Why That Prompt Worked

The prompt was short and contained no clever technique. What it did contain was a complete verification loop: a way to produce output, a way to observe the target, a way to compare the two, and a condition for stopping.

- **Produce:** rewrite the app in Swift.
- **Observe:** run the original in a virtual machine and screenshot it.
- **Compare:** check it pixel by pixel against the new version.
- **Stop condition:** do not stop until you are done.

> **Verification rule:** Give the model a way to check its own output the way you would check it yourself. Without that, it gets stuck. With it, it can run for weeks.

## There Is No One Weird Trick

Boris was asked what separates the top 1% of Claude Code users. His answer was to stop looking for a trick, and he specifically advised against taking cues from social media influencers on the subject.

The method he described instead is a loop. Give it a task that is too hard. Give it the tools to verify the work. Watch where it struggles. Fix that specific struggle. Repeat.

For fixing a struggle, he named three options depending on the cause.

- Better prompting, if the instruction was unclear.
- A skill, if the model needs a repeatable procedure.
- An MCP server, if the model is missing context it cannot reach.

## Product Overhang and Unhobbling

These are two sides of one idea, and Boris presented them as the largest source of untapped opportunity for builders right now.

- **Product overhang:** the model can already do something valuable, but no product exists that lets it express that ability.
- **Hobbling:** the product actively gets in the way of what the model could do.

Unhobbling means removing the scaffolding that constrains the model so the existing capability can surface. Boris stressed that this is about capabilities in today's models, not future ones.

### How Claude Code Was Born

Claude Code itself is the clearest example. When Boris started building it, the best available coding model was Sonnet 3.5. The coding products of that era offered single line autocomplete, some multi line autocomplete, and read only chat about a codebase.

His judgment was that the model could already write an entire file at a time, and nothing on the market allowed it to. The design answer was to strip away scaffolding and hand the model the simplest possible harness with full terminal access.

### The Bun Rewrite

Claude Code runs on Bun, an open source JavaScript runtime and a faster alternative to Node.js. Bun was written in Zig, a low level systems language that requires manual memory management, which makes memory leaks easy to introduce.

- Initially the Bun team used Claude to fuzz the codebase and trigger memory leaks, finding them one case at a time.
- Jared on the Bun team kept throwing a harder test at each new model generation: rewrite the whole thing.
- Starting with Fable, the model could do it. He defined success using the existing Bun and Node.js test suites.
- One prompt, run as a dynamic workflow, rewrote the codebase from Zig to Rust over 11 days.
- The codebase is over 100,000 lines. Boris said the same work would have taken skilled engineers well over a year.
- It was not fully one shot. There was steering along the way, but previous models could not do it even with steering.
- The result is in production and is what Claude Code runs on today.

### Teaching a Model to Draw

A second example came from internal play rather than a business problem. Someone at Anthropic discovered that if you give Opus 5 access to OpenCV, a computer vision library, and ask it to draw, it produces credible portraits, animals, and landscapes.

The model was never trained to draw. Boris called this an elicitation gap: the capability exists, and asking correctly is all it takes to reach it. His hypothesis is that dozens or hundreds of comparable discoveries are sitting unclaimed.

> **Practical method:** Keep a list of problems that models failed at before, and re-run them on every new release. The previous model's failure tells you nothing about the current one.

## Orchestrating Thousands of Agents

Boris described two distinct mechanisms for putting large amounts of compute behind a single intent, and the distinction between them matters.

### Dynamic Workflows

A dynamic workflow takes one large task and breaks it into coordinated stages run by many agents. Boris said the way to trigger it is simply to say use a workflow, and Claude handles the rest.

- The workflow runs in a sandbox, using a virtual machine started inside the Bun runtime.
- It does not just run one agent, or ten parallel agents. It orchestrates in stages.
- A first wave of agents does an initial pass across the work.
- A second stage verifies or summarizes what the first wave produced.
- A third stage fans out again, and so on, until the task is complete.

Good fits include rewriting a codebase, running deep analysis over complicated data, and building a complex feature that spans multiple stages and dozens of pull requests.

### An Algebra for Agents

Boris comes from a functional programming background and designed the system around composition. There is a way to run agents in sequence and a way to run them in parallel, and Claude has tools to combine these inside the sandbox to use tokens efficiently on very complex work.

### A New Form of Test Time Compute

Model capability has historically scaled with the size of the neural network, the amount of training data, and the number of flops used in training. Test time compute was added more recently, and refers to how many tokens the model generates while working.

Boris framed dynamic workflows as a new way to orchestrate test time compute, and a way to dramatically increase how much of it goes into a single hard task. He noted this has not been written about much.

## Loops and Routines

Loops and routines solve a different problem from dynamic workflows. They handle one repetitive task run over and over on a schedule, rather than one large task split into pieces.

- A loop is essentially a cron job running Claude locally.
- A routine is the same thing running in the cloud, so you can close your laptop.
- Each run does not share context with the previous run, though it may share memory.
- They can be scheduled every five minutes, every hour, or every day.

## Codebases That Maintain Themselves

Anthropic now has Claude maintaining its own products. The team set up a Slack channel and started routines against the Claude Code CLI, the iOS app, the Android app, and the desktop app.

### The Routines They Run

Boris emphasized that each of these is a single prompt, often one sentence long. The model works out the implementation details itself.

- **Clean up dead code.** Runs daily, uses static and dynamic analysis, and opens a pull request removing what it finds. Boris noted nobody prompted it to use those analysis methods, it worked that out on its own.
- **Ship finished experiments.** Finds experiments already rolled out to 100%, deletes the experiment scaffolding from the codebase, and ships the result.
- **Write missing tests** for areas of the codebase with weak coverage.
- **Delete useless tests,** including low value tests added by older models or by people over time.
- **Abstraction police.** Finds near duplicate abstractions that drifted apart across a large codebase and unifies them back into one.

### The Scale of It

There are now roughly 20 to 30 of these routines running daily across all their codebases, launching hundreds and sometimes thousands of agents a day.

Boris estimated this covers work that used to take dozens or hundreds of engineers. He said they are not fully there yet, but they are on the path to automating app maintenance entirely, which frees engineers to ship new product and talk to users.

> **Pattern worth copying:** The highest leverage routines are maintenance chores that are obvious, repetitive, and easy to skip. One sentence each, run daily, no human trigger required.

## The Mindset That Separates the Best Users

Boris returned repeatedly to one theme: working with models has become an empirical science rather than a theoretical one.

### Forget Your Priors

- Set aside what you learned about how previous models behaved.
- Set aside assumptions from computer science theory about what should be hard.
- Try the task, watch where it struggles, and adjust based on what you observed.
- Stay open to retrying ideas that failed before, because the reason they failed may be gone.

He described over engineering as the recurring failure mode, and unlearning it as a genuine journey for experienced builders.

## Where Coding Is Not Solved Yet

Boris has said publicly that coding is solved, and he added a caveat here: it is solved for the kind of coding he does, not for everyone. He named the areas where Claude still struggles.

- Very deep systems codebases.
- Distributed systems.
- Detailed visual verification, such as catching something off by a pixel. Opus 5 was a large leap in vision and computer use, but it is not perfect.

When he polled the room, a meaningful number of founders said 100% of their code is now written by agents, with a similar number above 50%.

## What Is Still Worth Learning by Hand

Boris learned to program on a TI-83 calculator in middle school, writing BASIC programs to solve algebra problems on tests, then teaching himself assembly when calculus made the problems harder. He even wrote an online guide for it that is still up. The point of the story is that he always learned in service of a concrete problem.

His advice to students is to learn computer science, but to weight the surrounding skills just as heavily, because those are what make the technical ability valuable.

- Applying knowledge to a real problem rather than studying it in isolation.
- Building products and startups.
- Developing design sense.
- Developing business sense.
- Learning data science.
- Learning how to talk to users.

## Key Takeaways

1. **Configuration decays.** Every instruction you write is a correction for a specific model's weakness, and it becomes dead weight when the next model no longer has that weakness.
2. **Delete on a schedule.** Roughly every six months, and on major model releases, clear your `CLAUDE.md`, skills, and hooks, then see what the model does without them.
3. **Rebuild from evidence, not prediction.** Add an instruction back only after the model has stumbled on the same thing more than once. You are bad at guessing which lines it needs.
4. **Over specifying is the common mistake.** Give the task, the guardrails, and the exit criteria, then let the model work. Do not script the steps.
5. **Verification is the highest leverage thing you can add.** A short prompt with a real way to check its own work will outperform a long prompt without one.
6. **Aim above the ceiling.** Give the model a task slightly harder than you think it can do, and re-run old failures on every new release.
7. **Product overhang is where the opportunity is.** Models can already do valuable things that no product lets them express, and Claude Code itself came from spotting exactly that.
8. **Automate maintenance with one sentence routines.** Anthropic runs 20 to 30 daily routines that clean dead code, unify duplicated abstractions, and manage tests across all their apps.

---

*Converted from the uploaded PDF into Markdown.*
