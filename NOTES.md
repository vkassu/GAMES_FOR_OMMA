# NOTES.md — детали по играм и общим механизмам

Памятка для разработки. Высокоуровневая карта — в `CLAUDE.md`. Здесь подробности,
чтобы не перечитывать все файлы. Обновлять при существенных изменениях.

## Общие механизмы (инлайн-копии в каждой игре)

В проекте НЕТ общих JS/CSS файлов (правило «1 игра = 1 файл»). Поэтому одни и те же
блоки физически продублированы в каждой игре. Если меняешь общий механизм — меняешь
во всех файлах, где он есть.

### 1. Структура экранов (квиз-игры: naydi, zagadki, chasy, magazin, tatar; + chasy/labirint)
- `#startScreen` (выбор уровня/темы) → `#gameScreen` → `#endScreen`. Переключение классом `.hidden`.
- Функции: `showStart()`, `startGame(level)`, `renderQuestion()/renderRiddle()`, `onPick()`, `goNext()`, `showEnd()`.
- `matematika` и `pamyat` — «бесконечные», без start/end-экранов: верхний бар + левое меню режимов, всё на одном экране.

### 2. Верхний бар навигации
- `<header class="topbar">` с `#homeBtn` (🏠 К играм → index.html) и `.navright` (`#levelBtn` 🎚️, `#restartBtn` 🔄).
- `.navright` показывается только во время игры (класс `.hidden` снимается в `startGame`, ставится в `showStart`/`showEnd`).
- В matematika/pamyat бар называется `.top` (со счётчиками), лейбл тоже `🏠 К играм`.

### 3. Подтверждение выхода с середины
- Overlay `#confirmOverlay` + `askConfirm(msg, onYes)` + кнопки `#cfYes`/`#cfNo`.
- `midGame()` — игра в процессе (обычно `pos > 0`; в pamyat — `moves>0 && matched<totalPairs`).
- Вешается на `homeBtn`/`levelBtn`/`restartBtn` (и на смену уровня/набора в pamyat).

### 4. Конфетти (перед `</body>`, отдельный `<script>`)
- `<canvas id="confetti" style="position:fixed;inset:0;...z-index:60">` + IIFE, экспортит `window.fireConfetti()`.
- Эмодзи салюта: ⭐🌟💖💛✨🎉🌸. Вызов: на финале/победе `if(window.fireConfetti) fireConfetti();`.
- matematika: салют каждые 5 правильных подряд.

### 5. Копилка звёзд (тот же `<script>`, рядом с конфетти)
- `window.awardStars(game, n)` — пишет в `localStorage`:
  - `omma_stars_total` (накопительный счёт), `omma_best_<game>` (лучший рейтинг 1–3), `omma_plays_total`.
- Вызов в конце: `awardStars('<game>', stars.length)` (stars = строка из ⭐, `.length` = число звёзд).
- pamyat: `awardStars('pamyat', 2)` на победе. matematika: `awardStars('matematika', 1)` каждые 5 подряд.
- Лаунчер читает эти ключи → «⭐ N собрано» + бейджи (10/25/50/100 звёзд, 20 игр, 3★ в игре).

### 6. Имя и герой (персонализация)
- Лаунчер: форма `#setup` (имя `#nameInput` + кнопки `.hero-btn`), сохраняет `omma_name`, `omma_hero`. Приветствие `#hello`.
- Игры читают `omma_name` в `showEnd`: `endTitleEl.textContent = nm ? title.replace(/!$/, ', '+nm+'!') : title;`
- pamyat персонализирует победное сообщение (`msg`).

### 7. Темы оформления (перед `</body>`, отдельный `<script>`)
- IIFE инжектит `<style>` с правилами `html[data-theme="rainbow|flower|space"]` и ставит `data-theme` из `omma_theme`.
- Светлые темы меняют только фон; `space` (тёмный) ещё перекрывает цвет текста ВНЕ карточек на светлый
  (селекторы: h1, .subtitle, .lead, .hello, .progress, .hint, .question, .score-line, .switcher-label, + в tatar `.q-label`, `.fc-label`).
- Переключатель (`#themebar`) и сами CSS-правила для лаунчера — прямо в `index.html` (`<style>` + основной IIFE).

### 8. Переключатель игр (на финальных экранах)
- `<div class="switcher">` с `.game-pill` ссылками на ОСТАЛЬНЫЕ игры. При добавлении новой игры — дописать пилюлю во все switcher'ы.
- Иконки: 🧮 matematika, 🧠 pamyat, 🔍 naydi-lishnee, 🤔 zagadki, 🕐 chasy, 🛒 magazin, 🌀 labirint, 🗣️ tatar, 🔤 english, 🧱 kirpichi, ❌ krestiki-noliki.
- Переключатели есть в: naydi-lishnee, zagadki, chasy, magazin, labirint, tatar, english, kirpichi, krestiki-noliki (matematika/pamyat — без финального экрана).

### 9. Звук (правило iOS)
- Один переиспользуемый `AudioContext`; `ensureAudio()` вызывать внутри пользовательского жеста (клик уровня/ответа).
- `tone(freqs, dur, type)`, `playGood()`/`playRetry()` (+ `playCoin` в labirint, `playMatch`/`playWin` в pamyat).

## Детали по играм

- **matematika** — генераторы `genAdd/genSub/genMul/genDiv/genMix`; экранный нумпад + физические коды клавиш (Numpad работает без NumLock); серия 🔥, рекорд в `localStorage` `matematika_best`.
- **pamyat** — `THEMES` (animals/sweets/flowers/faces/transport, ≥20 эмодзи каждый), `MODES` (easy..impossible: pairs/cols/bw), `currentTheme`/`mode`; переворот карточек через rotateY; рекорд ходов на уровень `pamyat_best_<mode>`; выбор набора — `#themes`.
- **naydi-lishnee** — `QUESTIONS` (≈100, поле `level` 1–3, `type`: odd/match/common/sequence/attribute, `q`, опц. `prompt`{kind:big|row}, `items`, `answer`, `big`, `why`); `GAME_SIZE=15` случайных задач уровня.
- **zagadki** — `RIDDLES` (101 шт.), каждая `{q, a, opts:[[эмодзи,слово]×6]}`; `GAME_SIZE=12`. Только «старые» эмодзи (см. правило).
- **chasy** — `ALLOWED_MIN` по уровням; `makeQuestion` (циферблат + 4 варианта времени `Ч:ММ`); рисование стрелок на Canvas; `GAME_SIZE=10`.
- **magazin** — `PAY={1:10,2:50,3:100}`, цена < купюры, сдача = pay−price; обманки = цена/купюра/соседние; `PRODUCTS` (эмодзи ≤E12); `GAME_SIZE=10`.
- **labirint** («Помоги зайке найти морковку», файл остался `labirint.html`) — зайка 🐰 (игрок, `pr/pc`) ищет морковку 🥕 (цель, `gr/gc`). Генерация — рандомизированный Прим + braiding (`BRAID` по уровням → петли/несколько маршрутов), битовая маска N/E/S/W; `bfsFrom(start)` — расстояния. Кошка и мышка ставятся на два конца диаметра лабиринта (двойной BFS). Звёзды-собиралки `coins`(Set) + `visited` (след); звёзды финала по `totalMoves/totalOptimal`; 5 лабиринтов; управление: стрелки/WASD/ЦФЫВ, свайпы, экранный крестик `.dpad`. Иконка в лаунчере/переключателях осталась 🌀.
- **tatar** — `SETS` (11 тем, `[татарское, русское, эмодзи]`, 133 слова: частотное ядро Сводеша/A1 — pron «Я и ты», verbs «Действия», adj «Какой?» — + тематика: беседа, семья, животные, еда, цвета, числа, тело, природа). У служебных слов/глаголов эмодзи = `''`; карточка «Учить» прячет пустой `#fcEmoji` (`display:none`). Режим «Учить» (`#studyScreen`, карточки-перевёртыши) и «Квиз» (`#gameScreen`, направление tr/rt чередуется); квиз проходит ВСЕ слова темы. Татарские буквы ә/ө/ү/җ/ң/һ — обычный Unicode. Эмодзи показывается ТОЛЬКО в «Учить»; в квизе скрыт (картинка подсказывала ответ).
- **kirpichi** («Увернись от кирпичей») — аркада на реакцию. Canvas (логический `W×H` 360×470, CSS масштабирует; пойнтеры мапятся через `getBoundingClientRect`). Герой = `omma_hero`, внизу, двигается влево-вправо. Падают 🧱 (`bricks`), сложность растёт со временем (`fall`/`spawn` + `ramp*` по уровням `LEVELS`). Спец-кирпичи (поля `s/v/dmg/gray` на кирпиче): каждый 8-й — двойной (`2·BS`, скорость ×1.5, −2 ❤️), каждый 5-й маленький — серый (скорость ×3, рисуется Canvas'ом в `drawGrayBrick` — эмодзи 🧱 не перекрасить, `ctx.filter` нет на старых iOS). Режимы (`gameMode`, выбор `.mode-btn` на старте): `'fall'` — падают сверху; `'roll'` — катятся по земле с двух сторон (поля `roll/dir` на кирпиче, y на земле, только маленькие обычные без спец-кирпичей, `spawnEvery ×2` реже, вращение как у колеса, пороги звёзд 12/6 вместо 25/12, в `scoreLine` приписка «по земле»; на `level 2` вторым потоком (`fallTimer`, `spawnSmallFalling`) сверху падают маленькие обычные). 3 жизни-сердечка, неуязвимость `invuln` после удара (мигание), `lives` клампится в 0. Управление: drag по канвасу, кнопки ◀▶ (`holdBtn`), клавиши (`e.code`: Arrow/KeyA/KeyD). Прыжок: кнопка ▲ / Space / ArrowUp / KeyW → `jump()`; физика `yOff/vy/onGround` + `GRAV/JUMP_V` (высота ≈97px), хитбокс и отрисовка героя — по `playerY + yOff`. Звёзды по `dodges` (≥25→3, ≥12→2, иначе 1), ключ `kirpichi`. Пауза на overlay подтверждения (`paused`).
- **english** — клон tatar: `SETS` те же 8 тем и те же слова (`[английское, русское, эмодзи]`, ~99 слов), направление er/re. Менять словарь — синхронно с tatar (одинаковый порядок/эмодзи, разная только колонка перевода). Синяя палитра (`--accent:#2563eb`), ключ звёзд `english`, иконка 🔤. Эмодзи так же только в «Учить».
- **krestiki-noliki** («Крестики-нолики») — три размера доски (`N=3`, `4` или `5`); длина выигрышной линии `K = kForSize(N)` — `K=3` для 3×3 и 5×5, `K=4` для 4×4. `WIN_LINES` считается динамически в `computeWinLines(n,k)` (строки, столбцы, обе диагонали → 8 линий для 3×3+K3, 10 для 4×4+K4, 48 для 5×5+K3). Значения N и K пересчитываются в `buildBoard(n)` при каждом старте сессии; `checkWinner`/`findLineMove` полностью обобщены под `line.length`, поэтому смена K не трогает игровую логику. В счёт-полоске показывается «📐 N×N · K в ряд». `buildBoard(n)` пересоздаёт `.cell`-кнопки под текущий N, ставит `boardEl[data-n]` (CSS переключает grid-template-columns и размер), инициализирует `board = new Array(N*N).fill(0)` и вешает click-обработчики. `checkWinner` и `findLineMove` обобщены под любую длину линии. Выбор размера — селектор `.size-picker` на стартовом экране, значение хранится в `localStorage['krestiki_size']`. Игрок всегда `1=❌` и ходит первым. 3 режима (`MODES.friend/cat/fox`): `friend` — 2 игрока; `cat` — случайные ходы; `fox` — win→block→ПОЗИЦИОННАЯ ЭВРИСТИКА (для каждой пустой клетки суммируем вес незаблокированных линий, `1 + mine*4 + theirs*3`; работает и на 3×3, и на 5×5) с 20% шансом уйти в `catMove`. Задержка «раздумья» бота 500–950мс. Экраны: `#startScreen`→`#gameScreen`→ маленький оверлей `#roundOverlay` («Ещё раз»/«Хватит») → финал `#endScreen`. Счёт сессии в `session {wins, losses, draws, games, p1, p2}` — сохраняется до нажатия «Хватит» или смены режима, звёзды в конце сессии. Правила звёзд (бот): 3+ побед → 3⭐, 2 → 2⭐, 1 → 1⭐, 0 но играл → 1⭐; 2 игрока: 3+ игр → 2⭐, 1+ → 1⭐. Ключ звёзд `krestiki`. Фиолетовая палитра (`--accent:#6366f1`), победная линия подсвечивается жёлтым (`.win`) с пульсацией.

## Деплой
- `git push origin main` → GitHub Pages → `https://vkassu.github.io/GAMES_FOR_OMMA`.
- Коммит-сообщения на английском, заканчиваются `Co-Authored-By: Claude ...`.
- Коммитить/пушить только по явной просьбе пользователя.

## Гигиена контекста (как вести разработку)
- Одна задача/игра = одна сессия. После пуша — `/clear` или новый чат.
- Просить работать с конкретным файлом → не сканировать весь проект.
- Этот файл + `CLAUDE.md` позволяют поднять контекст без чтения всех игр.
