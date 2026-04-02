

origin remote
upstream remote

## Grundlegende Befehle

```bash
git status              # Zeigt den aktuellen Stand (geänderte, staged, untracked Files)
git add .               # Alle Änderungen stagen
git add <file>          # Einzelnes File stagen
git commit -m "msg"     # Commit erstellen
git push                # Commits auf Remote pushen
git pull                # Remote-Änderungen holen und mergen
git fetch               # Remote-Änderungen holen, aber NICHT mergen
```

## Branching

```bash
git branch                    # Alle lokalen Branches anzeigen
git branch <name>             # Neuen Branch erstellen
git switch <name>             # Zu Branch wechseln
git switch -c <name>          # Branch erstellen und direkt wechseln
git branch -d <name>          # Branch löschen (nur wenn gemergt)
git branch -D <name>          # Branch force-löschen
```

## Merge vs. Rebase

### Merge

```bash
git merge <branch>
```

Fügt einen Branch in den aktuellen ein. Erstellt einen **Merge Commit**.  
Die History bleibt erhalten, wird aber nicht-linear.

```
main:    A --- B ------- M
                \       /
feature:         C --- D
```

### Rebase

```bash
git rebase main
```

Schreibt die Commits des aktuellen Branches so um, als wären sie **nach** dem Ziel-Branch entstanden. Ergibt eine **lineare, saubere History**.

```
# Vorher:
main:    A --- B
feature:  \-- C --- D

# Nachher:
main:    A --- B
feature:         \-- C' --- D'
```

> ⚠️ **Nie** auf einem geteilten/öffentlichen Branch rebasen. Nur auf lokalen Feature-Branches.

### Interactive Rebase

```bash
git rebase -i HEAD~3    # Die letzten 3 Commits interaktiv bearbeiten
```

Öffnet einen Editor. Befehle:

- `pick` – Commit behalten
- `squash` / `s` – Mit vorherigem Commit zusammenführen
- `reword` / `r` – Commit-Message ändern
- `drop` – Commit löschen

## Amend – Letzten Commit nachträglich ändern

```bash
git commit --amend -m "Neue Message"   # Nur Message ändern
git commit --amend --no-edit           # Files zum letzten Commit hinzufügen
```

> Vorher das gewünschte File mit `git add` stagen.  
> ⚠️ Nur bei noch **nicht gepushten** Commits verwenden. Sonst muss man `git push --force` nutzen, was die History auf dem Remote überschreibt.

## Stash – Änderungen zwischenspeichern

```bash
git stash               # Aktuelle Änderungen wegspeichern
git stash pop           # Zuletzt gestashte Änderungen wiederherstellen
git stash list          # Alle Stashes anzeigen
git stash drop          # Zuletzt gestashten Stash löschen
```

## Reset & Restore

```bash
git restore <file>              # Änderungen in einem File rückgängig machen
git restore --staged <file>     # File aus Staging entfernen (unstage)
git reset HEAD~1                # Letzten Commit rückgängig (Änderungen bleiben)
git reset --hard HEAD~1         # Letzten Commit und Änderungen löschen ⚠️
```

## Log – History anzeigen

```bash
git log                         # Alle Commits
git log --oneline               # Kompakte Ansicht
git log --oneline --graph       # Als ASCII-Graph
git log --oneline -10           # Letzte 10 Commits
```

## Hilfreiche Extras

```bash
git diff                        # Unstaged Änderungen anzeigen
git diff --staged               # Staged Änderungen anzeigen
git cherry-pick <commit-hash>   # Einzelnen Commit von anderem Branch übernehmen
git blame <file>                # Zeigt, wer welche Zeile zuletzt geändert hat
```