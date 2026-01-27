# MCScript Compiler

Компилятор высокоуровневого языка для создания Minecraft Datapacks. Преобразует читаемый синтаксис `.mcs` в стандартные `.mcfunction` файлы.

## 🚀 Возможности

- **Упрощённый синтаксис** для работы со scoreboards
- **Условные операторы** с поддержкой вложенности
- **Автоматическая генерация** execute команд
- **Режим наблюдения** для автоматической перекомпиляции
- **Поддержка всех условий** Minecraft (block, entity, biome и др.)

## 🎯 Использование

### Базовая компиляция

```bash
java -jar mcs.jar ./datapack/name/data/game
```

### Режим наблюдения (автоперекомпиляция)

```bash
java -jar mcs.jar ./datapack/name/data/game --watch
```

## 📝 Синтаксис MCScript

### 1. Инициализация скорбордов

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

### 2. Операции со скорбордами

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

#### Операции между игроками

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
(@s).a >< (@p).b    # Обмен значениями (swap)
(@s).score < (@p).score   # Минимум
(@s).score > (@p).score   # Максимум
```

### 3. Условные операторы

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
```

### 4. Расширенные условия

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

#### Сравнение скорбордов

```mcscript
if score(@s, money, >, @p, wealth) {
    /say Вы богаче игрока
}

if score(@s, level, >=, @a[limit=1], minLevel) {
    /say Уровень достаточен
}
```

### 5. Контекстные блоки (as/at)

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

### 6. Вызов других функций

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

### 7. Прямые команды

любой текст после `/` копируется как есть:

```mcscript
/say Привет мир!
/tp @s 100 64 200
/give @s diamond 64
```

## 📂 Структура проекта

```
datapack/
└── your_datapack/
    └── data/
        └── game/              # Имя вашего namespace
            ├── scripts/       # Исходники .mcs
            │   ├── main.mcs
            │   ├── shop.mcs
            │   └── events.mcs
            └── function/      # Скомпилированные .mcfunction (генерируется)
                ├── main.mcfunction
                ├── shop.mcfunction
                └── events.mcfunction
```

## 💡 Полный пример

**Файл: `scripts/game.mcs`**

```mcscript
# Инициализация
init money
init health health
init kills playerKillCount

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
```

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

## ⚙️ Операторы сравнения

| MCScript | Minecraft | Описание |
|----------|-----------|----------|
| `>`      | `101..`   | Больше |
| `>=`     | `100..`   | Больше или равно |
| `<`      | `..99`    | Меньше |
| `<=`     | `..100`   | Меньше или равно |
| `==`     | `100`     | Равно |
| `!=`     | `!100`    | Не равно |

## 📋 Требования

- **Java** 21 или выше
- **Minecraft** 1.21+ (для datapacks)

## 🐛 Отладка

Компилятор выводит информацию о процессе:

```
Компилирую main.mcs...
DEBUG: Папка датапака: game
  ✓ game:main.mcfunction (15 команд)

Компилирую shop.mcs...
DEBUG: Папка датапака: game
  ✓ game:shop.mcfunction (8 команд)

Готово!
```

## 📄 Лицензия

MIT License

## 🤝 Вклад

Если вы нашли баг или хотите предложить улучшение - создавайте Issue или Pull Request!

---

**Автор:** lexeu
**Версия:** 0.0.1 bet
