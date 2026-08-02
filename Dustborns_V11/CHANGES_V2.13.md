# Dustborns v2.10 — оповещения через штатное уведомление о завершении квеста

## Проблема
`Action_Message` продолжал показывать «missing GuiElement» (текст награды выдавался,
но само окно сообщения не находило GuiElement ни как ExtendedGuiElement, ни иначе).
Документации/примеров использования `Action_Message` в ES2 нет — это неподдерживаемый путь.

## Решение
Убран `Action_Message` из всех 26 квестов-слушателей. Вместо него каждый слушатель
получил полноценный **QuestGuiElement** (имя = имя квеста, как у главных квестов):

- `<Title>` — «Каталог Миров: <Мир>» (короткий заголовок);
- `<Description>` — полный текст (лор + награда);
- `<Objective Name="Listener_<Тип>">` — совпадает с целью квеста;
- `<OutcomeCompleted>` — строка награды.

Теперь при первом полном истощении мира каждого типа игра показывает **штатное
уведомление о завершении квеста** (тот же механизм, что у ванильных «Деяний»/Deeds):
название квеста + текст награды.

## Файлы
- `Quests/QuestDefinitions[Dustborns].xml` — удалены 26 блоков `Action_Message`.
- `Gui/GuiElements[Dustborns].xml` — удалены 26 `ExtendedGuiElement …_Message`,
  добавлены 26 `QuestGuiElement` с именами квестов.
- `Localization/english/ES2_Localization_Locales.xml` — добавлены 26 ключей
  `%DustbornsCatalogListener_<Тип>_Outcome` (тексты наград).
- `Dustborns.xml` — версия 2.10, ReleaseNotes.

## Как проверить (важно!)
1. **Полностью удали старую папку мода** из `My Documents\Endless Space 2\Community`,
   скопируй новую `Dustborns_V11` (проверь, что в списке модов в игре версия 2.10).
2. Уведомление срабатывает **один раз за партию** на каждый тип мира. Типы, которые
   ты уже истощил на текущем сохранении (Ash, Arid...), повторно НЕ сработают —
   проверяй на **новой игре** или на ещё не истощённом типе мира.
3. Истощи мир нового типа → должно появиться уведомление «Каталог Миров: <Мир>»
   с текстом награды.
4. Если уведомления не будет вовсе (Hidden-квесты не шлют их в твоей версии) — скажи,
   я уберу тег `Hidden` у слушателей (они появятся в журнале квестов и будут завершаться
   с уведомлением) или вернусь к `Action_Message` с именами квестов.

---
# Dustborns v2.09 — фикс оповещений «Каталога Миров» (missing GuiElement)

## Проблема
После истощения мира вместо текста выводилось `%DustbornsCatalogListener_Ash_Message (missing GuiElement)`.

## Причина
`Action_Message` в движке Amplitude резолвит атрибут `Message` как **имя GuiElement**,
а не как ключ локализации. Мы передавали `Message="%DustbornsCatalogListener_Ash_Message"`
(ключ локализации с `%`) — движок искал GuiElement с таким именем, не находил и показывал
сырую строку + дебаг-текст «missing GuiElement».

## Исправление (файлы)
- `Quests/QuestDefinitions[Dustborns].xml` — во всех 26 `Action_Message` атрибут
  `Message` теперь содержит **имя без `%`**: `DustbornsCatalogListener_Ash_Message`.
- `Gui/GuiElements[Dustborns].xml` — добавлены 26 `ExtendedGuiElement` с этими именами:
  - `<Title>%DustbornsCatalogListener_<Тип>_MessageTitle</Title>` — короткий заголовок
    («Каталог Миров: Пепельный мир»);
  - `<Description>%DustbornsCatalogListener_<Тип>_Message</Description>` — полный текст
    (лор + награда, как и раньше).
- `Localization/english/ES2_Localization_Locales.xml` — добавлены 26 ключей заголовков
  `%DustbornsCatalogListener_<Тип>_MessageTitle`.
- `Dustborns.xml` — версия 2.09, ReleaseNotes.

## Проверка
1. Истощить мир любого типа → должно появиться окно/уведомление с заголовком
   «Каталог Миров: …» и полным текстом (лор + награда).
2. Если текст показывается, но без заголовка (или наоборот) — скажи, поменяю местами
   Title/Description (это правка в одном месте GuiElements).
3. Если снова появится «missing GuiElement» — значит, в твоей версии движок ожидает
   `%`-имя; тогда вернём `%` в Message и переименуем GuiElement с `%` в начале.


## Суть изменений

Раньше: за каждый новый истощённый тип планеты — одинаковые +2/+4 к сбору Пыли
(общий счётчик `DustbornCatalogPlanets`).

Теперь: **у каждого типа мира заранее расписанный эффект** (без выбора), награда
выдаётся при ПЕРВОМ полном истощении планеты этого типа, всего **26 типов**:

| Группа | Типы | Эффект |
|---|---|---|
| Богатые (4) | Atoll, Forest, Ocean, Terran | +2/+4 к сбору Пыли с неистощённых миров |
| Умеренные (11) | Boreal, Mediterranean, Monsoon, Jungle, Savannah, Steppes, Tundra, Arctic, Snow, Arid, Desert | +1/+2 к сбору Пыли + разовый бафф **+10% к основному ресурсу мира на 10 ходов** |
| Ледяной | Ice | каждый Пылерождённый истощает миры на 0.05 в ход медленнее |
| Пепельный | Ash | Безвольные +3 производства на истощённых мирах |
| Лавовый | Lava | Безвольные +1 производства |
| Пустошь | Barren | пока мир истощается, каждый Пылерождённый даёт +4 науки в ход |
| Ядовитый | Toxic | Пылерождённые +1 Пыли |
| Газовые (6) | Gas Burning / Hot / Warm / Temperate / Cold / Frozen | +1 промы. Пылерожд. / +1 промы. Безвольн. / +1% пыли империи / +1 пыли Пылерожд. / +1 науки Пылерожд. / +1 науки Безвольн. |

Баффы умеренных миров (10 ходов):
- Наука (+10%): Boreal, Tundra, Arctic, Snow
- Промышленность (+10%): Mediterranean, Jungle, Arid, Desert
- Еда (+10%): Monsoon
- Пыль (+10%): Savannah, Steppes

## Файлы

### Изменённые
- `Simulation/SimulationDescriptors[Dustborns].xml` — из трейта Primary удалены
  счётчик `DustbornCatalogPlanets` и его бинарники (+2/+4 × счётчик); добавлены
  свойства `DustbornPatientWindsReduction` и `DustbornIceReduction` + два бинарника
  вычета истощения (−ставка × число Пылерождённых на планету).
- `Simulation/SimulationDescriptors[DustbornsQuests].xml` — удалён
  `DustbornCatalogPlanetEntry`; «Скупая Мудрость» (3-C) больше не растёт от счётчика
  (Стерильные = фикс-база 2); добавлены 17 дескрипторов-наград каталога
  (`DustbornCatalogRichDust`, `DustbornCatalogModerateDust`, `DustbornCatalogIce/Ash/Lava/Barren/Toxic`,
  `DustbornCatalogGasBurning/Hot/Warm/Temperate/Cold/Frozen`) и 4 дескриптора баффов
  (`DustbornBuffScience10/Industry10/Food10/Dust10`); «Терпеливые Ветра» переписаны —
  техника теперь ставит свойство `DustbornPatientWindsReduction=0.1` (в V10 путь
  модификатора вёл на сущность счётчика популяции, а не на планету — эффект бы не работал).
- `Quests/QuestDefinitions[Dustborns].xml` — секция слушателей переписана: 26 квестов
  (было 16). Умеренные дополнительно вызывают `Action_ApplyQuestEffect`.
- `Dustborns.xml` — новый плагин `QuestEffectDefinition` (с ExtraType
  `QuestEffectAddDescriptorsDefinition`), версия 2.08, ReleaseNotes.
- `Localization/english/ES2_Localization_Locales.xml` — 26 новых сообщений-оповещений
  (лор + описание награды), названия баффов; удалены ключи `%DustbornCatalogPlanetEntry*`.
- `Gui/GuiElements[Dustborns].xml` — удалён `ExtendedGuiElement DustbornCatalogPlanetEntry`.
- `DESIGN_Quests_and_Tech.md` — §0.1 переписан под новую систему.

### Новые
- `Quests/Effects/QuestEffectDefinitions[Dustborns].xml` — 11 временных эффектов
  (Duration=10) для умеренных миров.

## Важно: имена типов планет

Слушатели переведены на имена из вики-списка ES2 (26 типов, включая газовые гиганты):
`PlanetTypeAtoll/Forest/Ocean/Terran/Boreal/Mediterranean/Monsoon/Jungle/Savannah/Steppes/
Tundra/Arctic/Snow/Ice/Arid/Desert/Ash/Lava/Barren/Toxic/GasBurning/GasHot/GasWarm/
GasTemperate/GasCold/GasFrozen`.

Старые `PlanetTypeSteppe`, `PlanetTypeFrozen`, `PlanetTypeAsteroid` удалены —
в игре (по достижению «Home Is Where the Heart Is») таких типов нет: ледяной мир
называется Ice, снежный — Snow, степь — Steppes, а Frozen — это газовый гигант.

**Проверь в своей игре:** `Public\Simulation\SimulationDescriptors[Planet].xml` →
поиск `PlanetType`. Если какой-то слушатель не срабатывает (нет оповещения после
истощения мира этого типа) — пришли точное имя, поправим точечно.

## Что проверить в игре
1. Истощить первый богатый мир (Terran/Ocean/Forest/Atoll) → оповещение + сбор Пыли
   с неистощённых миров вырос на +2/+4 (в тултипе Пылерождённого).
2. Истощить умеренный мир (например, Boreal) → оповещение + +1/+2 к сбору + бафф
   «+10% науки» на 10 ходов (проверить в списке эффектов империи, что он есть и
   пропадает через 10 ходов).
3. Истощить Ash → у Безвольных на истощённых мирах +3 промы (итого +6 на истощённой).
4. Ice → истощение планеты с 3 Пылерождёнными = +3 − 0.15 = 2.85/ход (с техникой −0.3).
5. Barren → пока мир не истощён, у Пылерождённых +4 науки; после полного истощения — 0.
6. Газовые гиганты — только после техники «Экстремальная атмосфера» (Экономика и торговля).
7. Если какой-то тип не даёт оповещение — сверить имя PlanetType (см. выше).
