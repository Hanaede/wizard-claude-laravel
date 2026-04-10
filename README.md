# Claude Commands — Personal

Comandos personalizados de Claude Code para uso personal.

---

## Comandos disponibles

| Comando | Descripción |
|---------|-------------|
| `/nuevo-proyecto` | Wizard completo para iniciar un proyecto Laravel 13 + React, con deploy a tu servidor y repo en tu GitHub personal |

---

## Instalación

### Opción 1 — Pedírselo directamente a Claude en VS Code

Abre Claude Code en VS Code y escribe exactamente este mensaje:

```
Instala mis comandos personalizados de Claude:
clona el repositorio Hanaede/wizard-claude-laravel usando gh CLI,
copia todos los archivos .md de su raíz a ~/.claude/commands/
(crea la carpeta si no existe), y elimina el repositorio clonado.
```

Claude ejecutará los pasos automáticamente.

---

### Opción 2 — Instalación manual (una línea)

```bash
gh repo clone Hanaede/wizard-claude-laravel /tmp/wizard-claude-laravel \
  && mkdir -p ~/.claude/commands \
  && cp /tmp/wizard-claude-laravel/*.md ~/.claude/commands/ \
  && rm -rf /tmp/wizard-claude-laravel \
  && echo "Comandos instalados correctamente en ~/.claude/commands/"
```

> Si aún no has iniciado sesión en GitHub, ejecuta primero `gh auth login`.

---

## Actualización

Para actualizar los comandos cuando haya nuevas versiones, repite el proceso de instalación. El comando siempre descarga la versión más reciente.

---

## Añadir nuevos comandos

1. Crea un archivo `nombre-comando.md` en la raíz de este repositorio
2. Haz commit y push
3. Actualiza en cualquier máquina ejecutando el comando de instalación de nuevo

Los archivos `.md` en `~/.claude/commands/` se convierten automáticamente en comandos `/nombre-comando` disponibles en Claude Code.
