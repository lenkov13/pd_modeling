# pd_modeling
Implementation of an automated solution that uses market data to generate a PD curve for assessing the credit risks of emitents

Основные понятия
PD - 
LGD - 
EL - 

# Основные термины

**PD Modeling** (Probability of Default Modeling) — это моделирование вероятности дефолта заемщика, используемое для оценки кредитного риска в банках и финансовых институтах. Методология применяется в рамках требований IFRS 9 (International Financial Reporting Standard 9) и регуляторных требований Basel III.[1][2][3]

## Основные параметры кредитного риска

### PD (Probability of Default)

**Вероятность дефолта** — это вероятность того, что заемщик не выполнит свои долговые обязательства в течение определенного периода времени (обычно 12 месяцев или lifetime).[4][1]

- **Point-in-Time (PIT) PD**: Оценка на конкретный момент времени с учетом текущих экономических условий (требуется по IFRS 9)[1]
- **Through-the-Cycle (TTC) PD**: Средняя вероятность через экономический цикл (используется в Basel IRB подходе)[1]
- **Basel III PD floor**: Минимальная PD установлена на уровне 5 базисных пунктов (0.05%)[2]

### LGD (Loss Given Default)

**Потери при дефолте** — доля непогашенной задолженности, которую кредитор потеряет в случае дефолта заемщика. Измеряется в процентах от EAD.[5][4][1]

$$ LGD = 1 - RR $$

где $$RR$$ — Recovery Rate (коэффициент восстановления).[5]

**Downturn LGD**: Оценка потерь в неблагоприятных экономических условиях (требуется по Basel).[1]

### EAD (Exposure at Default)

**Задолженность на момент дефолта** — сумма, которую заемщик должен кредитору в момент дефолта. Для кредитных линий учитывается вероятность дополнительного использования лимита до дефолта.[4][1]

### RR (Recovery Rate)

**Коэффициент восстановления** — доля задолженности, которую кредитор сможет вернуть после дефолта через реализацию залога, реструктуризацию или банкротную процедуру. Типичное значение для корпоративных облигаций — 40%.[5]

$$ RR = 1 - LGD $$

**Систематический риск восстановления**: Исследования показывают, что RR коррелирует с PD — в периоды экономических спадов растут дефолты и одновременно падают recovery rates.[6][5]

## Основная формула Expected Credit Loss

### Формула ECL по IFRS 9

$$ ECL = PD \times LGD \times EAD $$

где:
- $$ECL$$ — Expected Credit Loss (ожидаемые кредитные потери)[4][1]
- $$PD$$ — вероятность дефолта за период (12 месяцев или lifetime)[1]
- $$LGD$$ — потери при дефолте (%)[1]
- $$EAD$$ — задолженность на момент дефолта[1]

### Полная формула с учетом вероятностей

$$ ECL = PD \times LGD \times EAD + (1 - PD) \times 0 \times EAD $$

Это probability-weighted формула, учитывающая два сценария: дефолт (с вероятностью PD) и отсутствие дефолта (с вероятностью 1-PD).[7][4]

### Пример расчета

Заемщик должен 1,000,000 руб. с вероятностью дефолта 20% и ожидаемыми потерями при дефолте 70%:[4]

$$ ECL = 0.20 \times 0.70 \times 1,000,000 = 140,000 \text{ руб.} $$

## Связь с интенсивностью дефолта

В reduced-form моделях используется **hazard rate** (интенсивность дефолта) $$\lambda(t)$$:[1]

$$ S(t) = e^{-\int_0^t \lambda(u) du} $$

где $$S(t)$$ — survival probability (вероятность выживания до момента $$t$$).[1]

Безусловная вероятность дефолта:

$$ PD(t) = 1 - S(t) $$

## Трехстадийная модель IFRS 9

**Stage 1**: Нормальное качество кредита — провизии на основе 12-месячной ECL.[8][9]

**Stage 2**: Существенное ухудшение кредитного качества (SICR — Significant Increase in Credit Risk) — провизии на основе lifetime ECL.[9][8]

**Stage 3**: Дефолт или credit-impaired — провизии на основе lifetime ECL с учетом фактических потерь.[8][9]

## Методы оценки PD

### Probit/Logit модели

Бинарная регрессия для оценки вероятности дефолта:[3]

$$ P(Y = 1) = \Phi(\beta' X) $$

где $$\Phi$$ — функция стандартного нормального распределения (для probit), $$X$$ — вектор финансовых показателей заемщика.[3]

### Определение дефолта

По Basel II/III: просрочка платежа более **90 дней** или случай официальной ликвидации/банкротства.[3]

## Ключевые источники и стандарты

### IFRS 9 (2014, вступил в силу 2018)

Международный стандарт финансовой отчетности, требующий:
- Forward-looking оценку кредитных потерь[1]
- Point-in-time параметры PD, LGD, EAD[1]
- Трехстадийную модель обесценения[8]

**Источник**: IASB — International Accounting Standards Board[1]

### Basel III (Basel Committee on Banking Supervision)

Регуляторные требования к банковскому капиталу:
- IRB approach (Internal Ratings-Based) для расчета RWA (Risk-Weighted Assets)[2]
- Through-the-cycle PD и downturn LGD[1]
- PD input floor: 5 bps с 2023 года[2]

**Источники**:
- Basel Committee on Banking Supervision (2006). "International Convergence of Capital Measurement and Capital Standards"[1]
- Basel III reforms (2017)[2]

### Академические источники

- **Altman, Resti, Sironi** (2004). "Default Recovery Rates in Credit Risk Modeling: A Review of the Literature and Empirical Evidence"[5]
- **Düllmann & Trapp** (2004). "Systematic Risk in Recovery Rates"[6]
- **Schönbucher** (2003). Credit risk modeling — описание интенсивности дефолта
- **Duffie** (1998). Credit triangle formula для связи спреда, PD и recovery rate

## Дополнительные ресурсы

- **GARP** (Global Association of Risk Professionals) — white papers по IFRS 9 ECL calculation[1]
- **KPMG, PwC** — практические руководства по внедрению IFRS 9[10][7]
- **Bank of Russia Working Papers** — исследования PD моделей для российского рынка[3]

***

*Эта заметка составлена для образовательных целей и охватывает основные концепции PD modeling в контексте IFRS 9 и Basel III требований.*

[1](https://www.garp.org/hubfs/Whitepapers/a1Z1W0000054wttUAA.pdf)
[2](https://www.garp.org/risk-intelligence/credit/basel-iii-the-impact-of-the-new-probability-of-default-input-floor)
[3](https://www.cbr.ru/StaticHtml/File/116468/wp-66_e.pdf)
[4](https://www.cpdbox.com/ifrs9-ecl-loss-rate-probability-of-default-examples/)
[5](https://didattica.unibocconi.it/mypage/upload/51724_20100707_111731_DEFAULTRECOVERYREVIEW.PDF)
[6](https://www.bundesbank.de/resource/blob/704154/31bac58987486c54742e606a9f17abaf/mL/2004-08-01-dkp-02-data.pdf)
[7](https://www.pwc.com/hu/hu/szolgaltatasok/ifrs/ifrs_9/ifrs9_kiadvanyok/IFRS%209%20Impairment%20-%20Intercompany%20loans%20In%20depthINT(April%202019).pdf)
[8](https://cfjournal.hse.ru/article/download/10458/11996/)
[9](https://www.pkf-l.com/insights/ifrs-9-the-two-ways-of-calculating-ecls/)
[10](https://assets.kpmg.com/content/dam/kpmgsites/in/pdf/2025/01/expected-credit-loss-ecl.pdf)


Reduced-form approach


DS + Structured form Aprroach



[1] - 
[2] - 
