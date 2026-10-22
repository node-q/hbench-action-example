# hbench-action-example

An example repository for [Harness Benchmark](https://github.com/node-q/hbench-action)
(`node-q/hbench-action`): three small benchmark tasks, and the workflows that check them and run coding
agents on them with [hbench](https://github.com/node-q/hbench).

Fork it, add a key, and run an agent from the Actions tab.

- [What's here](#whats-here)
- [Try it](#try-it)
- [The workflows](#the-workflows)
- [Add your own task](#add-your-own-task)
- [License](#license)

## What's here

```text
tasks/
  hello-world/    write a greeting to a file
  fix-median/     fix a Python median() that is wrong for lists of even length
  count-words/    find the three most frequent words in a text file
registry.json     the three tasks as a dataset, example@1.0
.github/workflows/
  check-tasks.yml    on every push and pull request: the oracle passes every task, an idle agent none
  run-agent.yml      by hand: run an agent on the tasks
  run-dataset.yml    by hand: run an agent on example@1.0 or a public dataset
```

A task is a folder with an instruction, a Docker environment, tests that write a reward, and a reference
solution:

```text
fix-median/
  task.toml             name, timeouts, resources
  instruction.md        what the agent is asked to do
  environment/          the Dockerfile and the files the agent starts from
  tests/test.sh         writes 1 or 0 to /logs/verifier/reward.txt
  solution/solve.sh     the reference solution, which the oracle runs
```

## Try it

1. Fork this repository. Actions run in your fork on GitHub's runners.
2. Add a key for the agent you want as a repository secret (Settings, Secrets and variables, Actions):

   | Agent | Secret | Example model |
   |---|---|---|
   | `claude-code` | `ANTHROPIC_API_KEY` | `anthropic/claude-opus-5-5` |
   | `codex` | `OPENAI_API_KEY` | `openai/gpt-5` |
   | `gemini-cli` | `GEMINI_API_KEY` | `google/gemini-3-pro` |
   | `deepseek-harness` | `DEEPSEEK_API_KEY` | `deepseek/deepseek-v4-flash` |

3. In the Actions tab, open **Run an agent**, choose **Run workflow**, pick the agent and its model, and run it.

The run's summary shows each task's score and the mean reward, and the job's files (trajectories, logs,
results) are attached as an artifact. **Check tasks** needs no key: it runs on every push.

## The workflows

**Check tasks** runs every task twice. With the oracle, every task must score 1, or the step fails
(`min-mean-reward: 1`): a task whose own solution fails is broken. With `nop`, an agent that does nothing,
every task must score 0: a task that passes without the work tests nothing.

```yaml
- uses: node-q/hbench-action@v1
  with:
    path: tasks
    agent: oracle
    min-mean-reward: 1
```

**Run an agent** runs the agent you pick on the three tasks. The keys come from secrets in `env`; an agent
reads only its own.

```yaml
- uses: node-q/hbench-action@v1
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  with:
    path: tasks
    agent: claude-code
    model: anthropic/claude-opus-5-5
```

**Run a dataset** runs a dataset instead of a folder: `example@1.0`, listed in this repository's
`registry.json`, or one of the public datasets at `oci://ghcr.io/node-q/hbench-datasets`, such as
`humaneval-plus@1.0` or `terminal-bench@2.0.1`, at most `n-tasks` of its tasks.

```yaml
- uses: node-q/hbench-action@v1
  with:
    dataset: example@1.0
    registry: registry.json        # or oci://ghcr.io/node-q/hbench-datasets
    agent: oracle
```

Every input and output of the action is in its
[reference](https://github.com/node-q/hbench-action/blob/main/docs/REFERENCE.md).

## Add your own task

1. Copy a task folder, such as `tasks/fix-median`, and give it a new name, in the folder and in `task.toml`.
2. Write the instruction, put the starting files in `environment/`, and make `tests/test.sh` write 1 to
   `/logs/verifier/reward.txt` when the work is done and 0 otherwise. Keep the expected answer in `tests/`,
   which the agent can't see.
3. Write `solution/solve.sh` so the oracle can pass it.
4. Add it to `registry.json` if it belongs in the dataset, and push: **Check tasks** runs it with the oracle
   and with `nop`.

To run the same checks on your machine, install hbench and run `hbench run -p tasks -a oracle`.

## License

Apache-2.0. See [LICENSE](LICENSE).
