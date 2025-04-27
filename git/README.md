# GIT

## Configurar Git:
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@example.com"
```

## Crear nueva rama:
```bash
git checkout -b <nombre_rama>
```
## Para ver los commits y conseguir los IDs:
```bash
git log --oneline
```

## Cambiar de rama:
```bash
git checkout <nombre_rama>
```

## Cambiar el Comportamiento por Defecto de git pull
Por defecto git pull hace un merge, pero puedes cambiarlo a rebase:
```bash
git config --global pull.rebase true
```
### O para un repositorio específico:
```bash
git config pull.rebase true
```
Puedes también usar:
```bash
git pull --rebase
```
para hacerlo solo una vez.

## Cambiar a un Commit Anterior
Para volver temporalmente a un commit anterior (sin perder historial):
```bash
git checkout <id_commit>
```

### Para resetear tu proyecto a un commit anterior de forma permanente:
#### Manteniendo los cambios locales:
```bash
git reset --soft <id_commit>
```
#### Descartando los cambios locales:
```bash
git reset --hard <id_commit>
```

## Rebase: Reescribir el Historial de una Rama
### Rebasar cambios de main a tu rama actual:
```bash
git fetch origin
git rebase origin/main
```
### Rebase interactivo para reordenar, combinar o editar commits:
```bash
git rebase -i HEAD~n
```
(donde n es el número de commits hacia atrás que quieres modificar)

## Stash: Guardar Cambios Temporalmente
### Guardar cambios sin hacer commit:
```bash
git stash
```
### Recuperar los cambios:
```bash
git stash pop
```
### Ver lista de stashes guardados:
```bash
git stash list
```

## Crear un tag anotado (más recomendado):
```bash
git tag -a v1.0.0 -m "Primer lanzamiento"
```
### Subir los tags al remoto:
```bash
git push origin --tags
```

## Cherry-pick: Aplicar un Commit Específico a Otra Rama

### Aplicar un commit individual en tu rama actual:
```bash
git cherry-pick <id_commit>
```

## Recuperar Commits Perdidos
A veces puedes perder commits debido a un `reset --hard`, `rebase` fallido o error humano.  
Afortunadamente, Git guarda un historial interno de los movimientos de `HEAD` que puedes consultar con `git reflog`.

### Pasos para recuperar un commit perdido:
### 1. Ver el historial de `HEAD`:
```bash
git reflog
```
Ejemplo de salida:
```
d1f8eac HEAD@{0}: commit: corregir validación de formulario
7f3c42e HEAD@{1}: reset: moving to HEAD~1
43a8e2b HEAD@{2}: commit (amend): corregir typo en login
```

### 2. Identificar el commit que quieres recuperar.
### 3. Aplicar el commit perdido usando `cherry-pick`:
```bash
git checkout <rama_destino>
git cherry-pick <id_del_commit>
```
Consejo:
Después de usar reflog, puedes hacer limpieza de referencias obsoletas o Limpiar objetos huérfanos con:
```bash
git gc --prune=now
```

## git bisect (Advanced)
git bisect sirve para encontrar en qué commit específico se introdujo un error (bug) en tu proyecto.

Lo que hace es usar una búsqueda binaria:
Git va navegando entre tus commits (de forma automática) hasta encontrar el primer commit malo.

¿Cuándo usar git bisect?

**1. Tu aplicación funcionaba hace una semana, pero hoy falla.**

**2. No sabes en cuál de los 100 commits recientes se rompió.**

**3. No quieres revisar uno por uno manualmente (sería una locura).**

Entonces, ¡usas git bisect para que Git te ayude a descubrirlo!

### Como se usa
1. Iniciar bisect
    ```bash
    git bisect start
    ```
2. Decirle cuál commit está malo (bug presente)

    (Usualmente, tu HEAD actual, que ya sabes que falla.)
    ```
    git bisect bad
    ```

3. Decirle cuál commit está bueno (sin bug)

    (Elegir un commit antiguo donde recuerdes que todo funcionaba.)
    ```bash
    git bisect good <id_commit_bueno>
    ```
4. Git empieza la búsqueda

    Git se moverá automáticamente a un commit intermedio entre el "bueno" y el "malo".
    Ahí debes probar manualmente si ese commit funciona o no:

    Si funciona, le dices
    ```bash
    git bisect good
    ```
    Si falla, le dices
    ```bash
    git bisect bad
    ```
    Así Git ajusta la búsqueda para acercarse más rápido al commit culpable.
5. Cuando encuentre el commit responsable, Git te mostrará algo como:
    ```bash
    <id_del_commit> is the first bad commit
    ```
    ¡Y listo! Encontraste exactamente el commit que introdujo el problema. 🔥
6. Salir de bisect

    Cuando termines, vuelves a la normalidad con:
    ```bash
    git bisect reset
    ```
### Resumen git bisect
```bash
git bisect start
git bisect bad             # El commit donde ves el error
git bisect good <id_bueno>  # Un commit donde todo funcionaba

# Git te moverá a un commit medio -> pruebas -> indicas si está bueno o malo
git bisect good             # si funciona
git bisect bad              # si falla
# Repetir hasta que Git encuentre el commit problemático

git bisect reset
```

## git fsck
git fsck significa "file system check".
Es un comando de diagnóstico en Git que verifica la integridad de toda la base de datos de tu repositorio.

1. Busca objetos corruptos (commits, blobs, trees, tags).

2. Detecta referencias rotas.

3. Te avisa de problemas ocultos que podrían hacer que el repo no funcione bien en el futuro.

En otras palabras: es como un escáner de salud para tu repositorio.

```bash
git fsck
```
Git revisa todo y muestra resultados.
Si no hay problemas, no mostrará nada raro (solo "Verifying objects..." y fin).

### ¿Qué puede reportar?

1. Dangling commits (commits que no pertenecen a ninguna rama actual, como commits huérfanos).

2. Dangling blobs (archivos que existieron pero ahora no tienen referencia).

3. Broken links (referencias a objetos inexistentes).

4. Corrupt objects (muy raro, pero puede pasar si tu disco tuvo errores, por ejemplo).

#### Ejemplo de salida

```bash
dangling commit abc1234 Commit sin referencia
dangling blob def5678 Archivo perdido
missing tree 9876543 Árbol inexistente
```
#### ¿Cuándo usar git fsck?

1. Si sospechas que el repositorio puede estar corrupto.

2. Después de operaciones complicadas (filter-branch, rebase, gc).

3. Antes de hacer un backup importante.

4. Cuando ves errores raros al hacer git pull, git fetch, etc.

#### ¿Qué hacer si encuentro problemas?
1. Dangling commits/blobs: no necesariamente es malo. Git limpia automáticamente estos objetos no referenciados después de un git gc (garbage collection).

2. Objetos corruptos o faltantes: en ese caso, es serio. Puede que necesites restaurar desde un backup o clonar de nuevo el repo si está en remoto.

#### Comandos utiles
- Limpiar objetos huérfanos:
```bash
git gc --prune=now
```

- Ver objetos huérfanos en detalle:
```bash
git fsck --full --dangling
```

## Flujos de Trabajo Recomendados

1. Gitflow (Popular en equipos grandes):

    - main: versión estable.

    - develop: integración de cambios antes de producción.

    - feature/xxxx: ramas de nuevas funciones.

    - release/xxxx: preparación para lanzamientos.

    - hotfix/xxxx: arreglos críticos directamente sobre producción.

2. Trunk-Based Development (Más ágil):

    - Todos trabajan sobre main o una rama corta y hacen merge continuo.