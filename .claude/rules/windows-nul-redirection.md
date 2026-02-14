# Windows NUL Redirection

## Problem

In Git Bash / MSYS2 environments, `>nul` and `> nul` create a file named `nul` instead of redirecting to the Windows NUL device.

## Solution

| Environment      | Correct Syntax    |
| ---------------- | ----------------- |
| Git Bash / MSYS2 | `> /dev/null`     |
| cmd / PowerShell | `>nul` (no space) |

## Examples

```bash
# WRONG - creates nul file
echo test > nul
echo test >nul

# CORRECT - Git Bash
echo test > /dev/null

# CORRECT - cmd/PowerShell
echo test >nul
```

## Rule

Always use `/dev/null` for output suppression to ensure cross-platform compatibility.
