# DeadAir Scripts — mlog4 fork — Changelog

> **English version below. Русская версия ниже.**

---

## English

This is a community-maintained compatibility fork of **DeadAir Scripts** by DeadAirRT, distributed under the GPL-3.0 licence the original author left when retiring (2025-09-23). The original gameplay logic is preserved unchanged — this fork only fixes things broken or hidden against the current X4 Foundations baseline.

**Companion mod** (recommended): [DeadAir Eco — mlog4 fork](https://github.com/mlog4/deadair_eco). This mod has hard cross-references to wares defined in Eco; running Scripts without Eco produces ~124 lookup errors per session.

### v1.13.1-beta1 — 2026-04-29

**Tested on**: X4 Foundations **9.0 beta 7**.
**Save-game**: should be compatible with existing saves (all changes are XML patches, no save-state schema changes).
**Upstream baseline**: DeadAir Scripts v1.13 (DeadAirRT, final release 2025-02-22).

#### Fixed (player-visible)

- **Dynamic News "New Station Started" event restored.** With X4 9.x's new prefab-construction handling, vanilla wraps the original `Request_Factory` body in a new `<do_else>`. The DA patch hooking the news signal failed silently (or fired twice in the `BuildInSector` case). News now correctly fires once when an NPC starts building a new factory.
- **Dynamic War — "Nemesis" / "Best Friends" event ending bug fixed.** When the `EndNow` flag was set on a relation between Faction A and Faction B, the mirror entry (B → A) was not flagged, so the war/peace event never properly ended from the second faction's perspective. Both directions now end together.
- **`DAJobsEXPLocFactionToCheck` no-op fixed.** A missing `$` prefix on a variable name made the engine treat it as a string literal, so the job-location filter was always empty. Now correctly references the variable.

#### Behavioural defaults changed

- **Infestation subsystem disabled by default.** The author's own comment was `<!-- TODO: Maybe play the game eventually, idk -->`. Patrol logic is incomplete (4 cues with empty actions); leaving it on caused silent failures. You can re-enable it manually via the in-game DA menu if you want to experiment.
- **DA's UI debug logging turned off by default.** Two cues had `$DAMenuDebugEnable = true` as the shipped default, which caused every UI menu interaction to write a debug line at 100% chance — flooding `script.log`. Now `false` by default. (This is DA's own debug toggle, separate from any mlog instrumentation.)

#### New diagnostic logging (no behavioural change)

- **Evolution Xenon ware probe.** The `LibraryEvolutionSetEQMods` library references 60 Xenon equipment-mod ware ids. If any are missing in the loaded mod stack, `add_equipment_mods` silently no-ops — Xenon never visibly evolve. We now log `[EVOLUTION/DIAG]` lines that probe `ware.X.exists?` for each id before use; check `script.log` for a missing-ware list if Xenon evolution looks broken.
- **Blueprint add probe.** `LibraryBPCheckTable` calls `add_blueprints` from inside an `instantiate="true"` cue context, where engine quirks can silently drop entity writes. We now log `[BP/DIAG]` lines around the call to detect silent failures.

#### Cleanup (debug.log)

- One persistent `=ERROR=` patch failure per session removed (the `factionlogic_economy.xml` XPath fix above).

#### Known issues / not yet addressed

- X4 9.0 release version not yet tested (currently testing on 9.0 beta 7). When 9.0 ships, we will retest and bump the build.
- `DALibraryGetMilitaryStrength` runs 8 `find_ship` calls per faction at every Dynamic War tick — performance issue, not yet refactored.
- A handful of upstream typos in saved-state field names (`Unkown`, `Evoltuion`, `InfSpeading`) cannot be fixed without breaking save-game compatibility — left as-is.

#### Credits

- Original mod: **DeadAirRT**.
- v1.14 community fork: **Sleaker** (we share the GPL-3.0 baseline; this fork descends from DA v1.13 not v1.14).
- 9.x compatibility maintenance: **mlog4**.

#### Licence

GPL-3.0 inherited from upstream. The original `LICENSE` file is preserved unchanged in this repository.

---

## Русский

Это форк-поддержка совместимости мода **DeadAir Scripts** от DeadAirRT, распространяется по лицензии GPL-3.0 которую автор оставил уходя (23.09.2025). Геймплейная логика оригинала сохранена без изменений — этот форк только чинит то что сломалось или было скрыто относительно текущей версии X4 Foundations.

**Парный мод** (рекомендуется): [DeadAir Eco — mlog4 fork](https://github.com/mlog4/deadair_eco). Этот мод жёстко зависит от товаров определённых в Eco; запуск Scripts без Eco даёт ~124 lookup-ошибки за сессию.

### v1.13.1-beta1 — 29.04.2026

**Тестировалось на**: X4 Foundations **9.0 beta 7**.
**Сейв-игры**: должны быть совместимы (все изменения — XML-патчи, схема сохранения не трогается).
**Upstream-базовая версия**: DeadAir Scripts v1.13 (DeadAirRT, финальный релиз 22.02.2025).

#### Исправлено (видно игроку)

- **Восстановлено событие Dynamic News «New Station Started».** В X4 9.x в куе `Request_Factory` появилась обёртка `<do_else>` для prefab-конструкции — это ломало патч, который прицеплял news-сигнал. Событие либо не срабатывало, либо дублировалось в случае `BuildInSector`. Теперь news корректно срабатывает один раз когда NPC начинает строить новую фабрику.
- **Исправлен баг завершения событий Dynamic War («Nemesis», «Best Friends» и т.д.).** При установке флага `EndNow` на отношения A→B зеркальная запись B→A не флагалась, и со стороны второй фракции событие никогда правильно не заканчивалось. Теперь обе стороны заканчивают одновременно.
- **`DAJobsEXPLocFactionToCheck` no-op исправлен.** Из-за отсутствия префикса `$` движок принимал имя переменной за строковый литерал — фильтр локаций для джобов всегда был пуст. Теперь переменная корректно резолвится.

#### Изменены значения по умолчанию

- **Подсистема Infestation отключена по умолчанию.** Комментарий автора: `<!-- TODO: Maybe play the game eventually, idk -->`. Patrol-логика недоделана (4 cue с пустыми actions). Ранее включалась автоматически. Если хотите экспериментировать — включается вручную через DA-меню в игре.
- **Debug-логирование DA-меню отключено по умолчанию.** Два cue имели `$DAMenuDebugEnable = true` по умолчанию — каждое взаимодействие с UI писало строку в `script.log` с шансом 100%, заваливая лог. Теперь `false`. (Это DA-вский тоггл, отдельно от любых mlog-инструментаций.)

#### Новое диагностическое логирование (без изменений поведения)

- **Проверка Xenon ware для Evolution.** Библиотека `LibraryEvolutionSetEQMods` ссылается на 60 ware-идентификаторов Xenon-эквипа. Если хотя бы один отсутствует в загруженном стеке модов — `add_equipment_mods` тихо ничего не делает, и Xenon-ы видимо не эволюционируют. Добавлены строки `[EVOLUTION/DIAG]` которые проверяют `ware.X.exists?` для каждого id перед использованием; если эволюция Xenon выглядит сломанной — проверьте `script.log` на список отсутствующих ware.
- **Проверка добавления чертежей.** `LibraryBPCheckTable` вызывает `add_blueprints` внутри cue с `instantiate="true"`, где из-за движкового quirk-а entity-записи могут тихо проваливаться. Добавлены строки `[BP/DIAG]` вокруг вызова чтобы детектировать тихие провалы.

#### Чистка (debug.log)

- Убрана одна постоянная `=ERROR=` ошибка патчей за сессию (фикс XPath-а в `factionlogic_economy.xml` выше).

#### Известные проблемы / ещё не сделано

- Совместимость с релизной X4 9.0 пока не проверялась (тестируем на 9.0 beta 7). Когда выйдет релиз — перепроверим и обновим билд.
- `DALibraryGetMilitaryStrength` делает 8 вызовов `find_ship` на фракцию при каждом тике Dynamic War — проблема производительности, рефакторинг отложен.
- Несколько опечаток в именах полей сохраняемого состояния (`Unkown`, `Evoltuion`, `InfSpeading`) нельзя поправить без поломки save-game-совместимости — оставлены как есть.

#### Благодарности

- Оригинальный мод: **DeadAirRT**.
- Community-форк v1.14: **Sleaker** (общая GPL-3.0 база; наш форк происходит от DA v1.13, а не v1.14).
- Поддержка совместимости с 9.x: **mlog4**.

#### Лицензия

GPL-3.0, унаследована от upstream-а. Оригинальный файл `LICENSE` сохранён в репозитории без изменений.
