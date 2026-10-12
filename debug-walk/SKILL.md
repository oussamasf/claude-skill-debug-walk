---
name: debug-walk
description: Walks one code behavior as a debugger would — real values at chosen breakpoints, in time order — and writes a Markdown walk report with a sequence chart and possible bugs, plus a Step Functions–style execution view. Use when the user asks to run a debugger, step through a flow live, or see concrete values for how a behavior works. It observes; a known bug to fix goes to diagnosing-bugs.
---

# Debug walk

You are the debugger. You walk the arrow of time through one **behavior**, stop at each **breakpoint**, and show the real values there. The reader sees only what the debugger shows: module, function, values in, values out, state changed.

A behavior is a question, not a function: "how does the pricing check learn which discount rule fired?" It can cross many modules.

## Steps

1. **Scope.** Fill three slots from the request and the code:
   - the behavior to understand;
   - the entry and the exit of the walk (`module · function` at each end);
   - the value source: real data (DB, fixtures, a sample file, a failing input the user reports) or synthetic values.

   Ask with AskUserQuestion, in one call, only for the slots that are still open. For an open behavior slot, offer 2–3 readings of the request.

   Done when you can write the behavior as one sentence: "When `<input>` enters at `<entry>`, how does it become `<output>` at `<exit>`?"

2. **Locate.** Find the **workline**: the full chain that the walked segment is part of. The user may start the walk with no knowledge of this chain, so you discover it from the code, not from the user.
   - Upstream: follow the callers of the entry until you reach a **trigger** that a person can name (HTTP route, CLI command, cron job, queue consumer, UI action, test).
   - Downstream: follow the output of the exit until you reach an **effect** that a person can observe (DB row, response, file, message, screen).
   - At a queue, event bus, or scheduler the call stack ends: find the consumer and continue from its handler.
   - For each node, write `module · function` and one line on its job in the system.

   Keep 5–10 nodes in one straight line. When there are many triggers, choose the one that the cases use and name the others in one line.

   Done when the chain runs from a named trigger to an observable effect, and you can say in one sentence what the workline does for the product and what fails downstream when the walked segment is wrong.

3. **Map.** Read the code on the path. List every `module · function` the value passes through, in call order. Mark a step as a breakpoint only when one of these occurs:
   - the value changes shape (parse, map, build);
   - the code makes a decision, so the next step depends on the value;
   - state outside the call changes (a map, a cache, a file, a DB row).

   Skip pass-through calls. Keep 4–8 breakpoints per case.

4. **Cases.** Choose 2–4 cases the way you choose test cases: the common path, the miss, and one edge that the code handles differently. When the user reports a bad input, it is one of the cases. Each case must end at a different exit or leave a different state. Drop a case that repeats the path of another case.

5. **Run.** Get the values in this order of preference:
   1. run the real code with a temporary logger harness in the scratchpad (or a gitignored `dev/` folder);
   2. run an existing test with logs added;
   3. read the code and simulate, and mark each value `(simulated)`.

   Make the run **sandboxed** before it touches real code paths: use a read-only connection or local data, and stub each effect from step 2 that writes (DB writes, queue publishes, mail, external calls). Ask the user before a run that touches shared state.

   Tag every harness log with one prefix, for example `[WALK-a4f2]`. Pin the clock, the random seed, and the flags that change the path, and record them.

   Copy each value from the run output. A value the code computes comes from the code, never from a guess. **Redact** before a value enters the report: write `<REDACTED>` for tokens, keys, passwords, and personal data. Keep IDs and shapes; they carry the signal.

6. **Write** the report in the shape that [REPORT.md](REPORT.md) gives. Save it to `dev/<topic>/<behavior>-walk.md` when the repo has a gitignored `dev/`. If not, save it to the scratchpad.

7. **Execution view.** Copy [EXECUTION.html](EXECUTION.html) next to the Markdown report as `<behavior>-walk.html`. Replace only the JSON in `#walk-data` and the `<title>`, with the same cases, steps, and values as the Markdown (see "Execution view" in [REPORT.md](REPORT.md)). Publish it with the Artifact tool.

8. **Clean up.** Delete the harness, and remove every log line with the prefix (`grep` for it).

9. **Check** the report against the list below, and correct each miss. Then give the user the Markdown path, the artifact URL, a 3-line summary, and each `suspected` bug as a hand-off to the `diagnosing-bugs` skill, which builds a red loop for it.

## The linear rule

Each case is one straight line of numbered steps in time order.

- A decision shows only the arm that this case took, and the value that chose it. Another case walks the other arm.
- A loop is a step that goes back: `② → ③ → back to ② (i = 1) → ③ → ④`. Show the first pass and the last pass. Write the passes between them as one line: `… 473 passes, all false …`.
- Concurrent work (`Promise.all`, threads, workers) shows in the order the run logged it. Write "observed order" on it: a second run can differ.
- The sequence chart in the summary is one straight line of calls, with `loop` as its only block.

## Check

- [ ] The header gives the commit, the value source, and the pinned values.
- [ ] "Where you are" runs from a trigger to an effect, and follows each async hop to its consumer.
- [ ] Every breakpoint names `path/file · function` and shows concrete values. Each simulated value has the `(simulated)` mark.
- [ ] Each case reads top to bottom with no branch, and ends at a different exit or leaves a different state.
- [ ] The report obeys every rule in [REPORT.md](REPORT.md): map, values block, ASD-STE100 notes.
- [ ] No secret or personal data is in the report.
- [ ] Each possible bug names a trigger value, the step where it shows, and a confidence.
- [ ] The report is at most about 150 lines.
- [ ] The execution view has the same cases, steps, values, and bugs as the Markdown, and its URL is in the final message.
- [ ] The harness is deleted, and the log prefix is not in the tracked tree.
