---
description: Crea un git worktree aislado en .trees/[nombre] y ejecuta ahí el requerimiento indicado
argument-hint: <requerimiento a ejecutar en el worktree>
---

Requerimiento del usuario: $ARGUMENTS

Sigue estos pasos en orden:

1. **Define el nombre.** Deriva un nombre corto en kebab-case (2-4 palabras, minúsculas, sin acentos ni espacios) a partir del requerimiento. Ejemplo: "agregar sonido al romper líneas" → `sonido-lineas`.

2. **Crea el worktree.** Desde la raíz del repo, ejecuta:

   ```bash
   git worktree add .trees/[nombre]
   ```

   Si `.trees/[nombre]` ya existe, elige otro nombre. No sobrescribas ni borres worktrees existentes.

3. **Ignora `.trees/`.** Si `.trees/` no está en `.gitignore`, agrégalo para que el contenido de los worktrees no aparezca como cambios sin seguir.

4. **Trabaja solo dentro del worktree.** Ejecuta el requerimiento usando rutas absolutas bajo `<raíz-del-repo>/.trees/[nombre]/`. Reglas:
   - No edites archivos del directorio principal.
   - Para git usa `git -C .trees/[nombre] ...`.
   - Respeta las convenciones de `CLAUDE.md` (JS vanilla, sin dependencias ni build).
   - Para probar, abre `.trees/[nombre]/index.html` en el navegador.

5. **Commit en la rama del worktree.** Al terminar, haz commit de los cambios dentro del worktree. No hagas push ni merge sin que el usuario lo pida.

6. **Reporta.** Indica el nombre del worktree, su ruta, su rama, un resumen de los cambios y cómo integrarlos o eliminarlos:

   ```bash
   git merge [rama]
   git worktree remove .trees/[nombre]
   ```

Si `$ARGUMENTS` está vacío, pregunta al usuario qué requerimiento ejecutar antes de crear nada.
