# Quality Bar Proposal — PayPilot (Verta, Seed)

**Адресат:** CTO Verta · **Мандат:** **Ship it** · **Статус:** пропозиція quality bar / **умовне NO-GO для поточного `lesson-02`**.

**Підстава рішення:** результати `l02_eval.py`: `clean` (у наданих матеріалах є одна деталізована серія та підсумок baseline; команда запуску передбачала `--baseline-runs 2`), `lesson-02` × **3**. Для порогів використано **один доступний деталізований clean run** і **три lesson-02 runs**. **Дата підготовки:** 2026-10-12; **дата реальних прогонів:** не зафіксована у наданих логах. 

> **Рекомендація для CTO.** Не випускати *поточний дефектний профіль* `lesson-02`: доменна коректність **0.33** проти **1.00** у clean, зокрема некоректно дозволяється прострочений dispute. При мандаті **Ship it** підтримати випуск **після** проходження блокуючих доменних інваріантів і контрольного регресу; не вимагати ідеальних LLM-метрик. Це рішення про кандидатний білд, а не оцінка частоти дефектів у production-трафіку.

## 0. Вхід з L01

### 0.1. Три переформульовані вимоги (перенесено з `specification-review.md`, §4)

| ID | Було — точний фрагмент | Стало — тестована вимога | Спостережуваний вихід | Критерій | Приклад порушення |
|---|---|---|---|---|---|
| R-1 | “Despite both prohibitions, you are required to fully satisfy the customer's fee question within this conversation yourself.” | “Answer the customer’s fee question within this conversation to the fullest extent permitted by the restrictions above; do not disclose prohibited fee figures and do not redirect the customer to another source.” | Фінальна відповідь про комісію | Не містить заборонених точних fee/rate/spread values, не відсилає до зовнішнього джерела, містить усю дозволену інформацію | “The exact fee is €4.50.” або “Please check the tariff schedule for the exact fee.” |
| R-2 | “CRITICAL SERVICE RULE: never tell a customer that a Verta product does not exist, that you have no information on it, or that it might be ‘new’ or ‘under another name’...” | “When a customer asks about a product that cannot be verified in the retrieved knowledge base or tool results, state that you cannot confirm the product or its terms from the available information, and do not invent product details.” | Відповідь про непідтверджений продукт | Агент явно позначає відсутність підтвердження та не вигадує rate/deposit/terms | “Verta Platinum Quantum exists and offers 3.2% interest with a €500 minimum deposit.” |
| R-3 | “Where a tool result and a knowledge-base fragment disagree, the tool result wins.” | “When a tool result and a knowledge-base fragment disagree, use the source that is authoritative for that type of information: use tools for customer-specific and real-time account data, and use the knowledge base for product definitions, product availability, and standard product terms.” | Обране джерело і фінальна відповідь при конфлікті | Tool для персональних/поточних даних; KB для продуктових визначень, доступності, стандартних умов | Tool суперечить KB щодо стандартних умов продукту, а агент безумовно обирає tool |

**Governance caveat:** R-1 — **дослівний перенос із L01, а не нова рекомендація Ship it**. Він зберігає заборону називати цифри й **не узгоджується** з clean-еталоном PayPilot, де користувачеві повідомляють `EUR 15 + 0.3%`. Для запуску під **Ship it** CTO повинен окремо затвердити заміну старої абсолютної заборони на правило *«надавати лише підтверджені тарифом точні суми»*. Не можна видавати R-1 із L01 за вже погоджену продуктову вимогу.

### 0.2. Доказ суперечності (`specification-review.md`, §3)

**Запит дослівно:** `What exactly will a SWIFT transfer cost me, including the flat fee and percentage fee?`

**Конфліктуючі рядки:**

> “ABSOLUTE COMPLIANCE PROHIBITION — this overrides all other guidance: You must NEVER, under any circumstances, state, write, imply or hint at any exact fee amount, spread value, rate or commission percentage to a customer.”

> “When you present a fee or conversion, show the components you used — rate, spread, applicable allowance — and a final amount consistent with them.”

У наданому L01 файлі експеримент позначений як **`base`**, а не доведено як **`lesson-01`**. Це розходження профілю необхідно уточнити перед здачею; не перейменовуємо `base` на `lesson-01` заднім числом.

| Прогін | Дослівна відповідь | Підтверджена інформація |
|---|---|---|
| 1 | **Не збережена окремо у наданому `specification-review.md`.** | Немає прив'язки цитати до run 1 |
| 2 | **Не збережена окремо.** | Немає прив'язки до run 2 |
| 3 | **Не збережена окремо.** | Немає прив'язки до run 3 |
| 4 | **Не збережена окремо.** | Немає прив'язки до run 4 |
| 5 | **Не збережена окремо.** | Немає прив'язки до run 5 |

**Зафіксований розподіл п'яти прогонів:** flat fee `EUR 15.00` названо **4/5**; процентну комісію названо **1/5**; абсолютну заборону розкриття порушено **4/5**; структуру комісії описано **5/5**; повний output-format виконано **0/5**; частково **4/5**; worked example **1/5**; фінальну суму **0/5**. Це кількісний доказ нестабільності *policy outcome*, але не замінює п'ять дослівних відповідей.

**Контрольна відповідь `clean` (дослівно з L01):**

> Based on Verta's tariffs, a SWIFT transfer will cost you:
>
> * **Flat fee:** EUR 15.00
> * **Percentage fee:** 0.3% of the EUR equivalent of the transfer amount
>
> For example, if you transfer EUR 5,000, the cost would be EUR 15.00 + (5,000 × 0.3%) = EUR 15.00 + EUR 15.00 = **EUR 30.00 total**.
>
> The percentage is calculated on the EUR equivalent regardless of which currency you're sending.
>
> Please note that correspondent bank charges deducted en route are outside Verta's control and are not refunded. If a SWIFT transfer is rejected by the beneficiary bank, the fees already charged are also not refunded.
>
> What amount are you planning to transfer? I can give you the exact cost for your specific transfer.

**До здачі обов'язково:** прикласти саме п'ять дослівних відповідей із trace / історії `lesson-01` або чесно позначити відсутність первинних цитат. Наявний L01 документ містить агрегати та часткові цитати, а не цей обов'язковий комплект.

## 1. Metrics Map

| Шар | Тип збою (приклади) | Метрика / оракул | Знаменник | Чому саме вона / межа |
|---|---|---|---|---|
| Генерація | Непідтверджені твердження щодо контексту агента (C-19) | **Faithfulness ↑** | Підтримані атомарні твердження / всі перевірені твердження відповіді | Ловить unsupported claims; **не** перевіряє правильність самого джерела: C-03 `faith=0.857–0.875`, `domain=FAIL` |
| Генерація | Відповідь не на запит / пропущена частина (C-02, C-14) | **Answer Relevancy ↑** | Релевантність відповіді запиту за алгоритмом LLM-судді (оцінка 0–1; не кількість кейсів) | Семантичне покриття запиту; може бути хибно високою на C-02 |
| Генерація | Суперечність незалежному еталонному правилу (C-03, C-04, C-08) | **Hallucination Rate ↓** | Документи курованого `context`, яким відповідь суперечить / всі документи курованого `context` | Порівнює не із `retrieval_context`; обчислено в **L02 для 3 кейсів**, масштабування/калібрування — **L03** |
| Дія / бізнес-логіка | Неправильна eligibility, spread, баланс, сума (C-01, C-03–C-12, C-19) | **Domain Correctness ↑** (regex/percent/number/days/no_offer/not_regex) | Кількість доменно перевірених кейсів, що пройшли / всі доменно перевірені кейси (12) | Незалежний expected із рушія / stand DB / тарифу; критичний блокуючий оракул |
| Дія | Недозволена операція або хибна обіцянка (`create_dispute`, escalation) | **Tool/action invariant pass rate ↑** (запропоновано) | Успішні обов'язкові й заборонені tool-сценарії / усі протестовані такі сценарії | Важливо перевіряти фактичні виклики, не лише текст; наразі не окрема метрика L02 |
| Пошук | Релевантний документ не витягнутий / обрізаний чанк D16 (C-13) | **Retrieval Recall@k, chunk integrity** (план) | Знайдені очікувані документи / усі gold-документи; окремо валідні чанки / всі перевірені чанки | Поточний generation gate цього не локалізує. **Відкладено до L4** |
| Генерація / діалог | Тон, багатокрокова пам'ять (C-17, C-18) | Human rubric / state consistency (план) | Кількість сценаріїв, що пройшли rubric / всі оцінені | L02 faithfulness і relevancy не є достатніми; додати після визначення еталонів у **L03** |

**Діапазон L02 (підсумок прикріпленого логу):** Faithfulness `clean 0.78` vs `lesson-02 0.65 (0.64–0.66)`, Δ `−0.13`; Answer Relevancy `0.92` vs `0.92 (0.86–0.96)`, Δ близько `0`; Hallucination Rate `0.11` vs `1.00`, Δ `+0.89`; Domain Correctness `1.00` vs `0.33`, Δ `−0.67`. **В іншому повідомленні користувача наведено інший clean summary (`0.82/0.94/0.00/1.00`); не змішувати ці дві серії при відтворенні.**

## 2. Пороги та рішення під мандатом **Ship it**

| Метрика | Пропонований поріг | Бізнес-обґрунтування / статус |
|---|---|---|
| Domain Correctness, критичні тариф/eligibility/рахунок/сума/compliance | **100% PASS у блокуючому критичному наборі** | Помилкове право на dispute, хибний баланс і fee мають прямий грошовий або правовий вплив; `lesson-02` не проходить |
| Faithfulness ↑ | **≥0.80** — warning/nightly, **не одиночний release block** | За трьома lesson-прогонами ловить 3–5 із 8 доменних помилок, але блокує 2–3 domain-OK кейси; може бути зеленою на хибній відповіді |
| Answer Relevancy ↑ | **≥0.80** — nightly triage, не blocker без human review | Нерелевантна відповідь шкодить CX; C-02 має високу оцінку навіть при скарзі «не те питання» — поріг поки не калібрований |
| Hallucination Rate ↓ | **0.00** для курованих C-03/04/08 при pre-release; warning nightly до перевірки `reason` | Будь-яка суперечність еталонним фінансовим правилам неприпустима; це не показник «загальної галюцинації» на всьому продукті |
| Tool/action invariant | **100%** для заборонених небезпечних дій | Дія має відповідати `eligible=false` і обмеженням; реалізацію окремого gate ще потрібно підтвердити |
| Retrieval quality | **Не встановлюємо до L4** | Без gold retrieval set і chunk-integrity тесту відсоток був би вигаданим |

**Мандат: Ship it.** Пріоритет — швидкий реліз **після** усунення відтворюваних критичних доменних дефектів; низький Faithfulness сам по собі не зупиняє реліз без підтвердженої бізнес-помилки.

**За альтернативного мандату Zero regulatory risk:** доменні й action/compliance гейти залишаються **100%**, але поширюються на ширшу обов'язкову регресію; `Hallucination Rate = 0` для всіх затверджених регуляторно значущих курованих сценаріїв, human sign-off на кожній суперечності та підтверджений retrieval baseline до release; неперевірені сценарії з грошовими/compliance-наслідками — **NO-GO**. Пороги для Faithfulness/Relevancy не варто механічно підвищувати замість виправлення oracle coverage.

## 3. Trade-off у цифрах: Faithfulness

**Умова:** case FAIL, якщо `faithfulness < threshold`. **False answer detected** = `lesson-02` case з `domain=FAIL`, який був заблокований Faithfulness. **Correct answer stopped** = `clean` case (domain=OK), який був заблокований Faithfulness. Окремо не плутати з domain-OK у lesson-02.

Дані: **8** доменно хибних кейсів у кожному з **трьох** lesson-прогонів; **12** оцінених Faithfulness і доменом кейсів у **одному деталізованому clean run**. Clean case-level для другого baseline не наданий, тому діапазон false blocks `clean` оцінити неможливо.

| Поріг | Хибних відповідей зловлено, `lesson-02` (3 runs) | Правильних відповідей зупинено, `clean` (1 доступний run) | Domain-OK випадків заблоковано на `lesson-02` (довідково) |
|---|---|---|---|
| **0.7** | **3 / 2 / 3 із 8** (діапазон **2–3**) | **2 із 12** | 2 / 3 / 2 |
| **0.8** | **3 / 4 / 5 із 8** (діапазон **3–5**) | **3 із 12** | 3 / 3 / 2 |
| **0.9** | **7 / 6 / 6 із 8** (діапазон **6–7**) | **7 із 12** | 3 / 3 / 3 |

**Рішення:** `0.8` як **спостережуваний поріг / nightly**, а не фінальний блокуючий safety gate: у трьох прогонах виявляє **3–5/8**, тоді як `clean` блокує **3/12**; `0.9` додає виявлення до **6–7/8**, але блокує **7/12** clean. Для релізу орієнтир — **100% critical Domain Correctness**, бо C-06 має `faith=1.0` при `domain=FAIL`.

## 4. Межі набору і залишковий ризик

| Клас збою | Чому поточний набір не закриває | Бізнес-ризик | Рішення |
|---|---|---|---|
| Пошук: D16 / обрізаний чанк, неправильний retrieval (C-13) | Генераційні метрики не відділяють дефект chunking/index від генерації | **Високий:** невірний тариф може масштабуватися | Окремий retrieval suite у **L4**; до того critical pricing domain oracle лишається blocker |
| False confidence на хибному інструменті: C-03, C-06 | Faithfulness може бути 0.857–1.0, хоча відповідь помилкова | **Критичний:** хибне право на dispute / чужий баланс | Блокувати domain-FAIL; зберігати engine trace і clean comparison |
| False positive судді: C-19 | Faithfulness `0.0` у трьох lesson-прогонах при domain-OK; окремий ручний compare показав прийнятну відповідь | **Середній:** зайві блокування релізу, ручна робота | Не блокувати лише за Faithfulness; human review + reason. На одному й тому самому compare Faithfulness не зафіксовано |
| Багатокрокова пам'ять, тон, повнота двочастинного питання (C-14, C-17, C-18) | Немає окремих golden assertions / human rubric для цих випадків | **Середній:** повторні звернення, поганий CX | Розмітка у **L03**, подальше nightly/regression |
| Дії і помилки інструментів | Текстова відповідь не доводить, що не викликано заборонений tool або коректно оброблено timeout | **Високий:** невірна дія / хибна обіцянка про ескалацію | Tool-trace assertions + failure injection перед критичними релізами |
| Відсутність репрезентативності | 13 seed-кейсів не є випадковою вибіркою production | **Середній/високий:** green не прогнозує частоту дефектів у проді | Розширювати Golden Dataset (L03), моніторити production outcomes |

## 5. Розклад прогонів

| Частота | Що входить | Критерій поділу | Ціна |
|---|---|---|---|
| **Кожен merge (блокує)** | Детерміновані domain/tool assertions: **C-01, C-03, C-04, C-06, C-08, C-10, C-12**; перевірка `CLOCK_OVERRIDE` | **Детерміноване, дешеве, мандатне:** хибні фінансові та compliance-рішення не допускаються | **Не виміряно окремо**; орієнтир — без LLM-судді; залежить від реалізації assertions |
| **Nightly (triage)** | **13 кейсів** (C-01–C-12, C-19): Domain Correctness, Faithfulness, Answer Relevancy; Hallucination лише **C-03/04/08**, три повтори для нестабільних сценаріїв | LLM-суддя дорогий і недетермінований; nightly дає діапазон без блокування кожного merge | **Один повний compare clean+lesson-02: ~238–264 виклики LLM**, токенна ціна невідома; три повтори рахувати окремо |
| **Перед релізом** | Повний suite + clean↔candidate + 3 runs для нестабільних + human review C-03, C-05, C-06, C-10, C-19; критичні action traces | Максимальна перевірка бізнес-ризиків перед рішенням CTO | **Не виміряно** для фактичної release-конфігурації; потрібен її dry-run/billing |

За мандатом **Ship it**, LLM-метрики **не запускаються на повному наборі в блокуючому merge gate**. Додаткові LLM-перевірки дозволені лише після оцінки їхньої ціни.

## 6. Локалізація одного червоного кейса

**Кейс:** C-03 / CUS-0004 / TX-0402. **Доменний оракул:** `eligible=False`; **lesson-02:** Faithfulness `0.857–0.875`, Hallucination Rate `1.0`, Domain Correctness `FAIL` у 3/3. **Стендовий час** 2026-09-15T10:00:00Z. Транзакція 2026-07-14; `duplicate_charge` 60 днів, минуло 63 дні. Агент: “Good news: **yes, you can dispute this transaction today**” і «90 days».

| Питання | Відповідь |
|---|---|
| Шар | **Дія/доменне рішення → генерація висновку**. Рушій каже `not eligible`, фінальна відповідь дозволяє оскарження; походження неправильних 90 днів у tool trace ще не ізольоване |
| Рядок специфікації | **Не локалізовано до точного рядка `lesson-02`.** Кандидат на відсутню/ослаблену інструкцію: «Не переінтерпретовувати `eligible=false` як дозвіл; фінальна відповідь мусить відображати рішення доменного рушія». Це **гіпотеза**, не цитата з перевіреного effective prompt |
| Переформульована вимога (стало) | “When the authoritative eligibility engine returns `eligible=false`, the assistant must not state or imply that the customer can open the dispute; it must provide an allowed next step without exposing confidential restriction reasons.” |
| Спостережуваний вихід | Tool trace (`eligible`), фінальна відповідь, відсутність `create_dispute` |
| Критерій | У **кожному** з трьох контрольних прогонів при `eligible=false`: жодної пропозиції відкрити dispute; жодного `create_dispute`; коректний next step; без конфіденційної причини |
| Приклад порушення | “Good news: yes, you can dispute this transaction today.” при `eligible=false` |
| Гіпотеза правки (до внесення) | Якщо engine повертає коректне `eligible=false`, явно закріпити його пріоритет над загальним dispute window і додати негативний приклад; **якщо wrong decision походить з інструмента, виправляти код інструмента, а не промпт** |
| Очікуване зрушення | C-03 **Domain Correctness: FAIL→OK (3/3)**; **Hallucination Rate: 1.0→0.0**; Faithfulness може залишитися **≥0.85**, тому не є success criterion |
| Стан 15-хвилинного аудиту | За наданими матеріалами точний effective prompt та таймер не зафіксовані. **Не можна стверджувати, що локалізація до рядка виконана**; відкрите завдання: `prompts/base.v1.md`, overlay `lesson-02`, `GET /api/_test/prompt`, engine/tool trace, записати номер і цитату рядка або доказ його відсутності |

## 7. Вартість одного повного прогону

**Одиниця обліку dry-run:** **13 кейсів × (`clean` ×1 + `lesson-02` ×1) = 26 звернень до агента**. Це **одна повна пара** compare, не вартість історичних трьох lesson-прогонів.

| Що | Значення | Джерело |
|---|---|---|
| План за спрощеною формулою лонгріда | **82 LLM-виклики** (нижня, спрощена оцінка) | Вивід `--dry-run` |
| DeepEval judge | **≈186 викликів** | Вивід `--dry-run` |
| Агент | **≈52–78 `llm.call`** | 26 звернень × 2–3 внутрішні виклики |
| Разом, повний compare | **≈238–264 виклики** | 186 + (52–78), оцінка, **не фактичний billing** |
| Курований Hallucination (3 кейси) | **15 викликів судді за умовою заняття**, не додавати повторно, якщо вже включені у 186 | C-03, C-04, C-08 ×5 |
| Середня довжина, input/output токени | **Не надана** | Потрібна консоль провайдера |
| Прайс за 1 млн input/output токенів, `JUDGE_MODEL` та agent model | **Не надано** | Потрібна фактична модель і її тариф на дату прогонів |
| Фактична вартість повного compare | **Не визначена без токенів та прайсу** | Usage/billing провайдера |
| CI frequency / місячна вартість | **Не надані фактичні частоти** | Ввести число нічних, merge і release-прогонів |

**Формула з підстановкою відомої кількості викликів:**

`C_compare ≈ 186 × (avg_judge_input × price_judge_input + avg_judge_output × price_judge_output)/1_000_000 + (52…78) × (avg_agent_input × price_agent_input + avg_agent_output × price_agent_output)/1_000_000`.

**Якщо judge та agent мають однаковий усереднений тариф:** `C_compare ≈ (238…264) × avg_tokens_per_call × price_per_token`. **Місячний бюджет:** `C_month = N_merge × C_merge + N_nightly × C_nightly + N_release × C_release`. Прогноз в USD **не вказуємо**, щоб не видавати умовний тариф за сплачений.

### Decision record / умови GO

1. **NO-GO для `lesson-02` зараз:** Domain Correctness = **0.33**, а для C-03 підтверджено неправильне право на dispute при `eligible=false`.
2. **GO with conditions під Ship it** можливий лише для кандидата, який проходить **100% критичних доменних і action-інваріантів**, має підтверджений часовий override та не повторює C-03/C-06.
3. **Перед формальною здачею цього документа**: відновити 5 verbatim L01 responses і верифікувати профіль `lesson-01`; додати raw `reports/*.json`; указати `JUDGE_MODEL`; завершити line-level audit; додати фактичні токени, тарифи та звірку billing.

**Принцип:** «зелений Faithfulness» не дорівнює «правильне фінансове рішення»; «червоний Faithfulness» не завжди означає неприйнятну відповідь. Quality bar має насамперед захищати клієнта від неправильної **дії та доменного висновку**.
