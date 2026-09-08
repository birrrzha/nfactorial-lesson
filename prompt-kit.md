# Промпт-кит: дизайн без дизайнера

Раздатка к уроку. Всё копируется и работает как есть.

**Почему промпты на английском.** Дизайнерская терминология внутри моделей англоязычная: «visual hierarchy», «whitespace», «type scale» — это термины, вокруг которых написаны все учебники и статьи, на которых модель училась. Русский перевод тех же понятий встречается в обучающих данных заметно реже. Отсюда практическое правило: запрос на английском попадает в более плотную часть того, что модель знает. Это не магия и не гарантия — на простых задачах разницы не будет. Но на дизайне английский остаётся безопасным дефолтом, поэтому все промпты ниже даны на нём. Ваш диалог с агентом при этом может идти на русском.

---

## 1. Что не работает (для понимания)

```
Build a landing page for my product. Make something absolutely
unique — make all design decisions completely at random.
```

Даёт другой результат, чем голый промпт, но между запусками — одинаковый. Модель не умеет быть случайной, она предсказывает токены, которые звучат случайно.

---

## 2. Seed string: внешняя энтропия

Когда применять: смотрите на пустой экран, направления нет, нужны варианты.

```
Build a landing page for [PRODUCT].

Follow this procedure:

1. Generate a long random alphanumeric string using a shell script.
2. Determine a creative direction (color palette, layout,
   typography) based on that string. Look beyond the surface: find
   sub-patterns, notable numbers, anything that inspires you.
3. Use your judgment to realize that direction and execute it well.

Do not expose the string in the design. It is only a source of
inspiration for you.
```

Запускайте 4–8 раз параллельно, выбирайте одно, дальше доводите промптами 4 и 6.

**Важно:** даёт разное, не даёт хорошее. Это генератор вариантов.

---

## 3. Поиск направления через свою реакцию

Три шага. Работает, даже если вы никогда не занимались дизайном, потому что опирается на отбраковку, а не на придумывание.

### Шаг 1 — широкий список без деталей

```
I want to come up with a bold, unconventional design language for
my product. List as many ideas as you can, one or two lines each,
no detail. Go wide, not deep. At least 20 of them.
```

### Шаг 2 — своя реакция на понравившиеся

Не «сделай лучше», а конкретное описание, что нравится и что раздражает. Пример формы ответа:

```
Industrial control panel:
— I want tactility. Clicky buttons, satisfying sounds.
— My first instinct was something cartoonish and skeuomorphic, but
  that feels cheap. Not that.
— I want consistent components and small details that give it the
  look without overdoing it.
— Grey gradients would be boring, it needs texture. Maybe add color
  while keeping the control-panel feel?

Sharpen this direction to my taste.
```

Повторять, пока не устроит.

### Шаг 3 — получить промпт

```
Write a short prompt an agent can use to build a first version of
the page in this direction.
```

---

## 4. Агент-критик

Основной рабочий инструмент. Арендует вкус у более сильной модели.

```
Improve this design. To decide what to focus on, use a
[STRONG MODEL] subagent as a design critic.

On each iteration:
— Take a screenshot of the current design
— Invoke the critic in a fresh context, passing ONLY the
  screenshot: no code, no implementation details, no prior
  iterations, no prior critiques
— Ask it to identify the aesthetic the design is reaching for,
  imagine how a top-tier design studio would execute that
  aesthetic, and list the main gaps
— Ask for a score out of 10: how close the current design is to
  that studio-level bar

In the critic's prompt, instruct it to:
— Consider both large-scale structure and composition, and fine
  details
— Specifically look for patterns that read as overused, excessive
  or obviously AI-generated, and penalize them
— Give dense, specific feedback rather than vague prose
— Be bold and opinionated rather than safe and agreeable

The work is done only when the critic independently scores it
9/10 or higher. Do not include this stopping criterion in the
critic's prompt — its scoring must stay objective. Use the
identical critic prompt on every iteration.
```

### Четыре правила настройки цикла

**Критерии критика — максимально объективные.**

| Уровень | Формулировка |
|---|---|
| Плохо | `Rate whether this design is beautiful and doesn't look AI-generated` — слишком субъективно, разброс от запуска к запуску |
| Нормально | `Identify the target aesthetic, imagine a top studio's execution, score the gap` — рамка мутная, но стабильная |
| Хорошо | `Here are 5 images: 4 are professional examples and 1 is a screenshot of our product. Rank them by level of polish and taste, and explain the ranking` — конкретно и объективно |

**Давайте референсы для калибровки планки.** Скриншоты того, что нравится, или даже сгенерённый концепт-арт. Явно сказать критику, что это мудборд, а не образец для копирования:

```
These reference images are a moodboard and a quality baseline, not
something to copy. Judge our screen against the level of craft they
show, not against their specific content or layout.
```

**Ставьте критерий остановки.** Иначе цикл не сойдётся: критик всегда найдёт претензию, агент будет жечь токены. Начните с двух итераций, посмотрите на сходимость, потом разрешайте больше.

### Критик для продуктового режима

Всё выше настроено на вкус и полировку — это режим витрины. Для формы, таблицы или настроек такой критик почти бесполезен: там нечего полировать, там надо проверить, что интерфейсом можно пользоваться. Критерии для этого давно написаны — эвристики Нильсена, стандарт индустрии с 1994 года.

Модель их знает, поэтому перечислять не нужно, достаточно сослаться:

```
Act as a usability reviewer. Evaluate the attached screen against
Jakob Nielsen's 10 usability heuristics.

For each heuristic:
— State whether the screen satisfies it, partially satisfies it,
  or violates it
— If not fully satisfied, describe the specific problem and where
  on the screen it appears
— Rate severity from 0 (not a problem) to 4 (usability catastrophe)

Then list the issues sorted by severity, highest first.

Ignore aesthetics entirely. I am not asking whether this looks
good — I am asking whether it works. Do not suggest visual
polish.
```

Последний абзац важен: без него модель скатывается на «добавьте больше воздуха» вместо «нет состояния ошибки и пользователь не поймёт, что форма не отправилась».

Отдельная польза этого промпта — он ловит ровно то, чего в сгенерённых интерфейсах не бывает никогда: пустые состояния, ошибки, обратную связь после действия, возможность отменить. Пункты 15–17 чеклиста ниже — как раз оттуда.

**Разные модели на разные роли.** Критик — самая сильная доступная (больше параметров, шире распределение идей, лучше вкус). Исполнитель — дешёвая и быстрая, но не совсем мелкая: она должна уметь исполнить направление. Критик даёт меньше 10% выходных токенов.

---

## 5. Ассеты: картинки и видео

```
The design looks flat. Add personality through image generation.
Consider shaders or 3D effects combined with generated images for a
more interesting visual.

Use this [OpenAI / Gemini] key for generation: sk-...
Use it locally only. Never commit it or ship it to production.

Verify the result frame by frame in the browser.
```

### Как подключить генерацию

| Ваш агент | Что делать |
|---|---|
| Codex, Antigravity, Grok Build | Сказать явно: `use your built-in image generation`. Умеет, но сам почти не делает |
| Claude Code + подписка ChatGPT | `Use the Codex CLI for image generation. Help me install it if it's missing. Make sure it bills against the subscription, not the API key` |
| Любой другой | Отдельный API-ключ с жёстким лимитом трат |

### Хранение ключей

```
Create a gitignored file .env.agents, put this API key in it, and
note in CLAUDE.md that these keys are for your own use during
development and must never reach production.
```

### Видео: анимированная графика без вида «вставленного видео»

```
Replace the image on the page with a looping video clip.
[ANIMATION DESCRIPTION].

To get convincing refraction, first render the video over the
page's background colors so the refraction effects are baked in,
then remove the background with a video matting model.

fal.ai key: sk-...
Find suitable up-to-date models for video generation and background
removal.
```

### Видео: переходы между состояниями

Недооценённый приём. Многие видеомодели умеют интерполировать между двумя опорными кадрами — то есть собирать плавный переход между двумя состояниями экрана. Клип потом можно проматывать скроллом или жестом.

```
Build a demo page that uses a video model for interactive
transitions between several screens. Each screen shows [OBJECT] in
a different state, with vertical motion suited to scrolling:
— [STATE 1]
— [STATE 2]
— [STATE 3]

Generate the first frame with image generation. Then generate a
clip that starts from that frame and animates to the next state.
Use the last frame of that clip as the seed for the next
transition so everything is seamless. Scrub through the transitions
as the user scrolls.

fal.ai key: sk-...
Use a video model with strong physics and consistency.
```

---

## 6. Вычитание

Главный недостающий навык. ИИ добавляет и почти никогда не убирает: удалять — рискованно, а модель обучена не рисковать.

```
Go through the design and remove everything that doesn't earn its
place. Specifically:
— Simplify the layout down to [TARGET STRUCTURE]
— Remove gradients, glows and unnecessary containers
— Remove labels that repeat what the visual already makes obvious
— Replace custom buttons and inputs with native
  [PLATFORM / LIBRARY] components
— Reduce text size and weight where it shouts without reason

Aim for a genuinely minimal aesthetic. Do not add anything.
```

Последняя строка обязательна: без неё агент удалит три элемента и добавит пять.

---

## 7. Чеклист: признаки сгенерённого дизайна

Прогоняйте свой экран по списку перед тем, как показывать кому-то.

**Цвет и эффекты**
1. Фиолетово-синий градиент в фоне или на кнопке
2. Градиентный текст в заголовке
3. Стеклянные карточки с блюром без причины
4. Свечения и мягкие цветные пятна в фоне
5. Тени по умолчанию на всём подряд

**Структура**
6. Hero: текст слева, картинка справа, две кнопки
7. Ровно три карточки с фичами в ряд
8. Эмодзи вместо иконок
9. Одинаковые скругления на всех элементах без иерархии
10. Всё выровнено по центру

**Текст**
11. Заголовки уровня «Elevate your workflow» и «Supercharge your productivity»
12. Выдуманная статистика: «10,000+ users», «99.9% uptime»
13. Подпись под каждым элементом, объясняющая очевидное
14. FAQ-аккордеон в конце, которого никто не просил

**Чего не хватает** (это выдаёт сильнее всего)
15. Нет пустых состояний
16. Нет состояний ошибки и загрузки
17. Всё на коротких идеальных строках — не проверено на длинных именах и переполнении

---

## 8. Блок для CLAUDE.md / AGENTS.md

Кладётся один раз в проект, действует во всех сессиях.

````markdown
## Design rules

Never use without an explicit request:
- Purple or blue-violet gradients
- Gradient text
- Glassmorphic cards with backdrop-blur
- Glows and soft colored blobs in backgrounds
- Emoji used as icons
- Invented statistics or social proof
- Headlines in the "Elevate / Supercharge / Unlock your" register

Always:
- Spacing on a 4/8 scale only
- Radii, shadows and spacing from tokens, never arbitrary values
- At most two font weights per screen
- Use components from [LIBRARY] instead of hand-rolling them
- For every screen, handle: empty state, loading, error, long text
  overflow

When editing a design, first consider what to remove and only then
what to add. If an edit increases the number of elements on screen,
explain why.
````

---

## 9. Словарь: семь слов, чтобы описать, что не так

Без них вы пишете агенту «сделай красиво» и получаете то же самое. Третья колонка — термин, который стоит использовать в промпте.

| Слово | Что означает | Как звучит претензия агенту |
|---|---|---|
| Иерархия | Что читается первым, вторым, третьим | `There's no visual hierarchy — everything is the same size` |
| Контраст | Насколько сильно главное отличается от второстепенного | `Weak contrast between the heading and the supporting text` |
| Воздух | Сколько пустого места | `Too dense. Add more whitespace between blocks` |
| Ритм | Повторяются ли расстояния | `Spacing is arbitrary. Put everything on an 8px scale` |
| Выравнивание | Сколько вертикальных линий на экране | `Too many alignment axes. Reduce to two` |
| Вес | Насколько жирный шрифт | `Bold everywhere. Keep one emphasis per screen` |
| Консистентность | Одинаковы ли радиусы, тени, отступы | `Corner radii vary with no reason. Unify them` |

---

## 10. Ссылки

**Техники из урока**
- String Seed of Thought, Sakana AI — https://pub.sakana.ai/ssot/
- Anshu Chimala, «How to turn your AI into a world-class designer» — https://www.lennysnewsletter.com/p/how-to-turn-your-ai-into-a-world

**UX-критика: эвристики Нильсена**
- 10 эвристик юзабилити — https://www.nngroup.com/articles/ten-usability-heuristics/ — основа для критика в продуктовом режиме
- Как проводить эвристическую оценку — https://www.nngroup.com/articles/how-to-conduct-a-heuristic-evaluation/
- Бесплатный воркбук для ручной оценки — https://media.nngroup.com/media/articles/attachments/Heuristic_Evaluation_Workbook_1_Fillable.pdf
- Все материалы NN/g по теме — https://www.nngroup.com/topic/heuristic-evaluation/

**Инструменты и референсы**
- fal.ai — агрегатор видеомоделей под один ключ
- Mobbin, Godly, Land-book — референсы для калибровки критика
