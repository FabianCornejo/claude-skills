# claude-skills

Reusable skills for [Claude](https://claude.ai) that work in both **Claude Code** and **claude.ai**.

A skill is a folder with a `SKILL.md` file. Claude reads only the skill's short description and loads the full instructions on its own when a request matches. You don't need to type a command.

| Skill | What it does |
|---|---|
| [`autopilot`](autopilot/SKILL.md) | Autopilot for recurring, multi-step tasks. Turns them into a saved runbook and runs them the same way every time, with verification and a run log. |

---

## Autopilot

Autopilot for the long task you repeat every week and keep postponing.

**How it works**

1. **Find the runbook.** It looks for a saved runbook for the routine. If one exists, it asks only what changed this time.
2. **Design it once.** On the first run it asks one round of questions (inputs, steps, definition of done, decision points), writes the runbook and asks you to approve it.
3. **Execute in order.** It runs the steps with a visible task list. It never skips a step silently, works on copies of your files and asks before anything irreversible (sending, overwriting, deleting).
4. **Verify.** It actually checks the result (totals, dates, comparison with the last run). It never treats "the steps ran" as "done".
5. **Log and improve.** It appends to the runbook's log and proposes improvements when a step needed a workaround.

**Where runbooks are stored.** The skill detects the environment and uses exactly one location:

| Environment | Runbook location |
|---|---|
| Claude Code, or a chat with a local folder connected | `runbooks/<name>.md` in the project folder |
| claude.ai chat inside a Project | Project document `runbooks/<name>.md` |
| Plain chat with no persistent storage | You keep the file. It asks you to paste it next time. |

**Try it**

> Every Monday I update the valuation model with closing prices, refresh the trading multiples and write a one-page summary. Standardize this.

The skill responds in the language you use, so Portuguese works too: *"Todo fechamento de mês eu atualizo a base, refaço o DRE gerencial e monto o resumo de variações. Padroniza isso."*

---

## Installation

Claude Code and claude.ai are separate systems. Install the skill in each one you use.

### Claude Code (all projects)

Put the skill folder in your **personal** skills directory:

```
~/.claude/skills/autopilot/SKILL.md          # macOS / Linux
%USERPROFILE%\.claude\skills\autopilot\SKILL.md   # Windows
```

**Windows (PowerShell).** Download `autopilot.zip` from the [latest release](../../releases/latest), then run:

```powershell
Expand-Archive "$env:USERPROFILE\Downloads\autopilot.zip" -DestinationPath "$env:USERPROFILE\.claude\skills" -Force
```

**macOS / Linux.** Clone the repository and copy the folder:

```bash
git clone https://github.com/FabianCornejo/claude-skills.git
mkdir -p ~/.claude/skills && cp -r claude-skills/autopilot ~/.claude/skills/
```

Start a **new** Claude Code session, because skills are loaded at startup. To install the skill for a single project only, use `<project>/.claude/skills/` instead.

### claude.ai (web, desktop, mobile)

1. Download `autopilot.zip` from the [latest release](../../releases/latest).
2. In claude.ai, open **Settings → Capabilities → Skills** and upload the zip. Code execution must be enabled for skills to be available.

Skills uploaded to claude.ai are tied to your account and available in every chat and Project. If your Claude Code is signed in with the same account, they may also be synced to it. In that case, keep a single source of truth so you don't end up with two diverging copies.

---

## Credits

Inspired by a post from [@gut1n](https://instagram.com/gut1n) (*Curso de Claude*). The skill text in this repository is an original implementation.

## License

[MIT](LICENSE). Use it, adapt it, share it. Issues and pull requests are welcome.

---

<details>
<summary>🇧🇷 Em português</summary>

Skills reutilizáveis para o Claude, que funcionam no **Claude Code** e no **claude.ai**.

**autopilot.** Transforma uma tarefa longa e recorrente em um roteiro salvo e o executa sempre do mesmo jeito, com verificação no final e registro de cada execução. Ela é acionada sozinha quando você descreve uma rotina ("toda semana eu...", "roda de novo o fechamento") e responde em português.

**Instalação no Claude Code (Windows).** Baixe o `autopilot.zip` na [última release](../../releases/latest) e rode no PowerShell:

```powershell
Expand-Archive "$env:USERPROFILE\Downloads\autopilot.zip" -DestinationPath "$env:USERPROFILE\.claude\skills" -Force
```

Depois, abra uma nova sessão do Claude Code.

**Instalação no claude.ai.** Envie o mesmo zip em **Configurações → Capacidades → Skills**.

</details>
