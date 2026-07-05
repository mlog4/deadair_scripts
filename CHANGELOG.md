# DeadAir Scripts — mlog4 fork — Changelog

> **English version below. Русская версия ниже.**


### v1.13.1-mlog012bp — 2026-07-04

**Tested on**: X4 Foundations **9.0 release** (2026-06-10 onwards). 5h 45min multi-mod soak with mlog_galactic_heroes + deadair_eco (mlog006rel) + SirNukes Mod Support APIs. Full DA feature set active with zero errors in the mod's namespace.

**Summary**: This build restores TWO major DA features that were previously either fully or partially disabled by mlog004 for X4 9.0 beta compatibility. God can now expand staged NPC stations by advancing their planned construction stages (mlog011 Option A), and Blueprint Scanning police-asset path is re-hooked into the modern vanilla scan loop (mlog012bp).

#### Changed

##### mlog011 — DA God Option A: vanilla-style stage advance for staged stations

X4 9.0 release marks effectively all NPC stations with `hasstagedconstruction=true`. The mlog004 guard we added to eliminate 200 000+ "cannot append to fixed sequence" errors per session had the side effect of blocking `EventGodExpandStation` on all such stations — DA God became a full no-op for expansion of existing stations.

Analysis of vanilla `md/factionlogic_economy.xml` revealed that vanilla itself handles staged expansion via `<add_build_to_expand_station>` with a `<stage>` param — the action advances the station's own planned construction plan by one stage rather than trying to append new modules. This is compatible with the 9.0 "fixed sequence" restriction.

Rewritten the two main expand paths in `md/deadairdynamicuniverse.xml` (Xenon shipyard evolution site at line ~3286, and God main expand at line ~10447) to route staged stations with future stages and idle build processors through `add_build_to_expand_station`, while non-staged stations continue using the original `create_construction_sequence` path with DA's custom modules. Sites without future stages or currently building are correctly skipped.

**Diff**: `md/deadairdynamicuniverse.xml` — 3 sites reworked with `<do_elseif>` branches; new `[GOD/DIAG] STAGE-ADVANCED` diagnostic log line added.

**Impact**: DA God is now fully operational on 9.0. In a 5h 45min soak: 46 stage advances observed, 262 new station orders issued, 262 "Ordering Stations" events. Zero "cannot append to fixed sequence" errors, down from tens of thousands per session in the previous behaviour.

##### mlog012bp — Blueprint scanning police-asset path re-enabled

`aiscripts/order.move.recon.xml` XPath was rewritten against the vanilla 9.0 release structure. The equivalent scan-complete-clear point is now under a `<do_else>` at line ~1601 (was line ~1597 in beta 8, and used to be inside a `do_if[$police]/do_else` at line ~1310 in 8.x). New XPath uses `starts-with()` on the sibling `debug_text` prefix to disambiguate from the interrupt-triggered clear branch (line ~205, "called to attack. clearing target").

**Impact**: 37 blueprints awarded across the 5h 45min soak (turrets, engines, shields, ships, station modules) from Argon, Teladi, and Paranid faction ships. Vanilla `policeassetscannedship` signal fires at the correct location; DA's `EventBPPlayerAssetScannedShip` cue receives it; `EventBPAddProgress` awards blueprints via `add_blueprints` action. Feature gate `$DABPEnable` in DA menu is respected.

#### Deployed since mlog007rel

- v1.13.1-mlog008dgod — Phase 1 diagnostic instrumentation on 4 God cues.
- v1.13.1-mlog009rl — Attempted relaxed guard (rolled back — 26k errors).
- v1.13.1-mlog010rvt — Reverted to strict mlog004 guard after failed relaxation attempt.
- v1.13.1-mlog011opta — Option A stage-advance implementation.
- v1.13.1-mlog012bp — BP scanning XPath rewrite. **This release.**

#### Soak validation summary (2026-07-04, 5h 45min)

| Feature | Events | Verdict |
|---------|-------:|---------|
| Dynamic War | 572 | working |
| Dynamic News | 454 | working |
| Xenon Evolution | 354 | working |
| DA Fill | 19883 | working |
| DA Jobs SST | 7437 | working |
| DA God new orders | 262 | working |
| DA God STAGE-ADVANCED | 46 | working (new) |
| DA Blueprint awards | 37 | working (restored) |
| `cannot append to fixed sequence` errors | 0 | fixed |
| DAGod-namespace errors | 0 | clean |
| `mlog_da_scripts` errors | 0 | clean |

#### Publishing

Ready for Steam Workshop and Nexus Mods.

---

### v1.13.1-mlog007rel — 2026-07-04

**Tested on**: X4 Foundations **9.0 release** (2026-06-10 onwards). 5+ hour multi-mod soak with mlog_galactic_heroes + deadair_eco (mlog006rel) + SirNukes Mod Support APIs. Full validation of all major DA modules:

- DADynamicWar — fired (BigBoost between Terran + Teladi observed)
- DADynamicNews — fired repeatedly (sector ownership + station expansion + relation shift reports)
- DAEvolution — 10 evolved Xenon fleets active across 5 sectors, cap enforced
- DAJobsEXP — all 9 expedition jobs active from t=306s
- DAFill — station cargo top-ups running (BUC, SCA, ARG, TEL observed)
- DA Wares — all Eco cross-references resolved (da_mil_schematics, da_adv_schematics, da_laborunion_contracts)
- Xenon evolution ware exists check: **60/60** (L1-L10 × engine/ship/shield/weapon/missile/eco)

**Zero MD errors from deadair_scripts namespace.** The only DA-mentioned errors in the log were the known `[GodStationEntry]` "unable to find suitable locations" warnings from DA God when the universe is already densely populated — cosmetic, not blocking.

**Save-game**: compatible with existing 8.x saves loaded on 9.0.

#### Changed
- `content.xml` — bumped `<dependency version="700"/>` —→ `"900"` (X4 9.0 target).
- `content.xml` — removed "updated for beta 8 vanilla diffs" statement (superseded by release validation).
- No code changes since build_007 (2026-04-30). This is a metadata / publication release only.

#### Publishing
- Ready for Steam Workshop and Nexus Mods.
- Companion mod: [deadair_eco (X4 9.x compat fork)](https://github.com/mlog4/deadair_eco). Install both.

---

---

## English

This is a community-maintained compatibility fork of **DeadAir Scripts** by DeadAirRT, distributed under the GPL-3.0 licence the original author left when retiring (2025-09-23). The original gameplay logic is preserved unchanged — this fork only fixes things broken or hidden against the current X4 Foundations baseline.

**Companion mod** (recommended): [DeadAir Eco — mlog4 fork](https://github.com/mlog4/deadair_eco). This mod has hard cross-references to wares defined in Eco; running Scripts without Eco produces ~124 lookup errors per session.

### v1.13.1-beta1 — 2026-04-30

**Tested on**: X4 Foundations **9.0 beta 7** (initial). Updated for **9.0 beta 8** vanilla diffs (analysis only — beta 8 game-load test pending).
**Save-game**: should be compatible with existing saves (all changes are XML patches, no save-state schema changes).
**Upstream baseline**: DeadAir Scripts v1.13 (DeadAirRT, final release 2025-02-22).

#### Fixed (player-visible)

- **Dynamic News "New Station Started" event restored.** With X4 9.x's new prefab-construction handling, vanilla wraps the original `Request_Factory` body in a new `<do_else>`. The DA patch hooking the news signal failed silently (or fired twice in the `BuildInSector` case). News now correctly fires once when an NPC starts building a new factory.
- **Dynamic War — "Nemesis" / "Best Friends" event ending bug fixed.** When the `EndNow` flag was set on a relation between Faction A and Faction B, the mirror entry (B → A) was not flagged, so the war/peace event never properly ended from the second faction's perspective. Both directions now end together.
- **`DAJobsEXPLocFactionToCheck` no-op fixed.** A missing `$` prefix on a variable name made the engine treat it as a string literal, so the job-location filter was always empty. Now correctly references the variable.
- **DynamicNews `find_sector` load errors fixed (X4 9.0 beta 8).** Three calls to `<find_sector>` for the DynamicNews sector list lacked the now-required `space=` attribute, throwing hard load errors in 9.0 beta 8 (was silent no-op in 8.x). Added `space="player.galaxy"` to all three.
- **Massive log-spam fix: 200 000+ "ConstructionPlan::Append cannot append to fixed sequence" errors per session eliminated.** X4 9.0 introduced "staged construction" — once a station's plan is marked staged, vanilla rejects appending modules to it (the new vanilla `ExpandPrefabs` cue in `factionlogic.xml` v18 is the proper mechanism). DA's three `<create_construction_sequence>` call sites — `EventGodExpandStation`, `EventGodCheckKeyStation`, and `EventEvolutionUpgradeShipyard` — now check `not $Station.hasstagedconstruction` before appending, and silently skip staged stations (vanilla handles them). Dramatic debug-log clean-up; no observable gameplay change beyond fewer errors.

#### Disabled / deferred

- **Blueprint scanning patch on `aiscripts/order.move.recon.xml` temporarily disabled.** Vanilla 9.x restructured the police-recon scan loop deeply; DA's XPath into `do_if[$police]/do_else/do_if[isclass.ship]/do_if[$haltedby]` no longer matches and the patch was failing to load. Until a proper rewrite is done, the `policeassetscannedship` signal does not fire, so DA's Blueprint Scanning feature does not advance from player-asset police scans. The independent feature gate `$DABPEnable` is unaffected; if it was off, you see no change. Other BP unlock paths in DA continue working.

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

### v1.13.1-beta1 — 30.04.2026

**Тестировалось на**: X4 Foundations **9.0 beta 7** (изначально). Обновлено под vanilla-диффы **9.0 beta 8** (только аудит — повторный game-load тест на beta 8 ожидается).
**Сейв-игры**: должны быть совместимы (все изменения — XML-патчи, схема сохранения не трогается).
**Upstream-базовая версия**: DeadAir Scripts v1.13 (DeadAirRT, финальный релиз 22.02.2025).

#### Исправлено (видно игроку)

- **Восстановлено событие Dynamic News «New Station Started».** В X4 9.x в куе `Request_Factory` появилась обёртка `<do_else>` для prefab-конструкции — это ломало патч, который прицеплял news-сигнал. Событие либо не срабатывало, либо дублировалось в случае `BuildInSector`. Теперь news корректно срабатывает один раз когда NPC начинает строить новую фабрику.
- **Исправлен баг завершения событий Dynamic War («Nemesis», «Best Friends» и т.д.).** При установке флага `EndNow` на отношения A→B зеркальная запись B→A не флагалась, и со стороны второй фракции событие никогда правильно не заканчивалось. Теперь обе стороны заканчивают одновременно.
- **`DAJobsEXPLocFactionToCheck` no-op исправлен.** Из-за отсутствия префикса `$` движок принимал имя переменной за строковый литерал — фильтр локаций для джобов всегда был пуст. Теперь переменная корректно резолвится.
- **Исправлены load-ошибки `find_sector` для DynamicNews (X4 9.0 beta 8).** Три вызова `<find_sector>` для построения списка секторов DynamicNews не имели обязательного теперь атрибута `space=` — в 9.0 beta 8 это hard load error (в 8.x было silent no-op). Добавлен `space="player.galaxy"` во все три.
- **Устранены 200 000+ ошибок «ConstructionPlan::Append cannot append to fixed sequence» за сессию.** X4 9.0 ввёл «staged construction» — как только план станции помечен staged, vanilla отклоняет append модулей (новый vanilla cue `ExpandPrefabs` в `factionlogic.xml` v18 — правильный механизм для таких станций). Три DA-евских вызова `<create_construction_sequence>` — `EventGodExpandStation`, `EventGodCheckKeyStation`, `EventEvolutionUpgradeShipyard` — теперь проверяют `not $Station.hasstagedconstruction` перед append-ом и тихо пропускают staged-станции (их обрабатывает vanilla). Резкая чистка debug-лога; геймплей-видимых изменений нет (кроме отсутствия ошибок).

#### Отключено / отложено

- **Патч Blueprint Scanning в `aiscripts/order.move.recon.xml` временно отключён.** Vanilla 9.x значительно перестроила loop полицейского сканирования; DA-евский XPath на `do_if[$police]/do_else/do_if[isclass.ship]/do_if[$haltedby]` больше не матчится — патч не загружался. До правильного rewrite сигнал `policeassetscannedship` не срабатывает, поэтому фича Blueprint Scanning от player-asset police-сканов не прогрессирует. Независимый feature-гейт `$DABPEnable` не затронут — если он был выключен, наблюдаемых изменений нет. Прочие пути unlock blueprint в DA продолжают работать.

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
