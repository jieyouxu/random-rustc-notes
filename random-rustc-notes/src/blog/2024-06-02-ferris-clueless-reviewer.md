# A ferrisClueless reviewer

| Metadata       |            |
|----------------|------------|
| Date published | 2024-06-02 |

In this post, I describe:

- My experiences being on the Rust compiler [`rustc`]'s review rotation for the first time.
- What goes into a compiler code review from my perspective.
- How I (try to) do code reviews.
- What I think PR authors can do to help make the review easier.

I hope this post can share some insights into how compiler PRs are reviewed :3

## Background

### Pull requests to [`rustc`] and code reviews

[`rustc`] is maintained by the [compiler team] ("T-compiler"). Pull requests (PRs) to the compiler
are expected to go through code reviews and requires approval (`r+`) from an authorized reviewer
before the changes can be merged to `master`. Specifically, T-compiler reviewers.

### What's the T-compiler review rotation?

The specifics are explained in more details in the [re-organize the compiler
team][compiler-reorganize] RFC, but the gist is that T-compiler members who volunteered to be on the
review rotation will be randomly assigned PRs (or explicitly requested by PR authors or others) to
review PRs primarily affecting the compiler.

## Who dis?

I am a *very* new T-compiler ~~toaster~~ contributor. I was invited to join T-compiler contributors
approximately [two months ago][compiler-invite] (at the time this blog post was written). I am new
to the compiler in general: I've only made changes to the compiler in [random PRs here and
there][my-prs]. Amusingly, looking through the PR history, many of my contributions to
[rust-lang/rust][rust] are not to the compiler itself (code in `compiler/`), but to the test
infrastructure: to the test harness `compiletest` and to the `run-make` test suite.

## Joining the compiler review rotation

As you can imagine, [`rustc`] is a big codebase, and I am not familiar with many parts of the
compiler. But that's why we have a compiler *team*: different members on the team are familiar with
different parts of the compiler.

<center>
    <figure>
        <img src="./2024-06-02-ferris-clueless-reviewer/ferris_clueless.png" width=100px>
        <figcaption>The `:ferrisClueless:` emote. Taken from the Rust community discord, made by @.foobles.</figcaption>
    </figure>
</center>

I [signed up for the compiler review rotation for the first time][rotation-signup][^review-pool]
*anyway* on 2024-04-05 and have since been [assigned PRs to review][assigned-prs] (this list is not
exhaustive because I have reviewed and/or approved PRs that are not assigned to me).

Having more compiler reviewers on rotation always help to relieve some of the [review capacity
pressure][review-capacity] that the compiler team was having causing long delays in review times for
compiler PRs (we went from having ~**8** compiler reviewers on rotation to now ~**17**!).

It's quite overwhelming during the first few assigned PRs, especially because I'm also new to the
compiler team. Actually, I'm just generally new to reviewing other people's code. We have some
docs/guidance for review policies and the review procedure, but it's kinda scattered around in a
couple of places: [Rust Forge][forge-review], [rustc-dev-guide][dev-guide-review], in T-compiler
folklore and presumably in some random HackMD documents somewhere too. It really is a
learn-as-you-go experience. I try to learn from what other reviewers are doing, such as the
reviewers assigned to my own PRs.

*Fake it till you make it*, right? Truly a `:ferrisClueless:` moment.

## What goes into a compiler code review?

As a T-compiler reviewer, to successfully (for some metric of "successfully") review a randomly
assigned PR, I need to:

- Have some familarity with the infrastructure:
    - Being familiar with our two bots, `rustbot` and `bors` and being familiar with our CI: try
      jobs `@bors try`, approvals `@bors r+`, marking as rollups (or as iffy/never rollup).
    - Help the PR authors run try jobs or perf runs if they don't have `try` or perf permissions; I
      need to make sure that the code that is submitted to try jobs or perf runs do not contain any
      malicious code, because the build artifacts are publicly available.
- Determine that I have "**jurisdiction**": I am only a T-compiler contributor, so I can only
  approve PRs that primarily affect code maintained by T-compiler.
    - If the PR is primarily affecting code or other artifacts maintained by another Rust team (you
      can visit <https://www.rust-lang.org/governance> to see the various Rust teams), then I'll
      need to pass the hot potato (I like to call reassigning the reviewer `r? another_team` as
      passing the hot potato) to a more suitable reviewer of the relevant team.
    - If the PR affects multiple teams, then I will need to focus on the part of the PR affecting
      the compiler, and determine if I need to also request reviewers from the other teams to review
      parts of the PR affecting the other teams.[^inter-team-review]
- **Suitability triage**: am I a suitable reviewer for the changes?
    - If the PR modifies parts of the compiler I am not familiar with or is very tricky, do I have
      enough confidence in determining the correctness of the changes / am I comfortable enough to
      eventually give an `r+`?
        - If I am not confident initially, am I likely to eventually become sufficiently confident
          as I dig into the relevant parts of the compiler more / consult with other compiler
          contributors?
        - Do I have enough time to review the changes? Am I likely to be able to review the PR
          in a reasonable timeframe?
- Determine if the changes require any Procedural Things™:
    - Does this PR need a ACP/RFC/MCP? If so, from which teams?
    - Does the PR need feature gating? Does the PR need (const-) stability attributes?
- Determine if the PR can cause any breaking changes or regressions?
    - Behavioral changes?
    - Does the PR cause existing code to start/stop compiling?
    - Does the PR affect any kind of stability?
    - Will the lint start linting on new code? Is the lint level suitable? Does the lint changes
      need T-lang approval?
- Determine if it's possible that the PR will have (widespread) effects if merged. Does the changes
  need to go through [crater]?
- Determine if the PR has sufficient testing.
- Determine if the PR introduce any new dependencies that are not approved.
- File relevant issues or track them elsewhere for problems/suggestions discovered/realized during
  code review, that is suitable to be addressed in follow-up PRs.
- Check if the code is generated by AI and contains nonsense. Yes, we have spam PRs generated by
  various AIs that make no sense when examined.

A lot of consideration can go into a simple

```bash
@bors r+ rollup
```

I feel like it's hard to come up with useful generically applicable PR review workflows, because
Every PR is Special™. Anyways, it does get easier as one gains more experience in the review
workflow and more familiarity with more parts of the compiler. You develop a better "feel" for if
you can review a given PR as you review more.

## How I (try to) do code reviews

By YOLOing it.

Okay, maybe not quite that.

I try to always keep in mind that code reviews are a "we" thing: the PR author and the reviewer works
**together** to improve the compiler. It's not a "versus" thing.

I try to:

- Gather and understand the context for why the changes are proposed: relevant issues, discussions,
  PRs, RFC/MCP/ACPs, etc.
- Review changes in a "sandwich" fashion:
    1. An overall pass to understand at a high level what the general changes are and how they try
       to accomplish the goal of the PR.
    2. Individual passes for individual commits: how does each commit try to further advance the
       objective?
    3. Another overall pass to consider if the change details make sense to serve the overall
       purpose.
- Ask lots of questions:
    - The PR author likely knows more about the specific changes than you do and have more context.
    - Some of the answers can become comments in code, in commit messages, which are very useful as
      documentation and not get lost in the PR discussion.
    - This helps to avoid shutting down discussion: I don't want to sound assertive when it doesn't
      need to. That discourages PR authors from contributing their own thoughts/suggestions
      (especially new contributors).
    - A simple "what do you think?" can be very powerful! It can make the PR author feel that their
      opinion is valued and their input is desired. They of course are, but an explicit question
      like this can make the appreciation itself explicit!
- Use [review comment prefixes][rev-prefixes] to clearly indicate which review comments are blocking
  vs non-blocking: I don't want to block the PR on nits if the PR author has different preferences.
- If there are some changes that makes sense to be addressed in follow-up PRs, then don't block this
  PR.
- Avoid getting bogged down in nits.
- If I think something is wrong, explain *why* I think it is wrong: it's quite possible that my
  analysis is flawed and it's actually correct!
- If I think there's better ways to do things, try to give some illustrative examples if possible.
- Praise what I like about the PR: appraisal establishes positive feedback loops: it reinforces good
  practices (subject to the reviewer's view on what constitutes good practices) and encourages PR
  authors to come back in the future for more contributions.
    - I don't do this often enough and am trying to get better at it. When reviewing, it's easy to
      get tunnel visioned on the potential improvements and problems and neglect what *is* nice
      about the changes.
- If we run into weird issues, or if the PR author has questions that I don't have the answers to, I
  try to help them find an answer: this can involve me acting as a liason and asking other
  contributors who *are* knowledgable for what the problem might be and if there are suitable
  solutions.

## What can make a PR easier to review?

With my reviewer hat on, these things can contribute to making a PR easier for me to review:

- [Atomic commits][atomic-commits]: try to make each commit focused on one aspect. For example,
  having one commit making a compiler change, and a follow-up commit that updates/blesses the tests
  can be helpful.
    - `@nnethercote`'s PRs are absolutely stellar to learn from: good PR descriptions, commit
      messages and atomic commits, makes it very plesant to review. For example,
      <https://github.com/rust-lang/rust/pull/125815>.
- It can make sense to have a series of PRs for bigger changes.
- Avoid unrelated changes: it can be tempting to do drive-by cleanups or changes
- After each round of reviews, posting a high-level summary of what changes are made can be very
  helpful. Making review comments as resolved or post a quick reply can be useful to indicate that
  the feedback has been addressed as well.
- Liberal use of `@rustbot ready`/`@rustbot author` to indicate to the reviewer that the author
  either intends to make more changes or that the PR is in a state that's ready for another round of
  reviews.

## Aside: the Hot Potato game

Sometimes a PR make changes to more "exotic" parts of the compiler. Like ABI or layout. Or codegen.
Then, what typically happens is that the first randomly assigned compiler reviewer is not
comfortable with reviewing the changes, so they reroll with `r?`.

Then the second reviewer is also not comfortable with reviewing.

Then the third.

And fourth.

Until eventually we realize the dice has been rolled too many times and ask around on Zulip who's
willing to take a look.

The Hot Potato game typically looks like this:

```
@author: *makes some ABI/layout/codegen changes*
@rustbot: *randomly assigns `@reviewer_2`*
@reviewer_1: r? compiler
@rustbot: *assigns `@reviewer_2`*
// 1 day later
@reviewer_2: I'm not comfortable reviewing this. r? compiler
@rustbot: *assigns `@reviewer_3`*
// 4 days later
@reviewer_3: Me neither. r? compiler
// 3 days later
@rustbot: *assigns `@reviewer_4`*
@reviewer_4: *opens zulip thread asking who can review this PR*
...
@rustbot: *assigns `@reviewer_n` who can finally review this PR*
```

There's just some extra delays if the hot potato keeps getting passed around, so we should really:

1. Identify reviewers who are comfortable with e.g. ABI/layout/codegen, and
2. Setup a ping group for experts who reviewers can consult with if they want extra pairs of eyes on
   delicate changes.

---

I'm sure I missed plenty of things I could've talked about, but that's what I was able to cook up
(toast up?) when I was writing this blog post. Thanks for reading if you got this far! I look
forward to more people (and toasters) joining the compiler review rotation in the future :3

---

[^review-pool]: Actually, not just for `compiler/` changes; I also am in the reviewer pool for PRs
affecting `src/tools/run-make-support`, `tests/run-make` and `src/tools/compiletest`.
[^inter-team-review]: I would love if `rustbot` has first-class support for supporting a
multiple-reviewers model, where it is possible to request multiple primary reviewers and require
joint approval.

[`rustc`]: https://github.com/rust-lang/rust
[rust]: https://github.com/rust-lang/rust
[compiler team]: https://www.rust-lang.org/governance/teams/compiler
[compiler-reviewers]: https://github.com/rust-lang/rust/blob/a83cf567b5949691de67f06895d9fe0404c40d27/triagebot.toml#L898-L919
[compiler-reorganize]: https://github.com/rust-lang/rfcs/pull/3599
[rotation-signup]: https://github.com/rust-lang/rust/pull/123509
[assigned-prs]: https://github.com/rust-lang/rust/pulls?q=is%3Apr+assignee%3Ajieyouxu+
[compiler-invite]: https://github.com/rust-lang/team/pull/1421
[my-prs]: https://github.com/rust-lang/rust/pulls?q=author%3Ajieyouxu+-label%3Arollup+
[review-capacity]: https://rust-lang.zulipchat.com/#narrow/stream/131828-t-compiler/topic/review.20queue.20.26.20capacity
[forge-review]: https://forge.rust-lang.org/compiler/reviews.html?highlight=review#review-policies
[dev-guide-review]: https://rustc-dev-guide.rust-lang.org/contributing.html#pull-requests
[crater]: https://github.com/rust-lang/crater
[rev-prefixes]: https://conventionalcomments.org/
[atomic-commits]: https://www.aleksandrhovhannisyan.com/blog/atomic-git-commits/
