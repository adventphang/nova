Nova - AI assistant on Claude Code

## Prerequisite

1. Install Claude Code and pre-requisites
    - git, python3, venv, sqlite3

2. Configure claude code


    install plugins
        - context7@claude-plugins-official
        - telegram@claude-plugins-official

    configure claude code
        - "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
        - autoMemoryEnabled: false

    - configure claude code status line to show the current folder name, current model, and colour-coded current context usage in progress bar and percentage value

3. Install and configure telegram plugin

## Installing Nova

1. Create a folder for the assistant. For example `mkdir ~/nova`.
2. Go to the assistant folder `cd ~/nova`
3. Start Claude code with `claude`
4. Type `Read the file SETUP.md and follow every step in it to set up a 24/7 AI assistant. Ask me for confirmation before each major step.`
5. Once everything is set up, start the assistant by running `claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions`

