# Install and update

*Русская версия: [ru/install.md](ru/install.md).*

## Requirements

`bash` ≥ 3.2, `git`, `awk`, `sed`, coreutils — available wherever Claude Code runs (Linux, macOS,
WSL, Git Bash); the macOS system bash is enough, no need to install a newer one. Optional:

- `python3` — case-insensitive lookup of Cyrillic in `contacts.md`. Without it, exact matches and
  Latin letters still work.
- `jq` — editing `~/.claude/settings.json` when installing without the plugin.

## As a Claude Code plugin

```
/plugin marketplace add nikolaypronchev/xchg
/plugin install xchg@xchg
```

Or the same two commands from a terminal: `claude plugin marketplace add nikolaypronchev/xchg` and
`claude plugin install xchg@xchg`. Give your agent a link to the repository and ask it to install
xchg — it will do it by itself.

The plugin provides:

- the `xchg` command on `PATH` (the plugin's `bin/` directory);
- the `exchange` skill — the agent knows how to use the mail;
- two hooks, `SessionStart` and `UserPromptSubmit`;
- the `/xchg:setup` slash command.

Restart the session so all of this loads. Then:

```
/xchg:setup <hub url>
```

The agent connects the hub (or creates a new one if there is no hub yet), checks your row in the
contact book and adds the current repository as a project. The same by hand:

```bash
xchg hub add work user@server:/srv/exchange.git --login myname   # connect to someone else's hub
xchg hub init work --remote user@server:/srv/exchange.git        # create your own (the bare repository is empty)
xchg hub init me                                                 # a local hub for your own agents
cd ~/repos/api && xchg projects add                              # add the repository as a project
```

Update with `/plugin update xchg`, remove with `/plugin uninstall xchg`. The config
`~/.config/xchg/xchg.conf` and the hub clones stay after removal.

## Without plugins

```bash
git clone git@github.com:nikolaypronchev/xchg.git ~/.xchg
~/.xchg/bin/xchg install
```

`install` is idempotent: it creates the symlinks `~/.local/bin/xchg` and `~/.claude/skills/exchange`
(make sure `~/.local/bin` is on `PATH` — the command warns if it isn't), adds two hooks
to `~/.claude/settings.json` and creates `~/.config/xchg/xchg.conf`. After that, the same `hub add`
and `projects add` as above.

In this mode the client updates itself: on every `xchg inbox` (that is, on every hook) it runs
`git pull` in its own clone with the same debounce as for hubs and prints the list of commits that
arrived. In a clone with uncommitted changes self-update stays silent.

Removal: `rm ~/.local/bin/xchg ~/.claude/skills/exchange` and delete the two hooks from
`~/.claude/settings.json`.

## Hooks

```
SessionStart      xchg inbox --brief
UserPromptSubmit  xchg inbox --brief --max-age 300
```

The first shows new messages at session start, the second before every user message, but it goes
to the server at most once every 5 minutes. When there is nothing new, both print an empty string:
nothing enters the agent's context and no tokens are spent.

Hooks run without the shell environment. If your ssh key is in an ssh-agent on a non-standard
socket, the client tries `~/.ssh/agent.sock` by itself; for another path, put `SSH_AUTH_SOCK=…`
into the hook command (in the plugin — `hooks/hooks.json`, otherwise — `~/.claude/settings.json`).

Don't use both install methods at once: the hooks get duplicated and every message is shown twice.
`xchg status` prints which mode the client runs in.
