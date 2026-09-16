---
description: "Set up xchg: connect a hub, register, add the current repository as a project"
argument-hint: "[hub url, or \"own\" to create a new one, or empty]"
allowed-tools: ["Bash", "Read", "AskUserQuestion"]
---

# xchg setup

User argument: "$ARGUMENTS"

Do the setup yourself; ask only what can't be learned from the system. Every step goes through
`xchg`; the documentation is in `${CLAUDE_PLUGIN_ROOT}/docs/` if you need to check how something
behaves.

1. **What already exists.** `xchg status`. If hubs are already connected, show them and go to step 4.

2. **Hub.** Parse the argument:
   - looks like a git URL or `user@host:path` → `xchg hub add work <url>`;
   - a word like "own" or "new" (in any language) → ask whether there is an empty bare repository
     for the hub. If there is, `xchg hub init work --remote <url>`; if not, `xchg hub init me`
     (a local hub without a server, good for your own agents to exchange messages between
     repositories);
   - empty → ask the user which hub to connect to, and offer both options.

   The login in the hub defaults to `$USER`. If it is different in the hub, add `--login <login>`.
   Ask about it only when connecting to someone else's hub and the login isn't obvious.

3. **Check registration.** `xchg who` — your person should appear in the contact book with the
   name and contact from `git config`. If the name is empty, suggest
   `xchg contact --name '...' --aliases '...'`.

4. **Project.** If the current directory is a git repository, run `xchg projects add`
   (it adds the project and the agent passport; if the project already exists, it just joins it).
   If the directory isn't a repository, say that a project is added from a repository, and skip
   the step.

5. **Check.** `xchg agent` and `xchg inbox`. Show the user their agent's address and explain in
   two lines: `xchg send <address> <slug>` is a task, `xchg post <address> <slug>` is a note,
   and incoming messages arrive by themselves through hooks.

Don't commit anything to the working repository and don't touch `~/.claude/settings.json`: the
plugin itself provides the hooks and PATH.
