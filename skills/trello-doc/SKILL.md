---
name: trello-doc
description: "Trigger: When user requests documenting development or adjustments to Trello cards. Integrates development closures with Trello API."
license: MIT
metadata:
  author: frontend_rh
  version: "2.0"
---

# Skill: trello-doc

## Activation Contract

Activate this skill upon finishing any module development or bug adjustment, when the user requests documenting the closure or updates on a Trello card.

## Hard Rules

- **Project Scoping**: ALWAYS save engram memories and session summaries (`mem_save`, `mem_session_summary`) under the active explicit project name (`backend_rh` or `frontend_rh`). NEVER save under the generic `antigravity` name.
- **Order of Closure**: Follow the exact sequence:
  1. Verify code completeness and compilation.
  2. Prompt the user for the Trello card ID/URL: *"¿Cuál es la tarjeta de Trello para documentar este trabajo?"*
  3. Wait for user response. Do NOT run the script before acquiring the card ID/URL.
  4. Perform local engram save (`mem_save`).
  5. Sincronizar engram cloud (`engram sync --cloud`).
  6. Execute the Trello documentation PowerShell script.
  7. Finalize session with summary (`mem_session_summary`).
- **Execution Command**: ALWAYS invoke the Trello script using:
  `powershell -File C:\laragon\www\rh_new\.agents\scripts\trello_comment.ps1 -Card "<CARD_ID_OR_URL>" -Text "<DOCUMENTATION_TEXT>"`

## Decision Gates

| Task Type | Documentation Template | Focus |
| --- | --- | --- |
| `modulo_nuevo` | **🚀 Nuevo Desarrollo: [Módulo]**<br>- Proceso/Funcionamiento<br>- Detalles técnicos (endpoints, tablas, rutas) | Architectural flow & features |
| `ajuste` | **🔧 Ajuste Realizado: [Ajuste]**<br>- Problema reportado<br>- Ajuste aplicado (archivos, lógica, queries) | Original issue vs fix details |

## Execution Steps

1. Verify functional correctness of all modified files.
2. Ask the user for the Trello card identifier and wait for their input.
3. Once received, format the documentation text according to the selected template.
4. Dispatch `mem_save` locally and sync to cloud.
5. Invoke the Trello comment script passing the card parameter and formatted text.
6. Write the final `mem_session_summary` and conclude the session.

## Output Contract

Produce the following outputs:
- Formatted markdown comment string for Trello.
- Successful PowerShell script execution log.
- Completed session summary.
