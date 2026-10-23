# hbench-action-example

An example repository for [hbench-action](https://github.com/node-q/hbench-action), the GitHub Action for
[hbench](https://github.com/node-q/hbench): three small benchmark tasks, and the workflows that check them and
run coding agents on them.

Fork it, add a key, and run an agent from the Actions tab.

- [What's here](#whats-here)
- [Try it](#try-it)
- [The workflows](#the-workflows)
- [Run it on your machine](#run-it-on-your-machine)
- [Add your own task](#add-your-own-task)
- [License](#license)
- [Contributors and contact](#contributors-and-contact)

## What's here

```text
tasks/
  hello-world/    write a greeting to a file
  fix-median/     fix a Python median() that is wrong for lists of even length
  count-words/    find the three most frequent words in a text file
registry.json     the three tasks as a dataset, example@1.0
.github/workflows/
  check-tasks.yml    on pushes to main and pull requests: the oracle passes every task, an idle agent none
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
results) are attached as an artifact. **Check tasks** needs no key: it runs on every push to main and every
pull request.

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
`humaneval-plus@1.0` or `terminal-bench@2.0.1`. It runs at most `n-tasks` of the dataset's tasks, 5 unless you
change it, so the score covers those tasks only. A score compares only with runs of the same tasks: the same
dataset and version from the same registry, with the same `n-tasks`.

```yaml
- uses: node-q/hbench-action@v1
  with:
    dataset: example@1.0
    registry: registry.json        # or oci://ghcr.io/node-q/hbench-datasets
    agent: oracle
```

Every input and output of the action is in its
[reference](https://github.com/node-q/hbench-action/blob/main/docs/REFERENCE.md).

## Run it on your machine

You need Docker, on Linux or macOS. From a clone of this repository:

```bash
curl -fsSL https://raw.githubusercontent.com/node-q/hbench/master/scripts/install.sh | sh

hbench tasks check -p tasks                                         # validate the tasks' files, no key
hbench run -p tasks -a oracle                                       # every task, with the oracle
hbench datasets list --registry-path registry.json                  # the datasets in registry.json
hbench run -d example@1.0 --registry-path registry.json -a oracle   # example@1.0, from registry.json
hbench run -d example@1.0 -a oracle \
  --registry-url https://raw.githubusercontent.com/node-q/hbench-action-example/main/registry.json
ANTHROPIC_API_KEY=sk-ant-... hbench run -d example@1.0 --registry-path registry.json \
  -a claude-code -m anthropic/claude-opus-5-5                       # an agent
hbench view jobs                                                    # the results in your browser
```

The tasks in `registry.json` name folders of this repository, so run it from the clone's root, by path or by
URL.

## Add your own task

1. Copy a task folder, such as `tasks/fix-median`, and give it a new name, in the folder and in `task.toml`.
2. Write the instruction, put the starting files in `environment/`, and make `tests/test.sh` write 1 to
   `/logs/verifier/reward.txt` when the work is done and 0 otherwise. Keep the expected answer in `tests/`,
   which the agent can't see.
3. Write `solution/solve.sh` so the oracle can pass it.
4. Add it to `registry.json` if it belongs in the dataset, and open a pull request: **Check tasks** runs it
   with the oracle and with `nop`.

To run the same checks on your machine, see [Run it on your machine](#run-it-on-your-machine).

## License

Apache-2.0. See [LICENSE](LICENSE).

## Contributors and contact

hbench-action-example is maintained by [node-q](https://nodeq.tech). See the
[contributors graph](https://github.com/node-q/hbench-action-example/graphs/contributors) for everyone who has
contributed.

- Questions and help: [#support on Discord](https://discord.gg/5b6wCn5sEK)
- New releases: [#announcements on Discord](https://discord.gg/dEvy6DCMTn)
- Bugs in this example: [GitHub issues](https://github.com/node-q/hbench-action-example/issues); for the
  action, [node-q/hbench-action's issues](https://github.com/node-q/hbench-action/issues); for hbench itself,
  [node-q/hbench's issues](https://github.com/node-q/hbench/issues)
- Every input and output of the action: its
  [reference](https://github.com/node-q/hbench-action/blob/main/docs/REFERENCE.md)
