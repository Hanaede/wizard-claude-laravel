# Claude Commands — AppsurDesarrollo

Comandos personalizados de Claude Code para el equipo de AppsurDesarrollo.

---

## Comandos disponibles

| Comando | Descripción |
|---------|-------------|
| `/nuevo-proyecto` | Wizard completo para iniciar un proyecto Laravel 13 + React, con deploy a Siteground y repo en AppsurDesarrollo |

---

## Instalación

### Opción 1 — Pedírselo directamente a Claude en VS Code

Abre Claude Code en VS Code y escribe exactamente este mensaje:

```
Instala los comandos personalizados de AppsurDesarrollo:
clona el repositorio AppsurDesarrollo/claude-commands usando gh CLI,
copia todos los archivos .md de su raíz a ~/.claude/commands/
(crea la carpeta si no existe), y elimina el repositorio clonado.
```

Claude ejecutará los pasos automáticamente.

---

### Opción 2 — Instalación manual (una línea)

```bash
gh repo clone AppsurDesarrollo/claude-commands /tmp/appsur-claude-commands \
  && mkdir -p ~/.claude/commands \
  && cp /tmp/appsur-claude-commands/*.md ~/.claude/commands/ \
  && rm -rf /tmp/appsur-claude-commands \
  && echo "Comandos instalados correctamente en ~/.claude/commands/"
```

---

## Actualización

Para actualizar los comandos cuando haya nuevas versiones, repite el proceso de instalación. El comando de instalación siempre descarga la versión más reciente.

---

## Añadir nuevos comandos

1. Crea un archivo `nombre-comando.md` en la raíz de este repositorio
2. Haz commit y push
3. Los miembros del equipo actualizan ejecutando el comando de instalación

Los archivos `.md` en `~/.claude/commands/` se convierten automáticamente en comandos `/nombre-comando` disponibles en Claude Code.
