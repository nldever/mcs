# MCScript Compiler

Компилятор высокоуровневого языка для создания Minecraft Datapacks. Преобразует читаемый синтаксис `.mcs` в стандартные `.mcfunction` файлы.

## 🚀 Возможности

- **Директивы времени** — `~autorun`, `~timer`, `~interval`, `~delay`, `~repeat`, `~once`
- **Упрощённый синтаксис** для работы со scoreboards
- **Управляющие конструкции** — `if/else`, `while`, `for`, `foreach`, `switch`
- **Контекстные блоки** — `as`, `at`, `positioned`, `rotated`, `facing`, `in`
- **Система событий** — обработчики `player_join`, `item_use`, `block_break`
- **Макросы и функции** — переиспользуемые блоки кода
- **Ресурсы датапака** — предикаты, loot tables, теги, рецепты, достижения
- **Автоматическая генерация** load.mcfunction и tick.mcfunction
- **Режим наблюдения** для автоматической перекомпиляции
- **Детальные ошибки** с указанием строки и столбца

## 🎯 Использование

### Интерактивный режим (рекомендуется)

```bash
java -jar mcs.jar
```

Выберите:
- **[1]** Создать новый датапак
- **[2]** Выбрать существующий датапак
- **[3]** Компилировать по пути

### Базовая компиляция

```bash
java -jar mcs.jar ./datapack/name/data/game
```

### Режим наблюдения (автоперекомпиляция)

```bash
java -jar mcs.jar ./datapack/name/data/game --watch
```

## 📝 Синтаксис MCScript

### Рекомендую к установке плагин Minecraft Custom Scripts в VSCode. [Установить](https://marketplace.visualstudio.com/items?itemName=lexeu.mcs)

### 1. Директивы времени

#### Автозапуск функции

```mcscript
~autorun(20)  # Запустить через 1 секунду (20 тиков)
```

Запускает функцию через N тиков после загрузки датапака.

#### Таймер с повторением

```mcscript
~timer(save_timer, 1200, repeat) {
    /tellraw @a ["",{"text":"💾 ","color":"aqua"},
                    {"text":"Автосохранение...","color":"gray"}]
}
```

Создаёт обратный отсчёт. С `repeat` — бесконечный цикл.

#### Интервал

```mcscript
~interval(broadcast, 600) {
    /tellraw @a "📢 Не забывайте про правила!"
}
```

Повторяющаяся функция с постоянным интервалом.

#### Задержка выполнения

```mcscript
~delay(100) {
    /say Прошло 5 секунд!
}
```

#### Повторение N раз

```mcscript
~repeat(10, 20) {
    /particle flame ~ ~1 ~ 0.1 0.1 0.1 0 5
}
```

#### Однократное выполнение

```mcscript
~once() {
    /tellraw @a {"text":"Игра началась!","color":"gold"}
}
```

Выполняется только один раз за всю игру.

---

### 2. Инициализация скорбордов

```mcscript
# Синтаксис: init <название> [тип]
init money
init health health
init kills playerKillCount
```

**Компилируется в:**
```mcfunction
scoreboard objectives add money dummy
scoreboard objectives add health health
scoreboard objectives add kills playerKillCount
```

Автоматически добавляется в `load.mcfunction`.

---

### 3. Переменные и константы

```mcscript
var playerName = "Steve"
var health = 20
var spawn = 0 64 0

const MAX_HEALTH = 20
const SPAWN_POINT = 0 64 0
```

**Типы данных:**
- `STRING` — `"текст"`
- `NUMBER` — `42`, `-10`
- `SELECTOR` — `@a[distance=..10]`
- `COORDINATES` — `0 64 0`, `~5 ~ ~-3`
- `BOOLEAN` — `true`, `false`
- `ARRAY` — `[1, 2, 3]`
- `OBJECT` — `{key: value}`

---

### 4. Операции со скорбордами

#### Присваивание значений

```mcscript
(@s).money = 100
(@a).health = 20
```

**Компилируется в:**
```mcfunction
scoreboard players set @s money 100
scoreboard players set @a health 20
```

#### Арифметические операции

```mcscript
(@s).money += 50          # Добавить
(@s).money -= 30          # Вычесть
(@s).score *= 2           # Умножить
(@s).score /= 2           # Разделить
(@s).score %= 10          # Остаток от деления
```

**Компилируется в:**
```mcfunction
scoreboard players add @s money 50
scoreboard players remove @s money 30
scoreboard players operation @s score *= #2 score
scoreboard players operation @s score /= #2 score
scoreboard players operation @s score %= #10 score
```

#### Операции между сущностями

```mcscript
(@s).money = (@p).wealth              # Копирование
(@s).score += (@a[limit=1]).bonus    # Сложение
(@s).total *= (@e[type=marker]).multiplier
```

**Компилируется в:**
```mcfunction
scoreboard players operation @s money = @p wealth
scoreboard players operation @s score += @a[limit=1] bonus
scoreboard players operation @s total *= @e[type=marker] multiplier
```

#### Специальные операции

```mcscript
(@s).a >< (@p).b          # Обмен значениями (swap)
(@s).score < (@p).score   # Минимум
(@s).score > (@p).score   # Максимум
```

---

### 5. Функции

```mcscript
function give_starter_kit() {
    give (@s) minecraft:wooden_sword 1
    give (@s) minecraft:bread 16
    (@s).money = 100
}

# Вызов функции
open give_starter_kit
```

**Создаёт:** `give_starter_kit.mcfunction`

---

### 6. Макросы

```mcscript
macro heal_player(player, amount) {
    ($player).health += $amount
    /effect give $player regeneration 5 1
}

# Использование
call heal_player(@s, 10)
call heal_player(@a[distance=..5], 5)
```

**Раскрывается при компиляции:**
```mcscript
(@s).health += 10
/effect give @s regeneration 5 1
```

---

### 7. События

```mcscript
event player_join {
    /tellraw @a ["",{"text":"🎉 ","color":"gold"},
                    {"text":"Новый игрок присоединился!","color":"yellow"}]
    (@s).money = 1000
}

event item_use(item=diamond_sword) {
    /playsound entity.player.attack.strong player @s
    (@s).combo += 1
}
```

**Генерирует:**
- Файл в папке `events/`
- Для `player_join` создаёт advancement
- Для других событий добавляет в `tick.mcfunction`

---

### 8. Условные операторы

#### Базовые условия (сравнение с числом)

```mcscript
if (@s).money > 100 {
    /say Богатый игрок!
}

if (@s).health <= 5 {
    /effect give @s regeneration 10 2
}

if (@s).score == 0 {
    /tellraw @s "Начните игру!"
}
```

#### Сравнение scoreboards между сущностями

```mcscript
# Сравнение уровня игрока с максимальным уровнем другого игрока
if (@s).level > (@p).maxlevel {
    /say Ваш уровень превышает максимум!
}

# Проверка, больше ли денег у игрока, чем у ближайшего
if (@s).money >= (@p[distance=0.1..]).money {
    /say Вы богаче соседа!
}

# Сравнение очков с маркером
if (@s).score < (@e[type=marker,limit=1]).threshold {
    /tellraw @s "Недостаточно очков"
}

# Проверка равенства
if (@s).team_id == (@p[tag=leader]).team_id {
    /say Вы в одной команде с лидером
}

# Проверка неравенства
if (@s).points != (@a[limit=1,sort=random]).points {
    /say У вас разное количество очков
}
```

**Компилируется в:**
```mcfunction
# if (@s).level > (@p).maxlevel
execute if score @s level > @p maxlevel run say Ваш уровень превышает максимум!

# if (@s).money >= (@p[distance=0.1..]).money
execute if score @s money >= @p[distance=0.1..] money run say Вы богаче соседа!

# if (@s).score < (@e[type=marker,limit=1]).threshold
execute if score @s score < @e[type=marker,limit=1] threshold run tellraw @s "Недостаточно очков"

# if (@s).team_id == (@p[tag=leader]).team_id
execute if score @s team_id = @p[tag=leader] team_id run say Вы в одной команде с лидером

# if (@s).points != (@a[limit=1,sort=random]).points
execute unless score @s points = @a[limit=1,sort=random] points run say У вас разное количество очков
```

#### Отрицание условий

```mcscript
if not (@s).ready == 1 {
    /tellraw @s "Вы не готовы"
}

if unless (@s).hasKey > 0 {
    /tellraw @s "Нужен ключ"
}
```

#### Else ветвления

```mcscript
if (@s).money >= 100 {
    (@s).money -= 100
    /give @s diamond_sword
} else {
    /tellraw @s ["",{"text":"Недостаточно денег!","color":"red"}]
}

# Else if
if (@s).score >= 100 {
    /tellraw @s "Отличный результат!"
} else if (@s).score >= 50 {
    /tellraw @s "Хороший результат"
} else {
    /tellraw @s "Попробуйте ещё раз"
}
```

---

### 9. Расширенные условия

#### Проверка блока

```mcscript
if block(0, 64, 0, diamond_block) {
    /say Найден алмазный блок!
}

if block(~, ~-1, ~, emerald_block) {
    /say Стоишь на изумруде
}
```

#### Проверка области блоков

```mcscript
if blocks(0, 0, 0, 10, 10, 10, 20, 20, 20, all) {
    /say Области идентичны
}
```

#### Проверка сущностей

```mcscript
if entity(@e[type=zombie,distance=..10]) {
    /say Зомби рядом!
}

if not entity(@e[type=creeper]) {
    /say Нет криперов
}
```

#### Проверка направления взгляда

```mcscript
if facing(entity, @p, eyes) {
    /say Смотрит на игрока
}

if facing(100, 64, 200) {
    /say Смотрит на координаты
}
```

#### Проверка биома

```mcscript
if biome(minecraft:plains) {
    /say Вы на равнине
}
```

#### Проверка измерения

```mcscript
if dimension(minecraft:the_nether) {
    /say Добро пожаловать в Ад!
}
```

#### Проверка загруженности чанка

```mcscript
if loaded(100, 64, 200) {
    /say Чанк загружен
}
```

#### Проверка предиката

```mcscript
if predicate(namespace:my_predicate) {
    /say Предикат выполнен
}
```

#### Сравнение скорбордов (альтернативный синтаксис)

```mcscript
if score(@s, money, >, @p, wealth) {
    /say Вы богаче игрока
}

if score(@s, level, >=, @a[limit=1], minLevel) {
    /say Уровень достаточен
}
```

---

### 10. Циклы

#### While

```mcscript
while (@s).counter > 0 {
    /particle flame ~ ~1 ~ 0.1 0.1 0.1 0 1
    (@s).counter -= 1
}
```

Создаёт рекурсивную функцию.

#### For

```mcscript
for (@s).i = 0, 10, 1) {
    /tellraw @s ["",{"text":"Итерация: ","color":"gray"},
                    {"score":{"name":"@s","objective":"i"},"color":"yellow"}]
}
```

**Параметры:** `(селектор.objective = start, end, step)`

#### Foreach

```mcscript
foreach (@a[team=red]) {
    /effect give @s strength 10 1
    (@s).team_bonus = 5
}
```

**Компилируется в:**
```mcfunction
execute as @a[team=red] run effect give @s strength 10 1
execute as @a[team=red] run scoreboard players set @s team_bonus 5
```

---

### 11. Switch

```mcscript
init gamemode

switch (@s).gamemode {
    case 0: {
        /tellraw @s "Режим: Выживание"
    }
    case 1: {
        /tellraw @s "Режим: Творчество"
    }
    case 2: {
        /tellraw @s "Режим: Приключение"
    }
    default: {
        /tellraw @s "Неизвестный режим"
    }
}
```

---

### 12. Контекстные блоки

#### As — выполнение от лица сущности

```mcscript
as (@a) {
    /say Это выполнит каждый игрок
}

as (@a) at (@s) {
    /particle heart ~ ~2 ~
    if (@s).money > 100 {
        /playsound entity.player.levelup player @s
    }
}
```

**Компилируется в:**
```mcfunction
execute as @a run say Это выполнит каждый игрок
execute as @a at @s run particle heart ~ ~2 ~
execute as @a at @s if score @s money matches 101.. run playsound entity.player.levelup player @s
```

#### At — выполнение в позиции сущности

```mcscript
at (@e[type=armor_stand,tag=particle_point]) {
    /particle flame ~ ~ ~ 0.5 0.5 0.5 0 20
}
```

#### Positioned — выполнение в координатах

```mcscript
positioned 0 64 0 {
    /summon minecraft:villager ~ ~ ~ {CustomName:'{"text":"Спавн торговец"}'}
}
```

#### Rotated — выполнение с поворотом

```mcscript
rotated 0 0 {
    /tp @s ^ ^ ^5
}
```

#### Facing — выполнение лицом к точке/сущности

```mcscript
# Лицом к координатам
facing 0 64 0 {
    /tp @s ^ ^ ^1
}

# Лицом к сущности
facing entity (@p) eyes {
    /tp @s ^ ^ ^0.5
}
```

#### In — выполнение в измерении

```mcscript
in minecraft:the_nether {
    /say Эта команда выполняется в аду
}
```

---

### 13. Диалоги

```mcscript
dialog {
    npc = "Старый маг",
    text = "Приветствую тебя, путник.",
    text = "Я могу научить тебя древней магии."
}
```

**Генерирует:**
```mcfunction
tellraw @a [{"text":"[Старый маг] ","color":"gold","bold":true},{"text":"Приветствую тебя, путник.","color":"yellow"}]
tellraw @a [{"text":"[Старый маг] ","color":"gold","bold":true},{"text":"Я могу научить тебя древней магии.","color":"yellow"}]
```

---

### 14. Конфигурация

```mcscript
config {
    game_name = "SkyWars",
    max_players = 12,
    difficulty = "hard"
}
```

**Генерирует:**
```mcfunction
data merge storage my_datapack:config {"game_name":"SkyWars","max_players":12,"difficulty":"hard"}
```

---

### 15. Ресурсы датапака

#### Предикаты

```mcscript
predicate {
    name = "is_sneaking",
    condition = "minecraft:entity_properties"
}
```

**Создаёт:** `data/namespace/predicate/is_sneaking.json`

#### Loot Tables

```mcscript
loot_table {
    name = "zombie_rare_drops",
    type = "minecraft:entity",
    pool = {
        item = "minecraft:diamond",
        rolls = 1
    }
}
```

#### Теги

```mcscript
tag {
    name = "valuable_blocks",
    type = "block",
    values = ["minecraft:diamond_ore", "minecraft:emerald_ore"]
}
```

#### Рецепты

```mcscript
recipe {
    name = "magic_sword",
    type = "minecraft:crafting_shaped",
    pattern = [" D ", " D ", " E "],
    key = {
        "D" = "minecraft:diamond",
        "E" = "minecraft:emerald"
    },
    result = {
        item = "minecraft:diamond_sword",
        count = 1
    }
}
```

#### Достижения

```mcscript
advancement {
    name = "first_diamond",
    title = "Первый алмаз!",
    description = "Добудьте свой первый алмаз",
    icon = "minecraft:diamond",
    trigger = "minecraft:inventory_changed"
}
```

### 16. Вызов других функций

```mcscript
open my_function

# С условием
if (@s).trigger == 1 {
    open another_function
}
```

**Компилируется в:**
```mcfunction
function game:my_function
execute if score @s trigger matches 1 run function game:another_function
```

---

### 17. Прямые команды

Любой текст после `/` копируется как есть:

```mcscript
/say Привет мир!
/tp @s 100 64 200
/give @s diamond 64
```

---

## 📂 Структура проекта

```
datapack/
└── your_datapack/
    ├── pack.mcmeta
    └── data/
        └── game/              # Имя вашего namespace
            ├── scripts/       # Исходники .mcs
            │   ├── main.mcs
            │   ├── shop.mcs
            │   └── events.mcs
            │
            ├── function/      # Скомпилированные .mcfunction
            │   ├── load.mcfunction     # Автогенерация
            │   ├── tick.mcfunction     # Автогенерация
            │   ├── main.mcfunction
            │   ├── shop.mcfunction
            │   └── events/
            │       └── player_join.mcfunction
            │
            ├── advancement/   # Достижения
            ├── loot_table/    # Таблицы добычи
            ├── predicate/     # Предикаты
            └── tags/          # Теги
                └── function/
                    ├── load.json
                    └── tick.json
```

---

## 💡 Полный пример

**Файл: `scripts/game.mcs`**

```mcscript
~autorun(20)

# Инициализация
init money
init health health
init kills playerKillCount

# Приветствие нового игрока
event player_join {
    (@s).money = 1000
    /tellraw @s ["",
        {"text":"💰 ","color":"gold"},
        {"text":"Вы получили ","color":"gray"},
        {"text":"1000","color":"yellow"},
        {"text":"$ стартового капитала!","color":"gray"}
    ]
}

as (@a) {
    if block(0,0,0, air) {
        /say Generator active
    }
    if not entity(@e[type=zombie,distance=..10]) {
        /say Safe zone
    }
}

# Проверка здоровья игроков
as (@a) {
    if (@s).health <= 5 {
        /effect give @s regeneration 10 2
        /title @s actionbar ["",{"text":"⚠ Низкое здоровье!","color":"red"}]
    }
}

# Магазин
as (@a[scores={clickShop=1..}]) {
    (@s).clickShop = 0
    
    if block(~, ~-1, ~, emerald_block) {
        if (@s).money >= 100 {
            (@s).money -= 100
            /give @s diamond_sword
            /playsound entity.player.levelup player @s
        } else {
            /tellraw @s ["",{"text":"Недостаточно денег!","color":"red"}]
        }
    }
}

# Передача денег
as (@a[scores={transfer=1..}]) {
    (@s).transfer = 0
    
    if (@s).money >= 10 {
        if entity(@p[distance=0.1..5]) {
            (@s).money -= 10
            (@p[distance=0.1..5]).money += 10
            /tellraw @s ["",{"text":"✓ Переведено","color":"green"}]
        }
    }
}

# Детектор мобов
as (@a) {
    if entity(@e[type=zombie,distance=..10]) {
        if entity(@e[type=skeleton,distance=..10]) {
            (@s).danger = 2
        } else {
            (@s).danger = 1
        }
    } else {
        (@s).danger = 0
    }
}

# Соревнование по очкам
as (@a) {
    # Проверка, является ли игрок лидером
    if (@s).score > (@a[limit=1,sort=random]).score {
        /tag @s add leader
    }
    
    # Проверка достижения целевого значения
    if (@s).score >= (@e[type=marker,tag=target]).goal {
        /title @s title {"text":"Цель достигнута!","color":"gold"}
        /playsound ui.toast.challenge_complete player @s
    }
}

# Ежедневная награда
~interval(daily_reward, 24000) {
    as (@a) {
        (@s).money += 100
        /title @s actionbar ["",
            {"text":"💵 Ежедневная награда: ","color":"gold"},
            {"text":"+100$","color":"yellow"}
        ]
    }
}
```

---

## 🔧 Комментарии

MCScript поддерживает три типа комментариев:

```mcscript
# Однострочный комментарий (стиль Shell)

// Однострочный комментарий (стиль C++)

/* 
   Многострочный
   комментарий
*/
```
---

## ⚙️ Операторы сравнения

| MCScript | Minecraft | Описание |
|----------|-----------|----------|
| `>`      | `101..`   | Больше |
| `>=`     | `100..`   | Больше или равно |
| `<`      | `..99`    | Меньше |
| `<=`     | `..100`   | Меньше или равно |
| `==`     | `100` или `=` | Равно |
| `!=`     | `unless score ... =` | Не равно |

---

## 🐛 Отладка

### Формат ошибок компилятора

```
=========================================================
                  ОШИБКА КОМПИЛЯЦИИ
=========================================================

Файл: scripts/main.mcs
Позиция: строка 15, символ 9

Ошибка: Ожидалось "}", но найдено "if (@s).health"
Найдено: 'i'

Фрагмент кода:

  14 | /effect give @s regeneration 10 2
>  15 |     if (@s).health <= 10 {
              ^
              └─ Здесь ошибка
  16 |         /say Низкое здоровье!

Подсказки:
   • Возможно, пропущена закрывающая скобка '}'
   • Проверьте, что все блоки кода правильно закрыты
```

### Команды в игре

```mcfunction
/reload                         # Перезагрузить датапак
/datapack list                  # Список датапаков
/function namespace:main        # Запустить функцию
/scoreboard objectives list     # Список objectives
/scoreboard players list        # Список игроков с очками
```

---

## ⚠️ Известные ограничения

### 1. Интерполяция переменных в JSON

❌ **Не работает:**
```mcscript
var player = "Steve"
/tellraw @a {"text":"Привет, $player!"}
```

✅ **Используйте:**
```mcscript
/tellraw @a ["",{"text":"Привет, "},{"selector":"@s"}]
```

### 2. Switch требует objective

❌ **Ошибка:**
```mcscript
switch (@s).gamemode {  # objective не создан
```

✅ **Правильно:**
```mcscript
init gamemode
switch (@s).gamemode {
```

---

## 📋 Требования

- **Java** 8 или выше
- **Minecraft** 1.21.11 (pack_format 48)

---

## 📊 Совместимость

| Версия Minecraft | Pack Format | Поддержка |
|------------------|-------------|-----------|
| 1.21.11          | 48          | ✅ Полная |
| 1.21.x           | 48          | ✅ Полная |
| 1.20.5-1.20.6    | 41          | ⚠️ Изменить pack_format |
| 1.20.3-1.20.4    | 26          | ⚠️ Изменить pack_format |

---

## 📄 Лицензия

MIT License

---

## 🤝 Вклад

Если вы нашли баг или хотите предложить улучшение — создавайте Issue или Pull Request!

---

**Автор:** lexeu  
**Версия:** 0.0.1.260130b
