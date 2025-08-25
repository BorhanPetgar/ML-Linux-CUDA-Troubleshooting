To get to (attach to) one of those detached screens, you can use the `screen -r` command. Here's how:

## For the detached screens:

**Attach to the "llm_service" screen:**
```bash
screen -r 12345.llm_service
```
or just
```bash
screen -r 12345
```

**Attach to the "kb" screen:**
```bash
screen -r 123457.kb
```
or
```bash
screen -r 123457
```

## For the attached screens:

If you need to connect to one of the attached screens (shared session), use:
```bash
screen -x 123458.tts  # For the tts screen
```
or
```bash
screen -x 123477.omni  # For the omni screen
```

## Useful screen commands once attached:

- **Detach**: `Ctrl + A` then `D`
- **Create new window**: `Ctrl + A` then `C`
- **Switch between windows**: `Ctrl + A` then `N` (next) or `P` (previous)
- **List windows**: `Ctrl + A` then "`

## Additional options:

- To see all screens again: `screen -ls`
- To create a new screen: `screen -S session_name`

The numbers (12345, 123457, etc.) are the process IDs of the screen sessions, and the names after the dot are the custom names given when the screens were created.
