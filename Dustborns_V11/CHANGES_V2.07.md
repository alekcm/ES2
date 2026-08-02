# Dustborns v2.07 — изменения по заказу

## 1. Переваривание больше не работает на Безвольных и Считающих Оболочках

**Файлы:**
- `Simulation/PopulationDefinitions[Dustborns].xml`
- `Simulation/ConstructibleElement_Industry[StarSystemImprovement].xml`

**Что сделано:**
- `AffinityBezvolny` и `AffinityBezvolnyScholar` получили `IsInvincible="true"` —
  это штатный атрибут `PopulationDefinition` (см. `PopulationDefinition.xsd`):
  постройки-конверсии (без `IgnoreInvinciblePopulation`) больше **никогда** не выберут
  такую популяцию в качестве цели. Оболочка уже переварена — переваривать её повторно нельзя.
- В обе постройки — «Переварить в Безвольного» и «Переварить в Считающую Оболочку» —
  добавлена проверка: переваривать можно только если
  `Population − MajorPopulationCount − BezvolnyCount − ScholarCount > 0`
  (счётчики Безвольных и Оболочек читаются через их агрегирующие дескрипторы
  `ClassPopulationStarSystemAffinityBezvolny(Count)`/`...BezvolnyScholar(Count)`,
  четвёртый аргумент `true` у `Property()` = «вернуть 0, если популяции в системе нет»).
  Когда в системе остались только оболочки, кнопка гаснет (состояние «недостаточно населения»)
  вместо молчаливого «съедания» собственных Безвольных.

*Проверить в игре:* на планете с 1 чужим минором + 1 Безвольным конверсия превращает минора
(раньше могла выбрать Безвольного — эвристика LowestCount); при одних Безвольных кнопка недоступна.

## 2. «Терпеливые Ветра» — истощение −0.1 за КАЖДОГО Пылерождённого

**Файл:** `Simulation/SimulationDescriptors[DustbornsQuests].xml`

**Было (плоско, на всю империю):**
```xml
<Modifier TargetProperty="PlanetDepletionPerTurn" Operation="Subtraction" Value="0.1"
          Path="../ClassEmpire/ClassColonizedStarSystem/ClassColonizedPlanet" .../>
```

**Стало (за каждого Пылерождённого на планете):**
```xml
<BinaryModifier TargetProperty="PlanetDepletionPerTurn" Operation="Subtraction" Left="0.1"
                BinaryOperation="Multiplication" Right="$(PopulationCount)"
                Path="../ClassEmpire/ClassColonizedStarSystem/ClassColonizedPlanet/ClassPopulation,ClassPopulationPlanetAffinityDustbornsCount"
                SearchValueFromPath="true" .../>
```
- `$(PopulationCount)` читается с той же сущности, что и путь (`SearchValueFromPath`)
  — это число Пылерождённых **на этой планете**.
- Результат: 1 Пылерождённый → истощение +1 → +0.9/ход; 3 → +3 → +2.7/ход.
- Локализация техники обновлена: «Каждый Пылерождённый истощает планету на 0.1 в ход медленнее».

## 3. Оповещения при выполнении скрытых квестов «Каталога Миров»

**Файлы:**
- `Quests/QuestDefinitions[Dustborns].xml`
- `Localization/english/ES2_Localization_Locales.xml`

**Что сделано:** во все 16 скрытых квестов-слушателей (по одному на тип планеты:
Arctic, Asteroid, Atoll, Arid, Ash, Barren, Desert, Forest, Frozen, Jungle, Lava,
Ocean, Steppe, Terran, Toxic, Tundra) после выдачи награды добавлен

```xml
<Action_Message Message="%DustbornsCatalogListener_<Тип>_Message">
    <Input_Empires VarName="$CurrentEmpire"/>
</Action_Message>
```

При первом полном истощении планеты типа игроку выходит оповещение в духе
«Пожрав пепловый мир, мы выяснили: даже пепел помнит, кем был.
Запись в Каталоге Миров: +2/+4 к сбору Пыли с неистощённых миров.» —
т.е. игрок всегда видит и лор, и награду (+2 обычные / +4 Teeming к сбору Пыли
за запись в каталоге, пока миры не истощены).

16 новых ключей локализации: `%DustbornsCatalogListener_<Тип>_Message`.

## 4. Прочее
- Версия мода поднята до 2.07 (`Dustborns.xml`, ReleaseNotes).
- `DESIGN_Quests_and_Tech.md` приведён в соответствие.

## Примечание по «Action_Message»
`Action_Message` — штатное действие квеста (есть в `QuestDefinition.xsd`):
`Message` = ключ локализации, `Input_Empires` = кому показать. Если в игре сообщение
не появится или выведется сам ключ — проверьте Diagnostics.log: возможно, в вашей
версии игры атрибут ожидает текст без `%` (тогда достаточно убрать `%` из атрибутов
`Message`, ключи локализации оставить). Структура XML при этом не меняется.
