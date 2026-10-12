# debug-walk

A [Claude Code](https://claude.com/claude-code) skill that walks one code behavior the way a debugger would. It shows real values at chosen breakpoints, in time order, and writes:

- a Markdown walk report with a sequence chart, a case table, and possible bugs;
- an interactive execution view in the style of an AWS Step Functions execution: a graph of states, Input / Output / State / Note for each state, and an event history.

It observes. To fix a bug it finds, hand it to a diagnosis workflow that builds a failing repro first.

## Install

Copy the `debug-walk` folder into your skills directory:

```sh
cp -R debug-walk ~/.claude/skills/
```

Then ask Claude Code to "walk" a behavior, or run `/debug-walk`.

## Files

| File | Job |
| --- | --- |
| `debug-walk/SKILL.md` | The steps: scope, locate, map, cases, run, write, execution view, clean up, check |
| `debug-walk/REPORT.md` | The report shape, value-block rules, and the JSON field map for the execution view |
| `debug-walk/EXECUTION.html` | The execution view template. The agent fills only the JSON in `#walk-data` |

## Notes

- The example in the templates is made up (a pricing-rules check). No code ran.
- The execution view is published with Claude's Artifact tool. Without it, open the HTML file in a browser.
- Step 9 refers to a `diagnosing-bugs` skill for suspected bugs. Swap in your own debugging workflow if you have none.

## License

MIT
