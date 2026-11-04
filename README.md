# Android Skills

Reusable Android development skills for AI coding agents. Each skill lives in its own directory and contains a `SKILL.md` file.

## Available skills

| Skill | Description |
| --- | --- |
| [`r8-dex-optimization`](r8-dex-optimization/SKILL.md) | Diagnose and improve R8/DEX optimization configuration for Android apps. |

## Install a skill

### Android Studio and Android CLI

Android Studio recognizes skills in `.android-studio/skills/` and `.agents/skills/`, either in your home directory or at the root of a project. Android CLI chooses destinations based on the target agent it detects; `--project` scopes its operation to a project. Paths shown here are the common Android Studio locations:

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

### Codex

For a user-wide Codex install, use `~/.agents/skills/` (Windows: `%USERPROFILE%\.agents\skills\`). For a project-only install, use `<project-root>/.agents/skills/`.

### Android CLI catalog

`android skills list` shows skills available through Android CLI. `android skills add <skill-id>` installs a skill from Android's catalog for detected agents; add `--agent=codex` to target Codex or `--project=<project-root>` to scope it to a project. This command does not install arbitrary skills from this repository, so use the copy steps above for `r8-dex-optimization`.

## Verify installation

Confirm that the destination contains `<skill-name>/SKILL.md`. In Android CLI, run `android skills list` to inspect catalog skills and agent installation status. Android Studio and compatible agents discover custom skills from the supported directories above.

## License

This repository is distributed under the MIT License. See [LICENSE](LICENSE).

---

# Android Skills (Español)

Skills reutilizables para desarrollo Android con agentes de programación. Cada skill vive en su propio directorio e incluye un archivo `SKILL.md`.

## Skills disponibles

| Skill | Descripción |
| --- | --- |
| [`r8-dex-optimization`](r8-dex-optimization/SKILL.md) | Diagnostica y mejora la configuración de optimización R8/DEX en aplicaciones Android. |

## Instalar una skill

### Android Studio y Android CLI

Android Studio reconoce skills en `.android-studio/skills/` y `.agents/skills/`, tanto en el directorio personal como en la raíz de un proyecto. Android CLI elige destinos según el agente detectado; `--project` limita la operación a un proyecto. Estas son las ubicaciones globales habituales de Android Studio:

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

### Codex

Para instalarla globalmente en Codex, usa `~/.agents/skills/` (en Windows: `%USERPROFILE%\.agents\skills\`). Para instalarla solo en un proyecto, usa `<raíz-del-proyecto>/.agents/skills/`.

### Catálogo de Android CLI

`android skills list` muestra las skills disponibles mediante Android CLI. `android skills add <id-de-skill>` instala una skill del catálogo de Android para los agentes detectados; agrega `--agent=codex` para elegir Codex o `--project=<raíz-del-proyecto>` para limitar la instalación a un proyecto. Este comando no instala skills arbitrarias de este repositorio; para `r8-dex-optimization`, usa los pasos de copia anteriores.

## Comprobar la instalación

Confirma que el destino contiene `<nombre-de-skill>/SKILL.md`. En Android CLI, ejecuta `android skills list` para consultar las skills del catálogo y su estado por agente. Android Studio y otros agentes compatibles detectan skills propias desde los directorios admitidos indicados arriba.

## Licencia

Este repositorio se distribuye bajo la licencia MIT. Consulta [LICENSE](LICENSE).
