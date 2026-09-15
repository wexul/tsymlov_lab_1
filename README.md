Полошков Ярослав Витальевич, группа 220042-11, вариант 4, лабораторная №1

# Лабораторная работа №1. Система контроля версий Git

Дисциплина: «Методы и технологии программирования» (часть 1).
Репозиторий: https://github.com/wexul/tsymlov_lab_1

Вариант 4 по таблице индивидуальных вариантов:

| Сложность  | Задания |
|------------|---------|
| Средняя    | №4 — изменить файл, сделать второй коммит; №6 — слить ветку `feature` с основной; №10 — склонировать чужой репозиторий и изучить историю |
| Повышенная | №5 — переписать историю коммитов с `git rebase`; №9 — сформировать отчёт о коммитах с `git shortlog` |

Ниже по каждому заданию приведены выполненные команды и доказательство
(хэши коммитов в этом репозитории и/или файл-отчёт).

---

## Средняя сложность

### Задание 4. Изменить файл, сделать второй коммит

```bash
echo "print('Hello, Python')" > script.py
git add script.py
git commit -m "feat: create script.py"                 # 6418c72

echo "print('Second commit modification')" >> script.py
git add script.py
git commit -m "feat: update script.py for task 4"      # 8665863
```

Доказательство — `git log --oneline -- script.py`:

```
8665863 feat: update script.py for task 4
6418c72 feat: create script.py
```

Один и тот же файл `script.py` создан в коммите `6418c72` и изменён во втором коммите `8665863`.

### Задание 6. Слить ветку `feature` с основной

```bash
git checkout -b feature
echo "print('Force merge commit')" >> feature.py
git add feature.py
git commit -m "feat: force merge commit for task 6"    # e09768c (ветка feature)

git checkout main
git merge feature --no-ff -m "Merge branch 'feature'"  # 6209f1a
```

Доказательство — фрагмент `git log --oneline --graph main`:

```
*   6209f1a Merge branch 'feature'
|\
| * e09768c feat: force merge commit for task 6
|/
* bffc294 docs: generate git shortlog report for task 9
```

Слияние выполнено с `--no-ff`, поэтому создан настоящий merge-коммит `6209f1a`
с двумя родителями (`bffc294` и `e09768c`). Ветка `feature` сохранена в
репозитории и запушена на GitHub (`origin/feature` → `e09768c`); её коммит
входит в `main`.

### Задание 10. Склонировать чужой репозиторий и изучить историю

Склонирован репозиторий **https://github.com/octocat/Hello-World.git**:

```bash
git clone https://github.com/octocat/Hello-World.git external_repo
cd external_repo
git log --oneline -n 10
git log --oneline --graph --all
git log --format='%h | %an <%ae> | %ad | %s' --date=short
git shortlog -sne
git shortlog
git branch -a
git log --merges --oneline
```

Доказательство — файл [`repo_history.txt`](repo_history.txt): в нём адрес
репозитория, дата выполнения, **фактический вывод** всех перечисленных
команд и анализ истории.

Кратко по результатам: в ветке `master` три коммита (`553c207` → `7629413` →
`7fd1a60`), три автора (cameronmcefee, Johnneylee Jack Rollins, The Octocat —
по одному коммиту у каждого), история нелинейная — `7fd1a60` является
merge-коммитом pull request #6; кроме `master` есть не влитые ветки
`octocat-patch-1` и `test`; тегов нет.

---

## Повышенная сложность

### Задание 5. Переписать историю коммитов с `git rebase`

Демонстрация перебазирования ветки на основную с фиксацией состояния
до и после.

```bash
# 1. Ветка с коммитом, которую будем перебазировать
git checkout -b rebase_branch
echo "Rebase target" > rebase_file.txt
git add rebase_file.txt
git commit -m "feat: add rebase target file"                       # 9233eb4

# 2. Параллельный коммит в main, чтобы ветки разошлись
git checkout main
echo "Main progress" > main_progress.txt
git add main_progress.txt
git commit -m "feat: update main branch for rebase demonstration"  # e09f1db

# 3. Снимок истории ДО rebase
git log --oneline --graph --all > before_rebase.txt
git add before_rebase.txt
git commit -m "docs: save git log BEFORE rebase"                   # 35e66f5

# 4. Сам rebase
git checkout rebase_branch
git rebase main            # 9233eb4 -> переписан как 9f2f8f4 поверх 35e66f5

# 5. Fast-forward main на перебазированную ветку и снимок ПОСЛЕ
git checkout main
git merge rebase_branch    # fast-forward, main -> 9f2f8f4
git log --oneline --graph --all > after_rebase.txt
git add after_rebase.txt
git commit -m "docs: save git log AFTER rebase proving hash change" # 0e8ec26
```

Доказательство — файлы [`before_rebase.txt`](before_rebase.txt) и
[`after_rebase.txt`](after_rebase.txt):

* **до rebase** (`before_rebase.txt`) коммит `feat: add rebase target file`
  имеет хэш **`9233eb4`**, его родитель `4936f8a`, и он лежит в стороне от
  `main` (расходящиеся ветки);
* **после rebase** (`after_rebase.txt`) тот же коммит имеет новый хэш
  **`9f2f8f4`**, его родитель уже `35e66f5`, и он лежит линейно поверх
  `main` — история линеаризована.

Смена хэша объясняется тем, что `git rebase` создаёт новый коммит с тем же
содержимым, но другим родителем; старый объект `9233eb4` остаётся недостижимым.

Линейная история после rebase (`git log --oneline main`, фрагмент):

```
0e8ec26 docs: save git log AFTER rebase proving hash change
9f2f8f4 feat: add rebase target file
35e66f5 docs: save git log BEFORE rebase
e09f1db feat: update main branch for rebase demonstration
```

Примечание к файлам-снимкам: они сняты с ключом `--all`, поэтому в них видна
и старая параллельная линия `master` (`955fb2f`, `2738c63`, `e3eacce`,
`10cf8ff`, `ea316d6`) — дубликат истории, оставшийся от первоначальной
попытки выполнения заданий. Ветка `master` затем удалена с GitHub, эти
коммиты в текущем репозитории недостижимы. На доказательство это не влияет: перебазированный
коммит и его хэши (`9233eb4` → `9f2f8f4`) видны в обоих файлах.

Дополнительно ранее был выполнен интерактивный rebase со squash
(`git rebase -i HEAD~2`) поверх `a8a0364`: коммиты `5125113` `chore: dummy
commit 1` и `6c2ff09` `chore: dummy commit 2` объединены в один — `d809cb9`
`chore: dummy commit 1` (родитель `a8a0364`, файл с обеими строками).
Исходные объекты `5125113`/`6c2ff09` после squash стали недостижимыми и на
GitHub не отправлялись. Временный файл `rebase_test.txt` из той демонстрации
удалён из репозитория как не несущий пользы; основным доказательством задания
служат `before_rebase.txt` / `after_rebase.txt`.

После демонстрации ветка `rebase_branch` удалена (локально и на GitHub),
т.к. полностью влита в `main`.

### Задание 9. Сформировать отчёт о коммитах с `git shortlog`

```bash
git shortlog > shortlog.txt
git add shortlog.txt
git commit -m "docs: update shortlog report for task 9"
```

Доказательство — файл [`shortlog.txt`](shortlog.txt): вывод `git shortlog`
(автор, число коммитов, список сообщений). Отчёт обновлялся после слияния
(`008ff08`), после rebase (`c106b07`) и в финале работы, чтобы соответствовать
текущей истории.

---

## Структура репозитория

| Файл | Назначение |
|------|------------|
| `README.md` | Данные студента (первая строка) и отчёт по заданиям |
| `script.py` | Задание 4: файл, созданный первым коммитом и изменённый вторым |
| `feature.py` | Задание 6: файл, изменённый в ветке `feature` и влитый в `main` |
| `repo_history.txt` | Задание 10: адрес чужого репозитория, вывод `git log` / `git shortlog`, анализ |
| `before_rebase.txt` | Задание 5: `git log --oneline --graph --all` до rebase |
| `after_rebase.txt` | Задание 5: `git log --oneline --graph --all` после rebase |
| `rebase_file.txt` | Задание 5: файл коммита, который перебазировался (`9233eb4` → `9f2f8f4`) |
| `main_progress.txt` | Задание 5: файл параллельного коммита в `main`, из-за которого ветки разошлись |
| `shortlog.txt` | Задание 9: отчёт `git shortlog` |
| `.gitignore` | Стандартные исключения для Python-проекта (`__pycache__/`, `*.pyc`, `.env`) |

## Ветки

* `main` — основная ветка со всей историей;
* `feature` — ветка задания 6, слита в `main` merge-коммитом `6209f1a`.

Сообщения коммитов оформлены по Conventional Commits (`feat:`, `docs:`, `chore:`).
