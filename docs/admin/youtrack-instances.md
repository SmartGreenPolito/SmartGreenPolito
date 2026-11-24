# YouTrack Instances

This page is for the people who administer the software unit's tools. It lists
the YouTrack servers the unit uses today and explains how to connect
Claude Code to them.

## Current setup

For now, each of the five teams has its **own YouTrack Cloud instance**:

| Team | YouTrack instance |
| --- | --- |
| Team 1 | https://smartgreen-greenhouse-team-1.youtrack.cloud |
| Team 2 | https://smartgreen-greenhouse-team-2.youtrack.cloud |
| Team 3 | https://smartgreen-greenhouse-team-3.youtrack.cloud |
| Team 4 | https://smartgreen-greenhouse-team-4.youtrack.cloud |
| Team 5 | https://smartgreen-greenhouse-team-5.youtrack.cloud |

Each instance holds the user stories assigned to that team and the tasks the
team creates from them. The backlog of user stories is not in YouTrack: the
Project Manager keeps it on GitHub (see
[How We Work](../team-guidelines/how-we-work.md#5-backlog-user-stories-and-tasks)).

The five instances are separate. Users, permissions, projects, and tokens are
managed in each instance on its own. A change in one instance does not reach the
others.

> This setup may change. If the instances are merged, renamed, or a team is
> added, update the table above and the MCP setup below.

## Connecting Claude Code (optional)

Each instance exposes an MCP endpoint at `<instance URL>/mcp`. With it,
Claude Code can read and update issues on that instance.

YouTrack does not support the browser login that Claude Code uses by
default (it lacks dynamic client registration), so each connection needs a
**permanent token**.

### 1. Create a token on each instance

On every instance: **Profile → Account Security → New token…**, with the
**YouTrack** scope. Use the same name everywhere, for example
`claude-code-mcp`. Add the machine to the name if you use more than one
computer (`claude-code-mcp-macbook`), so you know which token to revoke if a
device is lost.

A token sees only what its user can see on that instance.

### 2. Register the five servers

Run this in your terminal from the repository root. It asks for each token
without showing it and keeps it out of the shell history:

```zsh
for n in 1 2 3 4 5; do
  read -s "tok?Token Team $n: "; echo
  claude mcp add --scope local --transport http youtrack-team$n \
    https://smartgreen-greenhouse-team-$n.youtrack.cloud/mcp \
    --header "Authorization: Bearer $tok"
done
unset tok
claude mcp list | grep youtrack
```

All five servers should show `✔ Connected`. Start a new Claude Code session
to use them.

`--scope local` keeps the configuration on your machine (in
`~/.claude.json`) and limits it to this repository. **Never commit tokens**:
do not put them in a `.mcp.json` file in the repository.

### Replacing a token

To replace the token of one or more instances (for example after revoking it),
remove the server and add it again. Change the list of team numbers as needed:

```zsh
for n in 1 2; do
  claude mcp remove --scope local youtrack-team$n 2>/dev/null
  read -s "tok?New token Team $n: "; echo
  claude mcp add --scope local --transport http youtrack-team$n \
    https://smartgreen-greenhouse-team-$n.youtrack.cloud/mcp \
    --header "Authorization: Bearer $tok"
done
unset tok
```

Revoke the old token in YouTrack under **Account Security**.
