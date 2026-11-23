# jj-rex

`jj-rex` is a `zsh` script that manages `jj` workspaces and `rex` sessions. A workspace is a separate working directory
for the same repository. A session is a terminal group that `rex` manages. The script stores each workspace outside the
root repository in a sibling `workspaces` directory.

## Setup

You must install `zsh`, `jj`, `rex`, `jq`, and `fzf` before you run `jj-rex`. You must run `jj-rex` from a `jj`
repository outside the `workspaces` directory. If you run `jj-rex` from inside a path that contains `/workspaces/`, the
script stops with an error.

Run this command to download `jj-rex` into `~/.local/bin` and make it executable:

```sh
mkdir -p ~/.local/bin && curl -fsSL https://raw.githubusercontent.com/KingMichaelPark/jj-rex/main/jj-rex -o ~/.local/bin/jj-rex && chmod +x ~/.local/bin/jj-rex
```

Make sure that `~/.local/bin` is on your `PATH`. `PATH` is the list of directories that your shell searches for
commands. If you cloned the repository locally, you can link the script into `~/.local/bin` instead:

```sh
chmod +x /path/to/jj-rex
ln -s /path/to/jj-rex ~/.local/bin/jj-rex
```

## Directory layout

`jj-rex` reads the root directory of your current `jj` repository. If a `workspaces` directory does not exist next to
that repository, `jj-rex` creates one. Each workspace lives at `../workspaces/<repo>/<bookmark>`. A bookmark is a named
pointer to a commit in `jj`. For example, if your repository is `/Users/you/code/app` and the bookmark is `feat/login`,
`jj-rex` puts the workspace at `/Users/you/code/workspaces/app/feat/login`.

## Usage

Run `jj-rex` from your root `jj` repository. Run `jj-rex -h` or `jj-rex --help` to print the command reference. Each
command below operates on workspaces in `../workspaces/<repo>/`.

### Create or attach a workspace

Run `jj-rex -a <BOOKMARK>` or `jj-rex --attach <BOOKMARK>` from the root repository. If the workspace directory does not
exist, `jj-rex` creates it with `jj workspace add`. If the bookmark exists in `jj`, `jj-rex` sets that bookmark as the
parent revision.

`jj-rex` creates a `rex` session with the bookmark name in that workspace directory. If the session exists, `jj-rex`
reuses it. By default, `jj-rex` stays in your current `rex` session. To switch to the session after creation, pass the
`-s` or `--switch` flag.

```sh
jj-rex -a feat-auth
jj-rex -a feat-auth -s
jj-rex -as feat/login
```

### Switch to a workspace

Run `jj-rex` with no flags from the root repository. `jj-rex` lists all workspaces under `../workspaces/<repo>/` in
`fzf`. When you select a workspace, `jj-rex` starts or reuses the `rex` session for that workspace. It then switches
`rex` to that session.

```sh
jj-rex
```

### Delete a workspace

Run `jj-rex -d` or `jj-rex --delete` from the root repository to pick a workspace in `fzf`. If you pass a bookmark name,
`jj-rex` skips `fzf` and deletes that workspace directly. Do not run `jj-rex -d` if the workspace has files that you
want to keep. The command deletes the workspace directory from disk.

When you delete a workspace, `jj-rex` runs these steps in order:

1. `jj-rex` forgets the workspace in `jj` with `jj workspace forget`.
2. If a matching `rex` session exists, `jj-rex` kills it.
3. `jj-rex` deletes the workspace directory and any empty parent directories inside `../workspaces/<repo>/`.

```sh
jj-rex -d
jj-rex -d feat-auth
```
