---
name: fixing-flaky-tests
description: The quality bar for repairing a flaky test — replace the assumption that races with a guarantee, in the smallest diff that holds. Use when fixing an intermittently failing test, or when tempted by retries, sleeps, soft assertions, conditional flow, or force clicks.
---

# Fixing Flaky Tests

Diagnosis ends with a named race. This skill is about the step after — and it is where good work usually goes wrong, because the pressure is to make CI green and there are a dozen ways to do that without touching the bug.

The line that decides every choice below: **a fix changes the logic, not the odds.** Widening a timing window, retrying an action, or softening an assertion all raise the pass rate while the race stays exactly where it was. The test now fails less often — which is strictly worse than failing, because nobody will look again.

## Start from a named cause

Do not open the file until you can state the race as a falsifiable sentence: *"the test reads the total before the cart PATCH resolves."* If you can't, you are about to guess, and a guessed fix is indistinguishable from a hack even when it happens to work. Go run the `diagnosing-flaky-tests` loop first — reproduce at a stated rate, then come back.

Corollary: the reproduction command from diagnosis is the acceptance criterion for the fix. Without it you cannot tell repair from luck.

## A fix replaces an assumption with a guarantee

Every flaky test asserts something it assumed had already happened. There are only three honest repairs, and all three are about the assumption itself:

1. **Wait on the signal that actually guarantees it.** Not a longer wait — the *right* one: the response, the settled state, the URL, the enabled button. The auto-retrying assertion your framework already ships is the tool; a custom polling helper is not.
2. **Create the guarantee.** The state was never promised, so promise it: own the data instead of sharing it, seed the precondition through the API, pin the clock, scope to a fresh context.
3. **Remove the assumption.** The test depended on something it never needed — another test's leftovers, a background job, an ordering. Drop the dependency and the race has nothing left to lose.

Check the candidate against the stress question: **would this still hold on a machine 10× slower, with 50 parallel workers, at 23:59:59 on December 31st?** A guarantee shrugs. A tuned window says "probably". "Probably" is the thing you were sent here to remove.

## Counterfeit fixes

Each of these turns a red signal green without closing the race. Recognize them by what they touch: the test's tolerance, never its assumption.

- **Retries** (`retries: 2`, `test.retry()`, rerunning the job) — the flake still happens, it just stops being reported. This is the one that compounds: it hides the next ten flakes too.
- **Longer timeouts** — widens the window the race runs in. The race still wins on a bad day, and now every genuine failure costs 30 seconds.
- **Sleeps** (`waitForTimeout`, `Thread.sleep`) — a guess about the app's speed, wrong in both directions at once.
- **Retry loops and custom poll helpers** (`retryClick`, `waitUntil`, `attempts: 5`) — a sleep with better optics. Same absent guarantee, plus new code you now own, plus the real failure buried under four silent retries.
- **Soft assertions** (`expect.soft`, `try/catch` around a check, `.catch(() => {})`) — the assertion still fails; the test just stops caring. Soft assertions are a legitimate tool for collecting several independent checks in one run. As a flake remedy they are the purest form of the lie: the product is broken, the pipe is green.
- **Conditional flow** (`if (await banner.isVisible()) …`) — the test now passes under two different app behaviors, so it asserts neither. If the state genuinely varies, that nondeterminism is the bug; pin it, don't branch around it.
- **Bypasses** (`force: true`, `dispatchEvent`, clicking via `page.evaluate`) — you clicked through a state a user could not have clicked through. Whatever blocked the click was the finding; that's now suppressed, and the test no longer tests the product.
- **Serializing** (`describe.serial`, `--workers=1`, reordering to hide pollution) — trades a fixable isolation bug for a permanent constraint and a slower suite, on every run, forever.
- **Weakening the oracle** (exact → `toContain`, asserting visibility instead of value, deleting the checkpoint that flaked) — the test survives and stops proving the thing it existed for. This is a coverage cut wearing a stability costume.

```typescript
// COUNTERFEIT: the click happens later, still at a guessed moment
await page.waitForTimeout(2000);
await saveButton.click();

// COUNTERFEIT: a loop is a sleep with better optics — no guarantee, and now it's code you own
await retry(() => saveButton.click(), { attempts: 5 });

// REAL: the state the click requires, waited on directly
await expect(saveButton).toBeEnabled();
await saveButton.click();
```

If a counterfeit is genuinely the only thing available today, it is a **quarantine**, not a fix: annotate it, link a ticket, name an owner. Visible debt beats a green lie.

## Keep the diff small

A real flaky fix is usually **smaller than the test it repairs** — a sleep becomes an assertion, a shared account becomes a per-test one, a hand-rolled poll loop deletes outright. If your diff is growing helpers, wrappers, and try/catch, you are building scaffolding around the race instead of removing it, and every line of it is new surface to maintain and new places to hide the next flake.

**One cause, one change.** Change five things at once and go green, and you know nothing: not which one mattered, not what the other four cost you. If diagnosis found several causes, fix them as separate changes, each proven separately.

The fix should also read as intent. `await expect(row).toHaveText("Refunded")` says *wait for the settled state*; `await sleep(2000)` says nothing to the next person, who will only be able to make it bigger.

## Prove it twice

A fix you haven't looped is a guess — and a fix you've only looped might have bought stability with signal. Both checks, every time:

1. **Stability** — the reproduction command from diagnosis, same stress, at least as many runs as previously reproduced it, zero failures. State the numbers: *"was 12/50, now 0/200 with `--repeat-each=200 --workers=8`."*
2. **Signal** — the test must still be able to fail. Break the behavior it guards on purpose (revert the product fix, force the endpoint to 500, corrupt the expected value) and confirm it goes red. A test that passes against a broken app is not a fixed test; it's a deleted one that still costs CI minutes.

Then diff the assertions. If any got weaker or disappeared while you were "stabilizing", that part isn't a fix — put it back and repair it properly.

## Fix the class, not the instance

When a cause is systemic — the same shared account, the same missing signal, the same sleep pattern across nine siblings — repairing one test is triage. The durable fix lives at the source: a fixture, a factory, one waiting helper used everywhere, a lint rule that bans the sleep. Roll it out mechanically as its own change, after the diagnosis is proven on the first test.

Then close the loop where conventions live: if `docs/agents/testing.md` doesn't yet say what the project does about this class, add it, so the next test doesn't reintroduce it.

## When the honest fix is in the app

Sometimes there is no test-side guarantee to wait on, because the product never exposes one — nothing distinguishes "loaded" from "still empty", or the app really does race with itself. Adding a `data-state="ready"` attribute, fixing the debounce, or making the mutation await its own refetch is a *smaller and stronger* fix than anything the test could do about it, and it fixes it for every future test too.

Don't treat "I can't touch app code" as a fact of nature — raise it as the finding it is. And if the race turns out to be in the product, the flaky test was working: file the bug, keep the test capable of going red, and fix the app.
