# Android Skills

Reusable Android development skills for AI coding agents. Each skill lives in its own directory and contains a `SKILL.md` file.

## Available skills

| Skill | Description |
| --- | --- |
| [`r8-dex-optimization`](r8-dex-optimization/SKILL.md) | Diagnose and improve R8/DEX optimization configuration for Android apps. |
| [`aab-dex-checker`](aab-dex-checker/SKILL.md) | Audit Android App Bundles for DEX size and Android Vitals optimization signals. |
| [`android-video-automation`](android-video-automation/SKILL.md) | Explore Android apps, interact with devices or emulators, and record UI flows as videos. |

## Install a skill

### Claude Code, OpenCode, Cursor, Gemini CLI, and GitHub Copilot

Use the project-level locations below. `.agents/skills/` is shared by the agents that list it:

| Agent | Project-level skill locations |
| --- | --- |
| [Claude Code](https://code.claude.com/docs/en/skills) | `.claude/skills/` |
| [Codex](https://developers.openai.com/codex/skills) | `.agents/skills/` |
| [OpenCode](https://docs.opencode.ai/docs/skills/) | `.agents/skills/` or `.opencode/skills/` |
| [Cursor](https://cursor.com/docs/skills) | `.agents/skills/` or `.cursor/skills/` |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/using-agent-skills.md) | `.agents/skills/` or `.gemini/skills/` |
| [GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills) | `.agents/skills/` or `.github/skills/` |

To share a skill across agents that support `.agents/skills/`, copy its directory there. Use an agent-specific location from the table when needed. For example, on Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force .agents\skills | Out-Null
Copy-Item -Recurse .\r8-dex-optimization .agents\skills\
```

On macOS or Linux:

```sh
mkdir -p .agents/skills
cp -R r8-dex-optimization .agents/skills/
```

Replace `.agents/skills/` with the desired agent-specific path from the table to install for that agent only.

### Android Studio and Android CLI

Android Studio recognizes skills in `.android-studio/skills/` and `.agents/skills/`, either in your home directory or at the root of a project. Android CLI can list and install skills from its catalog; `--project` scopes installation to a project. Paths shown here are the common Android Studio locations:

| Operating system | User-wide location |
| --- | --- |
| Windows | `%USERPROFILE%\.android-studio\skills\<skill-name>\` |
| macOS / Linux | `~/.android-studio/skills/<skill-name>/` |

To install this repository's custom skill, clone the repository and copy its skill directory to the desired location. For example, on Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.android-studio\skills" | Out-Null
Copy-Item -Recurse .\r8-dex-optimization "$HOME\.android-studio\skills\"
```

On macOS or Linux:

```sh
mkdir -p ~/.android-studio/skills
cp -R r8-dex-optimization ~/.android-studio/skills/
```

For a project-only installation, copy the skill directory to `<project-root>/.android-studio/skills/` or `<project-root>/.agents/skills/`.

### Android CLI catalog

`android skills list` shows skills available through Android CLI. `android skills add <skill-id>` installs a skill from Android's catalog; use `--project=<project-root>` to scope installation to a project. This command does not install arbitrary skills from this repository, so use the copy steps above for `r8-dex-optimization`.

## Verify installation

Confirm that the destination contains `<skill-name>/SKILL.md`. In Android CLI, run `android skills list` to inspect available catalog skills and installation status. Android Studio and compatible agents discover custom skills from the supported directories above.

## License

This repository is distributed under the MIT License. See [LICENSE](LICENSE).

---

# Android Skills (Español)

Skills reutilizables para desarrollo Android con agentes de programación. Cada skill vive en su propio directorio e incluye un archivo `SKILL.md`.

## Skills disponibles

| Skill | Descripción |
| --- | --- |
| [`r8-dex-optimization`](r8-dex-optimization/SKILL.md) | Diagnostica y mejora la configuración de optimización R8/DEX en aplicaciones Android. |
| [`aab-dex-checker`](aab-dex-checker/SKILL.md) | Audita Android App Bundles por tamaño DEX e indicadores de optimización de Android Vitals. |
| [`android-video-automation`](android-video-automation/SKILL.md) | Explora aplicaciones Android, interactúa con dispositivos o emuladores y graba flujos de interfaz en video. |

## Instalar una skill

### Claude Code, OpenCode, Cursor, Gemini CLI y GitHub Copilot

Usa las ubicaciones a nivel de proyecto indicadas abajo. Los agentes que muestran `.agents/skills/` comparten esa carpeta:

| Agente | Ubicaciones de skills a nivel de proyecto |
| --- | --- |
| [Claude Code](https://code.claude.com/docs/en/skills) | `.claude/skills/` |
| [Codex](https://developers.openai.com/codex/skills) | `.agents/skills/` |
| [OpenCode](https://docs.opencode.ai/docs/skills/) | `.agents/skills/` o `.opencode/skills/` |
| [Cursor](https://cursor.com/docs/skills) | `.agents/skills/` o `.cursor/skills/` |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/using-agent-skills.md) | `.agents/skills/` o `.gemini/skills/` |
| [GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills) | `.agents/skills/` o `.github/skills/` |

Para compartir una skill entre los agentes que admiten `.agents/skills/`, copia su directorio allí. Si hace falta, usa una ubicación específica del agente de la tabla. Por ejemplo, en PowerShell de Windows:

```powershell
New-Item -ItemType Directory -Force .agents\skills | Out-Null
Copy-Item -Recurse .\r8-dex-optimization .agents\skills\
```

En macOS o Linux:

```sh
mkdir -p .agents/skills
cp -R r8-dex-optimization .agents/skills/
```

Para instalarla solo para un agente, sustituye `.agents/skills/` por la ruta específica de la tabla.

### Android Studio y Android CLI

Android Studio reconoce skills en `.android-studio/skills/` y `.agents/skills/`, tanto en el directorio personal como en la raíz de un proyecto. Android CLI puede consultar e instalar skills de su catálogo; `--project` limita la instalación a un proyecto. Estas son las ubicaciones globales habituales de Android Studio:

| Sistema operativo | Ubicación para el usuario |
| --- | --- |
| Windows | `%USERPROFILE%\.android-studio\skills\<nombre-de-skill>\` |
| macOS / Linux | `~/.android-studio/skills/<nombre-de-skill>/` |

Para instalar una skill propia de este repositorio, clónalo y copia el directorio de la skill al destino deseado. Por ejemplo, en PowerShell de Windows:

```powershell
New-Item -ItemType Directory -Force "$HOME\.android-studio\skills" | Out-Null
Copy-Item -Recurse .\r8-dex-optimization "$HOME\.android-studio\skills\"
```

En macOS o Linux:

```sh
mkdir -p ~/.android-studio/skills
cp -R r8-dex-optimization ~/.android-studio/skills/
```

Para instalarla solo en un proyecto, copia el directorio de la skill a `<raíz-del-proyecto>/.android-studio/skills/` o `<raíz-del-proyecto>/.agents/skills/`.

### Catálogo de Android CLI

`android skills list` muestra las skills disponibles mediante Android CLI. `android skills add <id-de-skill>` instala una skill del catálogo de Android; usa `--project=<raíz-del-proyecto>` para limitar la instalación a un proyecto. Este comando no instala skills arbitrarias de este repositorio; para `r8-dex-optimization`, usa los pasos de copia anteriores.

## Comprobar la instalación

Confirma que el destino contiene `<nombre-de-skill>/SKILL.md`. En Android CLI, ejecuta `android skills list` para consultar las skills disponibles en el catálogo y su estado de instalación. Android Studio y otros agentes compatibles detectan skills propias desde los directorios admitidos indicados arriba.

## Licencia

Este repositorio se distribuye bajo la licencia MIT. Consulta [LICENSE](LICENSE).
