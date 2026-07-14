# fullStat — индекс и заметки по данным TLI

Обновлено: 2026-07-14 22:37 MSK.

## reference_tldbInfo.json

Кратко: это локальный справочник TLIDB для сезона `lunaria`. Он связывает названия из leaderboard/stat snapshots с метаданными TLIDB: иконки, ссылки, теги, краткие эффекты, типы предметов/питомцы и частоту появления в лидерборде. Сам файл в документацию не копируем; используем как источник lookup-данных.

### Паспорт файла

- schema: `tlidb-reference-v1`
- version: `1`
- generated_at: `2026-07-07T15:26:42Z`
- source: `https://tlidb.com/`
- source_language: `en`
- season_id: `lunaria`
- asset_root: `icons`
- asset_count: `1383`
- asset_issue_count: `0`

### Категории records

- `main_skills`: 174 записей
- `support_skills`: 405 записей
- `unique_items`: 321 записей
- `talent_boards`: 30 записей
- `talent_skills`: 157 записей
- `servants`: 167 записей
- `hero_characters`: 156 записей

### Общая схема записи

Почти все категории имеют общий набор полей:
- `name` — человекочитаемое имя, ключ для join со статистикой.
- `category` — машинная категория (`main_skills`, `servants` и т.д.).
- `category_label` — лейбл категории для UI.
- `tlidb_url` — страница TLIDB; можно делать кликабельные ссылки.
- `tlidb_title` — заголовок страницы TLIDB.
- `icon` — локальный путь ассета, если папка icons/data доступна.
- `source_icon_url` — официальная CDN-иконка TLIDB; лучший вариант для нашего HTML.
- `tags` — теги механик: Attack, Spell, Projectile, Minion, Fire и т.п.
- `short_effects` — короткие строки эффекта; подходят для tooltip/карточки.
- `slot_or_type` — тип/слот предмета или сущности, если применимо.
- `drop_sources` — источники дропа, если TLIDB их дал.
- `season_tags` — сезонные пометки, если есть.
- `source_updated_at` — время генерации/обновления записи.
- `leaderboard_count` — сколько раз сущность встретилась в собранном leaderboard; полезно для популярности.

### Дополнительные поля по категориям

- `servants`: есть `pactspirit_type`, `rarity`, `classification_line`, `effect_lines`. Это позволит делать аналитику популярных pactspirits/питомцы и показывать их эффекты.
- `hero_characters`: есть `parent` — связь варианта/trait с базовым героем. Это важно для группировки `Rosa 2`, `Iris 2`, `Sage 1` и т.д.

### Покрытие иконок

- `main_skills`: source_icon_url есть у 172/174
- `support_skills`: source_icon_url есть у 405/405
- `unique_items`: source_icon_url есть у 321/321
- `talent_boards`: source_icon_url есть у 30/30
- `talent_skills`: source_icon_url есть у 132/157
- `servants`: source_icon_url есть у 167/167
- `hero_characters`: source_icon_url есть у 156/156

### Топ по leaderboard_count внутри TLIDB reference

- `main_skills`: Chromatic Shot (104762), Thunder Spike (55880), Timid (49289), Ice Lances (38745), Bull's Rage (36276), Spectral Slash (35364)
- `support_skills`: Extended Duration (171375), Cooldown Reduction (161739), Spell Tangle (138400), Manifold Entanglement (137164), Multiple Projectiles (135654), Quick Decision (113248)
- `unique_items`: Born in Fire (85190), Grace Boots (72827), False God's Skin (65821), Rock Lizard's Skull (56283), Old Grudge (51739), Golden Touch (47618)
- `talent_boards`: Lich (250559), Goddess of Knowledge (238739), Elementalist (159930), Goddess of Hunting (152449), Marksman (149721), Prophet (132254)
- `talent_skills`: Off The Beaten Track (216406), Peculiar Vibe (148420), Gale (134861), Quick Wits (106390), Extreme Coldness (100908), Rushed (84464)
- `servants`: Cloudgatherer - Surging Gold (281876), Dreamweaver (172995), Red Umbrella (142195), Chalk Spirit (135509), Happy Chonky (135466), Cloudgatherer - Sky (106433)

### Чем файл поможет отчёту

- Иконки: брать `source_icon_url` для skills/items/питомцы/hero traits вместо локальных путей.
- Tooltips: показывать `tags` и `short_effects` по наведению/в карточке.
- Ссылки: делать название кликабельным через `tlidb_url`.
- Join со статистикой: `main_skills[name]`, `servants[name]`, `hero_characters[trait/full name]` для обогащения leaderboard rows.
- Популярность: `leaderboard_count` как независимый сигнал частотности, рядом с нашими региональными снапшотами.
- Фильтры: по тегам (`Projectile`, `Minion`, `Melee`, `Spell`, `Persistent`, элементы) можно строить срезы стартеров.
- Питомцы: частотность + rarity/effect_lines поможет понять дорогие/метовые pactspirits.
- Hero grouping: через `parent` можно аккуратнее группировать traits по базовым героям.

### Ограничения

- Это справочник, а не сам leaderboard: DPS/level/player/region лежат в `regions_*_snapshots.json`.
- `leaderboard_count` зависит от методики сборки reference; использовать как сигнал, не как абсолютную истину.
- Названия в снапшотах могут отличаться пробелами/вариантами (`Scent Weaver Sage|Licorice Note` vs `Scent Weaver Sage | Licorice Note`), нужен нормализатор строк.
- Полные эффекты могут быть длиннее `short_effects`; для глубокого разбора лучше ходить по `tlidb_url` или расширять reference.

### Предлагаемая структура будущего пайплайна

1. Загрузить `regions_ASIA/EU/NA_snapshots.json`.
2. Нормализовать поля героя/trait/skill/питомцев.
3. Join с `reference_tldbInfo.records` по имени.
4. Добавить в HTML enriched rows: иконки, TLIDB ссылки, tags, short_effects.
5. Посчитать агрегаты: top hero+skill, top питомцы, hidden/open builds, регионы, срезы по времени.
6. Отдельно смотреть ASIA как ранний сигнал китайской меты.

## regions_ASIA_snapshots.json

Кратко: `ASIA`-файл имеет ту же схему, что EU: 5000 строк leaderboard одного снапшота `HR111401__sc` в режиме `sc`. Используем для сравнения региональной меты, особенно ASIA как прокси китайской/азиатской меты.

### Паспорт

- строк: `5000`
- `ms`: `HR111401__sc`
- `mode`: `sc`
- уровни: `99`–`100`
- max DPS: `1,243,659,376,968,104`
- hidden builds: `529/5000` (10.58%)

### Топ героев по количеству

- `Sage 1`: 1698 строк (33.96%), hidden 152, max DPS 24,557,242,826,301
- `Rosa 2`: 838 строк (16.76%), hidden 87, max DPS 1,243,659,376,968,104
- `Thea 1`: 404 строк (8.08%), hidden 45, max DPS 703,837,500,295
- `Moto 2`: 315 строк (6.3%), hidden 23, max DPS 518,858,150,737
- `Erika 1`: 185 строк (3.7%), hidden 13, max DPS 32,520,805,194
- `Iris 2`: 182 строк (3.64%), hidden 34, max DPS 6,464,893,139,714
- `Youga 2`: 166 строк (3.32%), hidden 19, max DPS 11,319,089,991
- `Carino 2`: 162 строк (3.24%), hidden 11, max DPS 451,825,327,050
- `Rehan 2`: 137 строк (2.74%), hidden 19, max DPS 691,236,787,433
- `Erika 3`: 107 строк (2.14%), hidden 9, max DPS 101,068,091,303
- `Bing 2`: 104 строк (2.08%), hidden 12, max DPS 8,601,961,256
- `Erika 2`: 101 строк (2.02%), hidden 20, max DPS 146,195,239,553

### Топ hero + skill по количеству

- `Sage 1 + Chromatic Shot`: 751 строк (15.02%), max DPS 4,414,814,946,081
- `Sage 1 + UNKNOWN`: 350 строк (7.0%), max DPS 5,078,645,741,422
- `Sage 1 + Split Shot`: 258 строк (5.16%), max DPS 24,557,242,826,301
- `Rosa 2 + UNKNOWN`: 194 строк (3.88%), max DPS 220,969,722,558,014
- `Rosa 2 + Lightning Shot`: 150 строк (3.0%), max DPS 779,498,681,959,602
- `Rosa 2 + Thunder Spike`: 137 строк (2.74%), max DPS 1,960,259,108,917
- `Thea 1 + True Flame Immolation`: 134 строк (2.68%), max DPS 116,900,499,003
- `Moto 2 + Thunderbolt Overload`: 126 строк (2.52%), max DPS 9,618,184,874
- `Rosa 2 + Split Shot`: 107 строк (2.14%), max DPS 1,243,659,376,968,104
- `Thea 1 + UNKNOWN`: 92 строк (1.84%), max DPS 140,392,746,872
- `Rosa 2 + Chain Lightning`: 82 строк (1.64%), max DPS 11,422,171,764,153
- `Carino 2 + Chromatic Shot`: 81 строк (1.62%), max DPS 36,555,352,057
- `Erika 1 + Thunder Spike`: 79 строк (1.58%), max DPS 32,520,805,194
- `Iris 2 + UNKNOWN`: 58 строк (1.16%), max DPS 6,464,893,139,714
- `Rosa 2 + Berserking Blade`: 58 строк (1.16%), max DPS 3,222,275,268,613

### Топ hero + skill по max DPS

- `Rosa 2 + Split Shot`: max DPS 1,243,659,376,968,104, строк 107
- `Rosa 2 + Frost Spike`: max DPS 814,379,446,851,753, строк 47
- `Rosa 2 + Lightning Shot`: max DPS 779,498,681,959,602, строк 150
- `Rosa 2 + UNKNOWN`: max DPS 220,969,722,558,014, строк 194
- `Sage 1 + Split Shot`: max DPS 24,557,242,826,301, строк 258
- `Moto 1 + Rising Edge`: max DPS 12,101,651,647,300, строк 1
- `Rosa 2 + Chain Lightning`: max DPS 11,422,171,764,153, строк 82
- `Carino 3 + UNKNOWN`: max DPS 7,024,235,592,850, строк 24
- `Iris 2 + UNKNOWN`: max DPS 6,464,893,139,714, строк 58
- `Rosa 2 + Whirlwind`: max DPS 5,460,521,230,115, строк 5
- `Sage 1 + UNKNOWN`: max DPS 5,078,645,741,422, строк 350
- `Rosa 2 + Groundshaker`: max DPS 4,572,953,414,694, строк 3

### Топ питомцы

- `Twilight Sparks`: 2224 (49.74% открытых билдов)
- `Elixir Fairies`: 1712 (38.29% открытых билдов)
- `Dreamweaver`: 1628 (36.41% открытых билдов)
- `Cloudgatherer - Surging Gold`: 1613 (36.08% открытых билдов)
- `Preserver of Eternity`: 1501 (33.57% открытых билдов)
- `Chalk Spirit`: 1094 (24.47% открытых билдов)
- `Miss Melancholy - Cyan`: 1055 (23.6% открытых билдов)
- `Doro-Doro Dorothy`: 988 (22.1% открытых билдов)
- `Red Umbrella`: 871 (19.48% открытых билдов)
- `Iron Lion`: 819 (18.32% открытых билдов)
- `Avatar of Moon`: 625 (13.98% открытых билдов)
- `Miss Melancholy - Pink`: 573 (12.82% открытых билдов)
- `Swaying Bonnie`: 571 (12.77% открытых билдов)
- `Drapion Lady`: 568 (12.7% открытых билдов)
- `Happy Chonky`: 511 (11.43% открытых билдов)

### Первичные выводы ASIA

- ASIA сильнее смещена в `Sage 1 + Chromatic Shot/Split Shot`: Sage 1 = 33.96% строк.
- Rosa 2 меньше по количеству, чем EU, но имеет экстремальные top-DPS outliers через Split Shot/Lightning Shot/Frost Spike.
- Moto 2 заметнее, чем в EU/NA, что полезно для проверки minion/Synthetic направления.
- Hidden выше, чем EU/NA: 529/5000, значит часть сильных сетапов не раскрыта.

## regions_EU_snapshots.json

Кратко: EU-файл — это список из 5000 записей leaderboard для одного снапшота `HR111401__sc` в режиме `sc`. Внутри уже есть детальные строки персонажей: игрок, герой, trait, уровень, DPS, основной skill/dps_skill, hidden-флаг и питомцев. Это более детальный источник, чем старые агрегаты top-связок.

### Паспорт EU-файла

- путь: `regions_EU_snapshots.json`
- тип: `list`
- строк: `5000`
- `ms`: `HR111401__sc`
- `mode`: `sc`
- уровни: `99`–`100`
- max DPS: `73,897,330,944,839`
- hidden builds: `229/5000`

### Схема строки

- `id` — уникальный id записи/персонажа, включает региональный префикс `EU-`.
- `ms` — id снапшота/рейтинга; здесь все строки `HR111401__sc`.
- `mode` — режим, здесь `sc`.
- `player` — ник игрока.
- `hero` — короткое имя героя/trait: `Rosa 2`, `Sage 1`, `Iris 2`.
- `hero_full` — полное имя героя + trait: `Lightbringer Rosa | Unsullied Blade`.
- `lvl` — уровень персонажа.
- `dps` — числовой DPS из рейтинга; 0 тоже возможен.
- `skill` — основной/показанный skill; может быть `null` у hidden.
- `dps_skill` — skill, по которому засчитан DPS; важнее для аналитики связок, но тоже может быть `null`.
- `hidden` — если `true`, билд скрыт; skill/dps_skill/питомцы часто отсутствуют.
- `servants` — список pactspirits/питомцы, обычно 6 штук; отсутствует у hidden.

### Полезные особенности

- В файле нет явных `snapshot`, `ranking`, `region`; регион задан именем файла, а тип рейтинга надо выводить из источника/контекста.
- Для связок лучше использовать `dps_skill`, fallback на `skill`, иначе `UNKNOWN`.
- `hidden=true` нельзя использовать для детального разбора skill/питомцы, но можно учитывать в доле скрытых билдов и top DPS.
- `servants` почти всегда содержит 6 элементов у открытых билдов; это отличный источник meta/pactspirit анализа.
- `hero_full` нужно нормализовать: встречается формат без пробела `Scent Weaver Sage|Licorice Note`.

### Топ героев по количеству строк EU

- `Rosa 2`: 1788 строк, hidden 64, max DPS 73,897,330,944,839
- `Sage 1`: 1075 строк, hidden 30, max DPS 6,306,634,617,842
- `Rehan 2`: 382 строк, hidden 12, max DPS 284,497,416,114
- `Thea 1`: 307 строк, hidden 9, max DPS 38,766,084,219
- `Erika 1`: 229 строк, hidden 7, max DPS 120,897,046,378
- `Youga 2`: 191 строк, hidden 6, max DPS 22,805,621,379
- `Carino 2`: 161 строк, hidden 9, max DPS 716,562,391,720
- `Iris 2`: 153 строк, hidden 14, max DPS 6,381,800,639,407
- `Rehan 1`: 91 строк, hidden 4, max DPS 62,275,536,074
- `Bing 2`: 78 строк, hidden 8, max DPS 11,887,471,955
- `Erika 2`: 77 строк, hidden 13, max DPS 216,082,106,581
- `Gemma 2`: 76 строк, hidden 3, max DPS 19,385,865,632

### Топ hero + dps_skill по количеству

- `Rosa 2 + Thunder Spike`: 773 строк, max DPS 1,397,087,574,106
- `Rosa 2 + Berserking Blade`: 454 строк, max DPS 2,358,507,868,124
- `Sage 1 + Chromatic Shot`: 407 строк, max DPS 977,843,786,351
- `Sage 1 + UNKNOWN`: 172 строк, max DPS 91,618,885,682
- `Rosa 2 + UNKNOWN`: 159 строк, max DPS 5,876,282,668,576
- `Rehan 2 + Thunder Spike`: 156 строк, max DPS 13,207,336,846
- `Thea 1 + Chromatic Shot`: 149 строк, max DPS 5,146,621,035
- `Rehan 2 + Spectral Slash`: 140 строк, max DPS 9,877,422,966
- `Erika 1 + Thunder Spike`: 112 строк, max DPS 120,897,046,378
- `Rosa 2 + Lightning Shot`: 95 строк, max DPS 5,316,515,442,533
- `Sage 1 + Frost Impact`: 92 строк, max DPS 226,595,024,810
- `Sage 1 + Burning Shot`: 89 строк, max DPS 19,772,892
- `Sage 1 + Groundshaker`: 79 строк, max DPS 27,723,454,376
- `Rosa 2 + Focused Slash`: 78 строк, max DPS 309,410,847,894
- `Youga 2 + Mind Control`: 77 строк, max DPS 22,805,621,379

### Топ hero + dps_skill по max DPS

- `Rosa 2 + Groundshaker`: max DPS 73,897,330,944,839, строк 20
- `Iris 1 + UNKNOWN`: max DPS 15,125,811,921,069, строк 4
- `Rosa 2 + Split Shot`: max DPS 12,240,676,899,220, строк 14
- `Iris 2 + Summon Erosion Magus`: max DPS 6,381,800,639,407, строк 22
- `Sage 1 + Life Tonic`: max DPS 6,306,634,617,842, строк 34
- `Rosa 2 + UNKNOWN`: max DPS 5,876,282,668,576, строк 159
- `Rosa 2 + Lightning Shot`: max DPS 5,316,515,442,533, строк 95
- `Rosa 2 + Spectral Slash`: max DPS 4,970,843,497,662, строк 26
- `Rosa 2 + Berserking Blade`: max DPS 2,358,507,868,124, строк 454
- `Rosa 2 + Blazing Bullet`: max DPS 2,112,384,161,708, строк 5
- `Iris 2 + UNKNOWN`: max DPS 1,444,918,653,682, строк 43
- `Rosa 2 + Thunder Spike`: max DPS 1,397,087,574,106, строк 773

### Топ питомцы EU

- `Red Umbrella`: 2382
- `Cloudgatherer - Surging Gold`: 2030
- `Dreamweaver`: 1983
- `Twilight Sparks`: 1729
- `Azure Gunslinger`: 1688
- `Miss Melancholy - Cyan`: 1255
- `Elixir Fairies`: 978
- `Chalk Spirit`: 950
- `Doro-Doro Dorothy`: 760
- `Avatar of Moon`: 745
- `Happy Chonky`: 744
- `Preserver of Eternity`: 699
- `Kitty Express`: 612
- `Iron Lion`: 605
- `Hell Cavalry`: 513

### Как использовать EU для итогового HTML

- Сделать вкладку `EU leaderboard`: top rows с иконкой героя/skill, DPS, lvl, hidden, питомцев.
- Сделать агрегаты `Hero meta`, `Skill meta`, `Hero+Skill`, `Мета питомцев`.
- Join с `reference_tldbInfo.json`: `dps_skill` → `main_skills`, питомцы → `servants`, trait из `hero_full` → `hero_characters`.
- Для стартера важны не только max DPS, а частотность + открытость билдов + питомцев/цена. Rosa 2 доминирует EU по количеству и top DPS, но может быть дорогим late/high-end выбором.
- Hidden builds держать отдельным флагом: они показывают силу героя, но не объясняют механику.

### Ограничения EU-файла

- Это один регион и один снапшот, не вся динамика сезона.
- Топ-5000 уровня 99–100 отражает endgame/late meta, а не обязательно Day1 starter.
- EU может отличаться от ASIA/NA; для китайской меты важнее отдельно разобрать ASIA.
- DPS outliers могут быть нестабильны или зависеть от скрытых/дорогих setup.


## regions_NA_snapshots.json

Кратко: `NA`-файл имеет ту же схему, что EU: 5000 строк leaderboard одного снапшота `HR111401__sc` в режиме `sc`. Используем для сравнения региональной меты, особенно ASIA как прокси китайской/азиатской меты.

### Паспорт

- строк: `5000`
- `ms`: `HR111401__sc`
- `mode`: `sc`
- уровни: `97`–`100`
- max DPS: `156,240,970,017,430`
- hidden builds: `148/5000` (2.96%)

### Топ героев по количеству

- `Sage 1`: 1484 строк (29.68%), hidden 37, max DPS 1,327,969,085,184
- `Rosa 2`: 659 строк (13.18%), hidden 12, max DPS 156,240,970,017,430
- `Thea 1`: 428 строк (8.56%), hidden 4, max DPS 40,167,238,920
- `Erika 1`: 373 строк (7.46%), hidden 4, max DPS 493,372,818,500
- `Rehan 2`: 313 строк (6.26%), hidden 6, max DPS 100,283,602,038
- `Carino 2`: 304 строк (6.08%), hidden 12, max DPS 52,987,094,045
- `Youga 2`: 238 строк (4.76%), hidden 10, max DPS 6,215,404,549
- `Iris 2`: 171 строк (3.42%), hidden 6, max DPS 683,552,903,887
- `Rehan 1`: 168 строк (3.36%), hidden 5, max DPS 21,592,451,634
- `Bing 2`: 115 строк (2.3%), hidden 3, max DPS 3,678,797,684
- `Gemma 2`: 94 строк (1.88%), hidden 7, max DPS 5,963,359,301
- `Erika 3`: 85 строк (1.7%), hidden 5, max DPS 11,686,380,234

### Топ hero + skill по количеству

- `Sage 1 + Chromatic Shot`: 805 строк (16.1%), max DPS 1,327,969,085,184
- `Thea 1 + Chromatic Shot`: 238 строк (4.76%), max DPS 5,161,598,143
- `Erika 1 + Thunder Spike`: 173 строк (3.46%), max DPS 493,372,818,500
- `Sage 1 + UNKNOWN`: 166 строк (3.32%), max DPS 619,211,776,722
- `Rosa 2 + Lightning Shot`: 141 строк (2.82%), max DPS 22,003,439,502,278
- `Rosa 2 + Thunder Spike`: 141 строк (2.82%), max DPS 1,261,226,060,080
- `Sage 1 + Groundshaker`: 120 строк (2.4%), max DPS 46,280,569,035
- `Rosa 2 + Frost Spike`: 112 строк (2.24%), max DPS 522,357,489,926
- `Rehan 2 + Spectral Slash`: 110 строк (2.2%), max DPS 23,287,247,562
- `Rehan 2 + Thunder Spike`: 103 строк (2.06%), max DPS 20,927,103,755
- `Sage 1 + Split Shot`: 95 строк (1.9%), max DPS 841,232,722,679
- `Rehan 1 + Whirlwind`: 89 строк (1.78%), max DPS 21,592,451,634
- `Sage 1 + Frost Impact`: 88 строк (1.76%), max DPS 86,977,652,874
- `Youga 2 + Mind Control`: 87 строк (1.74%), max DPS 6,215,404,549
- `Erika 1 + Lightning Shot`: 84 строк (1.68%), max DPS 12,368,983,641

### Топ hero + skill по max DPS

- `Rosa 2 + Split Shot`: max DPS 156,240,970,017,430, строк 19
- `Rosa 2 + UNKNOWN`: max DPS 87,683,480,685,493, строк 59
- `Rosa 2 + Lightning Shot`: max DPS 22,003,439,502,278, строк 141
- `Rosa 2 + Groundshaker`: max DPS 3,383,726,017,531, строк 3
- `Rosa 2 + Chain Lightning`: max DPS 2,651,569,718,773, строк 27
- `Sage 1 + Chromatic Shot`: max DPS 1,327,969,085,184, строк 805
- `Rosa 2 + Thunder Spike`: max DPS 1,261,226,060,080, строк 141
- `Rosa 2 + Hammer of Ash`: max DPS 1,036,083,169,685, строк 2
- `Rosa 2 + Whirlwind`: max DPS 911,433,556,558, строк 1
- `Gemma 3 + Arrow Einherjar`: max DPS 903,666,869,684, строк 12
- `Sage 1 + Split Shot`: max DPS 841,232,722,679, строк 95
- `Iris 2 + UNKNOWN`: max DPS 683,552,903,887, строк 31

### Топ питомцы

- `Cloudgatherer - Surging Gold`: 2197 (45.29% открытых билдов)
- `Dreamweaver`: 1791 (36.92% открытых билдов)
- `Twilight Sparks`: 1656 (34.14% открытых билдов)
- `Red Umbrella`: 1589 (32.76% открытых билдов)
- `Miss Melancholy - Cyan`: 1438 (29.64% открытых билдов)
- `Elixir Fairies`: 1297 (26.74% открытых билдов)
- `Iron Lion`: 1027 (21.17% открытых билдов)
- `Chalk Spirit`: 1004 (20.7% открытых билдов)
- `Preserver of Eternity`: 917 (18.9% открытых билдов)
- `Happy Chonky`: 873 (18.0% открытых билдов)
- `Avatar of Moon`: 847 (17.46% открытых билдов)
- `Doro-Doro Dorothy`: 841 (17.34% открытых билдов)
- `Azure Gunslinger`: 773 (15.93% открытых билдов)
- `Kitty Express`: 492 (10.14% открытых билдов)
- `Starfish Chanter`: 429 (8.84% открытых билдов)

### Первичные выводы NA

- NA похожа на ASIA по лидерству `Sage 1 + Chromatic Shot`, но Rosa 2 всё равно даёт top DPS.
- Hidden ниже всех: 148/5000, значит NA удобна для извлечения открытых билдов/питомцев.
- Erika 1/Carino 2/Rehan 2 заметнее, чем в ASIA, полезно для western meta сравнения.

## regions_*_snapshots.json

Пока только первичная заметка: это региональные снапшоты статистики по ASIA/EU/NA. Следующий шаг — отдельно задокументировать их схему и связать с reference.
- `regions_ASIA_snapshots.json`: top-level `list`; preview `5000`
- `regions_EU_snapshots.json`: top-level `list`; preview `5000`
- `regions_NA_snapshots.json`: top-level `list`; preview `5000`

## enriched_regions_summary.json

Промежуточный агрегированный файл для будущего HTML. Создан из `regions_ASIA/EU/NA_snapshots.json` + `reference_tldbInfo.json`.

### Что внутри

- `schema`: `tli-fullstat-enriched-summary-v1`
- `regions.ASIA/EU/NA`: агрегаты по каждому региону.
- `top_heroes`: герой, count, share_pct, hidden, max_dps, median_dps.
- `top_combos_by_count`: hero + нормализованный skill (`dps_skill` → `skill` → `UNKNOWN`), count, share, hidden, max/median DPS, skill_icon, skill_tags.
- `top_combos_by_max_dps`: те же связки, но сортировка по max DPS.
- `top_servants`: servant, count, share_open_pct, icon, rarity, type.
- `top_rows_by_dps`: топовые строки с player, hero_full, trait, lvl, dps, skill/dps_skill, питомцы, skill_icon/hero_icon.
- `global.hero_counts_matrix`: матрица популярности героев по регионам.

### Нормализация

- `hero_full`: пробелы и `|` приводятся к виду `Base Hero | Trait`.
- `trait`: извлекается из `hero_full` после `|`.
- `skill_key`: `dps_skill`, если есть; иначе `skill`; иначе `UNKNOWN`.
- Join с TLIDB: `skill_key` ищется в `records.main_skills`, питомцы — в `records.servants`, trait — в `records.hero_characters`.

### Зачем нужен

Этот файл — база для финального HTML, чтобы не грузить в браузер все 15k сырых строк и не делать тяжёлый join на клиенте.

## final.html

Новый чистый интерактивный HTML-отчёт, не завязанный на старый `tli-lunaria-report.html`. Первая версия сфокусирована на EU, но структура подготовлена под ASIA/NA и будущие вкладки.

### Что уже есть

- KPI по EU: rows, hidden, level range, max DPS, top hero.
- `Топ героев EU`: поиск + сортировка по count/max DPS/median DPS/hidden.
- `Топ питомцы EU`: поиск, иконки из TLIDB reference, rarity/type.
- `Hero + Skill связки EU`: переключение count/max DPS, поиск по hero/skill/tag, TLIDB skill icons/tags.
- `Top rows by DPS EU`: поиск по player/hero/skill/servant, фильтр hidden/open.
- Вкладка `План доработок` с идеями следующих шагов.


### Обновление final.html — regional/compare/starter

Добавлено:
- `Region meta` с переключателем EU/ASIA/NA.
- `Region compare`: матрицы популярности героев и skills по регионам.
- `Starter candidates`: первичный score кандидатов на основе popularity, region consistency, max/median DPS и hidden penalty.
- В `enriched_regions_summary.json` добавлены `starterCandidates`, `compare.hero_counts_matrix`, `compare.skill_counts_matrix`.

Текущий starter score — прототип для аналитики, не финальная SS13-рекомендация. Следующий слой: patch modifier, streamer/china watchlist и servant cost risk.

### Почему с нуля

Старый HTML был тяжёлым и исторически нарастал патчами. Новый `final.html` использует подготовленный `enriched_regions_summary.json`, поэтому его проще расширять и меньше риска сломать старые блоки.


### Обновление final.html — SS13 patch impact

Добавлено:
- `patch_modifiers.json`: ручная структурированная выжимка из локального HTML патчноутов.
- В `enriched_regions_summary.json`: `patchImpact`, `starterCandidatesPatchSorted`, поля `patch_adjustment`, `patch_score`, `patch_notes` у starter candidates.
- В `final.html`: вкладка `Patch impact` и сортировка starter candidates по `patch score` / `base score`.

Главные patch-сигналы:
- Rehan 2 / Seething Silhouette: сильный баф (`+48%` → `+78%`, снят штраф attack speed).
- Sage 1 / Licorice Note: пакет нерфов, высокий риск для статистического лидера Chromatic Shot.
- Chromatic Shot: переработка/нерф конверсии и base damage; Splendor удалён.
- Rosa 2 / Unsullied Blade: снижены caps/бонусы, плюс фикс Silver Domain; late power остаётся, но риск ниже score.
- Iris/Magus: mixed/nerf, Rock Magus base damage cut.
- Selena/Terra/Tangle и Synthetic/Bond/minions: новые позитивные watchlist-сигналы, но не представлены в Lunaria stats.


### Обновление final.html — риск питомцы и вердикт

Добавлено:
- Русские названия вкладок и основных заголовков.
- Вкладка `Риск питомцы / рынка`: глобальный топ питомцы и риск по каждому кандидату.
- Вкладка `Вердикт по стартерам`: итог `Старт / Можно, но проверить / Watchlist / Осторожно / не первым`.
- В `enriched_regions_summary.json` добавлены `servantRiskMeta`, `starterVerdicts`, `servant_risk`, `outlier_risk`, `verdict`, `verdict_reasons`.

Важно: текущий риск питомцы получился высоким почти у всех топ-кандидатов, потому что meta-питомцы сильно пересекаются между билдами. Это полезно как предупреждение по рынку, но позже можно сделать более мягкую формулу.


### Обновление final.html — режим без питомцы

Добавлено:
- Вкладка `Без питомцы`: вердикт без учёта market/servant risk.
- В `enriched_regions_summary.json`: `starterVerdictsNoServants`, `score_no_servants`, `verdict_no_servants`, `verdict_no_servants_reasons`.
- В этом режиме питомцы показываются только справочно; решение строится на статистике, патче, hidden/base risk и outlier risk.

Нюанс: даже без питомцы многие топы остаются `Watchlist`, потому что их держит `outlier risk` — max DPS сильно выше median DPS или мало строк.


### Обновление final.html — мягкий режим и пояснения

Добавлено:
- Вкладка `Мягкий режим`: без штрафа питомцы и без блокировки outlier. Outlier/питомцы показываются только как предупреждения.
- Мини-пояснения на вкладках `Вердикт`, `Без питомцы`, `Мягкий режим`.
- В `enriched_regions_summary.json`: `starterVerdictsSoft`, `score_soft`, `verdict_soft`, `verdict_soft_reasons`, `verdictModeExplanations`.

Смысл режимов:
- `Вердикт`: строгий практический режим — статистика + патч + питомцы + outlier.
- `Без питомцы`: убирает экономический риск питомцы, но оставляет outlier risk.
- `Мягкий режим`: показывает чистый потенциал статистики/патча, outlier и питомцы только предупреждают.


### Обновление final.html — вкладка `Герои`, Moto 2

Добавлено:
- Вкладка `Герои` с первым досье: `Commander Moto | Charge Calling / Moto 2`.
- Динамика по дням/неделям из старого `tmp/tli-report/report-data.json` по trait `Charge Calling`.
- Региональная fullStat-статистика ASIA/EU/NA по `Moto 2`.
- Топ skills, топ питомцы, топ строки по DPS, patch-сигналы.

Ограничение: fullStat — один late snapshot, без дней; динамика по дням берётся из старого Lunaria report-data, где Moto 2 представлен как `Charge Calling`.


### Обновление final.html — глобальные мини-иконки

Добавлено:
- `icons` map в `enriched_regions_summary.json` из `reference_tldbInfo.json`: main skills, support skills, питомцы, hero portraits.
- JS helpers в `final.html`: `skillMini`, `servantMini`, `heroMini`.
- Мини-иконки добавлены в основные таблицы: герои, skill-связки, starter candidates, питомцы, top rows, Moto 2 skills/питомцев/timeline.

Покрытие текущих видимых данных:
- Skills: 60/63. Не найдены в reference: `Barrier Burst`, `True Flame Immolation`, `UNKNOWN`.
- Питомцы: 89/89.
- Heroes: 22/22.


### Обновление final.html — источник данных и сезон

Добавлено в шапку отчёта:
- Источник статистики: leaderboard/stat snapshots ASIA/EU/NA, скачанные пользователем.
- Справочник/иконки: TLIDB (`https://tlidb.com/`), schema `tlidb-reference-v1`.
- Сезон статистики: `lunaria`.
- Snapshot/mode: `HR111401__sc`, `sc`.
- Отдельная оговорка: SS13 patch notes используются только как прогнозный modifier, не как источник leaderboard-статистики.

Это важно для публикации через GitHub Pages, чтобы читатели понимали происхождение данных.


### Публикация GitHub Pages

Подготовлена папка сайта:
- `fullStat/site/index.html` — копия текущего `final.html`
- `fullStat/site/README.md`

Репозиторий:
- `https://github.com/D1msn/tli-fullstat`
- branch: `main`
- commit: `245d293 Publish TLI fullstat dashboard`

Статус:
- Код успешно запушен в `main`.
- GitHub Pages через API включить не удалось: `403 Resource not accessible by personal access token`.
- Нужно вручную включить Pages: GitHub → repository Settings → Pages → Build and deployment → Deploy from branch → branch `main`, folder `/root` → Save.
- После включения URL должен быть: `https://d1msn.github.io/tli-fullstat/`

Обновление отчёта после правок:
1. Скопировать `fullStat/final.html` в `fullStat/site/index.html`.
2. `cd fullStat/site`
3. `git add index.html README.md && git commit -m "Update dashboard" && git push`


### Обновление публикации — attribution

Из `final.html` удалён отдельный блок `Источник данных и сезон`.
В шапке отчёта теперь коротко указано:
- часть статистики взята с `https://tliherorank.com/`;
- иконки и справочник — TLIDB (`https://tlidb.com/`);
- сезон статистики: Lunaria;
- SS13 patch notes используются как прогнозный слой.

Изменение запушено в GitHub Pages repo коммитом `7639515 Update source attribution`.

## TODO

- [x] Документировать схему `regions_*_snapshots.json` для EU/ASIA/NA.
- [x] Сделать базовый нормализатор названий skills/питомцы/hero traits в `enriched_regions_summary.json`.
- [ ] Пересобрать HTML-отчёт на enriched данных из fullStat.
- [ ] Добавить вкладки: regional meta, ASIA watchlist, питомцы meta, hidden/open builds.

### Обновление final.html — Rehan 2

- Добавлена вкладка `Rehan 2`: региональная статистика, топ питомцев, популярные DPS-скиллы, популярные полные доступные связки и топ строки по DPS.
- В `enriched_regions_summary.json` добавлен `heroDossiers.rehan2`.
- Важно: в исходных leaderboard-строках нет support-скиллов и экипировки. Поэтому полная доступная связка сейчас = DPS skill + 6 питомцев, сгруппированных без учёта порядка.
- Rehan 2 найден в 832 строках fullStat: 795 открытых и 37 скрытых.
