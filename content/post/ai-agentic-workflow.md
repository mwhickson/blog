+++
date = '2026-09-27T18:13:34-04:00'
draft = false
title = 'AI Agentic Workflow Experiment'
+++

Hello again!

I have returned to detail my experiments into AI agentic workflow for software development.

SPOILER: Once again, there is much promise, but ultimately things don't seem to be there just yet.

<!--more-->

More and more recently I read about software developers, programmers and dabblers that claim they "don't write code anymore!" or that they use some magnificent AI-based workflow for vibe-coded or other AI-assisted projects.

Well, I have to say that I was intrigued!

What would that even look like?

How far could it go?

Was there any truth to any of it?

To find out, I sat down with my vibe-coding buddy of choice, Gemini, and walked through the process.

I had specific limitations I wanted to include in this endeavor. These are a result of "testing the waters" and "keeping it safe" in light of sensitive information leakage and AI breakout concerns.

A quick overview of the specification was:

- all AI agents should confirm actions at pre-set intervals with a human orchestrator
- all activity should be local only (e.g. LLM environment, output, etc.)
- roles should be used to focus agentic effort (I ended up with Planner, Builder, Tester and Reviewer)
- all activity should be comprehensible and reviewable by an appropriately skilled human
- quality and reproducibility for the purposes of reliability, review and ongoing maintenance were key guiding factors

The goal I had was to utilize lower-end hardware (a no CUDA GPU, CPU-bound desktop) to produce working software from a reasonable specification, with minimal human involvement to correct issues that wouldn't be issues if the task(s) were being performed by a human.

As I have already spoiled the outcome, the results were promising, but not overwhelmingly impressive for larger scale use.

Before I talk about the final results, let's talk a bit about the process to turn my first grand specification into [Agent-Ranch-0](https://github.com/mwhickson/agent-ranch-0) (AR0).

>(`Agent-Ranch-0` was originally `Agent-0`, but I wanted to capture the visual of open plains as far as the eye could see with Agents roaming under the watchful eye of a lonesome cowpoke... So, I stuffed `Ranch` into the name and called it done. `0` was because this was the prototype version. Clever? Not particularly. Still, better than some of the random project names that were so en vogue a while ago... `strawberry-wombat-eleven` anyone?)

Anyway, the process I followed was as follows:

- know what I wanted to achieve, and generally how I wanted to achieve it
- accept that vibe-coding with Gemini was going to be a large part of this because:
  - I didn't really know what this should look like
  - I figured who understands AI agents better than an AI (especially since I was using Gemma, a model I figured Gemini should have a feel for...)
- break the desired outcome into steps, including strong statements about priorities like local-only, human-guided, human-comprehensible
- rein in the eagerness AI (even Gemini) has to want to one-shot something that it thinks I wanted, rather than specifically what I requested
- iterate each step with Gemini, providing ongoing feedback and tailoring output to get what I wanted
- test each iteration with my local LLM (Koboldcpp + Gemma) and identify actionable steps to move the project forward
- lather, rinse, repeat

It didn't take long to get something up and running, but to reach a stage of "Hey! I got a working program out of AR0!" took a few days.

At its core, AR0 is a serialized state machine passing instructions off to the LLM with agentic role prompts and keeping track of progress in a Sqlite database.

I was somewhat surprised at the pivotal role the LLM prompts played, though in retrospect, I really shouldn't have been.

The housing handled formal hand-offs between agentic roles, but the individual role LLM sessions where were the work happened.

The workflow evolved to:

- Planner reads the specification and provides a set of non-overlapping, non-ambiguous tasks (stored in Sqlite) for Builder to create from
- Builder reads tasks (from Sqlite) and builds code (stored on disk via `cat`) to fulfill the goals of the current task
- Reviewer examines the code (from disk) to make sure that Builder didn't miss the point of the task, and that the code generated is testable
- Tester examines the code and produces automated tests to help with quality control, and to avoid back-sliding if code is regenerated
- If Tester's tests pass, control returns to Builder to complete any remaining tasks; otherwise, the current task is returned to Builder to try again

Human sign-off occurs after the plan is produced, but before Builder starts writing code. Sign-off occurs again when the code has been generated and reviewed, but before it has been tested.

It became clear during the development of AR0 that the `Action Schema` (the JSON schema used to transmit actions/statuses between roles) was crucial -- and subject to a variety of hiccups. Using some strong JSON schema adherence (part of Koboldcpp) might have helped, but I wanted to avoid restricting the LLM runner to a specific application. Clear JSON examples, and some code in the execution environment to correct any deficiencies seems to have worked well enough. Gemini was superb at identifying when and where Gemma (4B, 2B and eventually 26B) was having difficulties and proposed prompt solutions that resolved issues almost always on the first attempt.

Other issues corrected/refined during development included:

- Agents make typos; rather than passing on broken code -- a Python compilation phase was used to push broken code back to Builder immediately so that it wouldn't be tested before it compiled
- Agents like to do as much work as possible in as little time as possible, sometimes involving shortcuts and often ignoring instructions to produce testable code -- Reviewer was added to stop this from happening, and it was a wonderful improvement
- Agents don't like redoing things -- Reviewer helped here too, ensuring that Builder took feedback from Tester into account and didn't just resubmit things unchanged (and broken) or make trivial changes to fulfill the obligation of the return, rather than addressing the issue resulting in the return
- Tester could make typos too! -- the Python compilation phase was used to avoid Tester returning valid code because the tests were broken
- Retries were part of the workflow from the start, to prevent Agents from entering into wheel spinning behaviour due to being unable to complete a request due to errors or token exhaustion -- human intervention was triggered at points where retries reached their limit to address concerns about the workflow itself, or costs associated with repeating sequences producing no actionable output
- Resuming work (after a retry limit was reached, or a crash required a workflow restart) meant that the work for a specific workflow stage or task could already be complete -- Planner, Builder and Tester were given instructions to issue a `NO_OP` with the reason that the work was already complete to shorten the cycle to get to new work (task state in the Sqlite database also helped Agents resume activity at a reasonable point)

When everything was all said and done, AR0 was capable of successfully tackling the following projects:

- a number guessing game in Python
- the same number guessing game in Go (required changes to the workflow harness, and also saw the Gemma 26B model come into use)
- a library (in Python) to support some common conversions for measurements
- a Tcl/Tk GUI wrapper application for the conversion library

Overall, taking a written specification in Markdown format and producing working code with tests, even for programs this small, impressed me.

With limited hardware and local resources only, it turns out it was entirely possible to build and execute an LLM-based software development pipeline that produced viable end results!

It wasn't until I tried to produce a more complex game in Python that limitations became apparent.

In short, a sizeable specification consumes a large percentage of the context the LLM agents use to complete their tasks, and token starvation results in a frustrating "will it even complete?" game of wait and see.

The "wait and see" became the point at which progress stalled and the project lost its lustre.

I know that I could push the project further by:

- extending the token budget (I tried this, and met with the same results. I should revisit this.)
- improving the quality of hardware used for the experiment (not in the budget at this time)
- pushing the experiment out to use an external model (Gemini, Claude, etc.) -- this runs counter to the fundamental goals of the project

At any rate, this was essentially the journey I took.

It helped solidify my understanding of what an Agentic software development workflow might look like, and helped me explore the issues and limitations that go along with a workflow of that nature.

I think it was a good and worthwhile experiment -- even if I ended up in the same spot at the conclusion of the experiment (AI is impressive, but still has sizeable gaps in what it can do and how it can do it).

Lastly, I'm not really sure the experiment is over...

I know I'm taking a breather from things at the moment; however, I do still want to revisit this workflow or something similar.

Watch [this space](https://github.com/mwhickson) for details!

---

## Bonus Content

These are a few tidbits I wanted to pass along as well, but couldn't find a way to work them into the post above in a timely and seamless manner.

- `cat <<<EOL` is a common technique to keep AI code production on the rails (Agents may try to get fancy and use things like `sed`... this rarely goes well and is best avoided through explicit prompt instructions forbidding it)
- some languages are better suited to AI code production, evaluation, etc.
  - Python, Go, Rust are all fairly terse, fairly well-known (with examples driving quality generation) and with good feedback when failure occurs
  - Java suffers from stack trace context bloat -- not recommended
  - C++ suffers from "kitchen sink" multi-paradigm nature of what it supports (Gemini cites template errors as a very strong source of "things that will confuse an LLM")
  - C is terse, but segfault parsing when the worst of the worst occurs, alongside the strong "need" for tools like `valgrind`, `gdb`, etc. for testing purposes are beyond what a local setup is likely to provide (without careful curation)
- often prompts will accumulate edge-cases, duplication, and other verbosity that eats up context space and reduces overall clarity for Agents -- review and clean up prompts when this happens
- a low `temperature` when submitting a prompt reduces hallucinations (and other creative solutions), but having more than one agent in a given role (where agents have differing temperatures) can produce higher quality results and avoid an individual Agent from getting "stuck in a rut"
