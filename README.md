# pd_modeling
Implementation of an automated solution that uses market data to generate a PD curve for assessing the credit risks of emitents

# Refresher основных терминов

**PD Modeling** (Probability of Default Modeling) — это моделирование вероятности дефолта заемщика, используемое для оценки кредитного риска в банках и финансовых институтах. Методология применяется в рамках требований IFRS 9 (International Financial Reporting Standard 9) и регуляторных требований Basel III.[1][2][3]

## Параметры кредитного риска

### PD (Probability of Default)

**Вероятность дефолта** — это вероятность того, что заемщик не выполнит свои долговые обязательства в течение определенного периода времени (обычно 12 месяцев или lifetime).[4][1]

- **Point-in-Time (PIT) PD**: Оценка на конкретный момент времени с учетом текущих экономических условий (требуется по IFRS 9)[1]
- **Through-the-Cycle (TTC) PD**: Средняя вероятность через экономический цикл (используется в Basel IRB подходе)[1]
- **Basel III PD floor**: Минимальная PD установлена на уровне 5 базисных пунктов (0.05%)[2]

### LGD (Loss Given Default)

**Потери при дефолте** — доля непогашенной задолженности, которую кредитор потеряет в случае дефолта заемщика. Измеряется в процентах от EAD.[5][4][1]

$$ LGD = 1 - RR $$

где $$RR$$ — Recovery Rate (коэффициент восстановления). Доля задолженности, которую кредитор сможет вернуть после дефолта через реализацию залога, реструктуризацию или банкротную процедуру. Типичное значение для корпоративных облигаций — 40%.  [5]

**Downturn LGD**: Оценка потерь в неблагоприятных экономических условиях (требуется по Basel).[1]

### EAD (Exposure at Default)

**Задолженность на момент дефолта** — сумма, которую заемщик должен кредитору в момент дефолта. Для кредитных линий учитывается вероятность дополнительного использования лимита до дефолта.[4][1]

**Систематический риск восстановления**: Исследования показывают, что RR коррелирует с PD — в периоды экономических спадов растут дефолты и одновременно падают recovery rates.[6][5]

## Основная формула Expected Credit Loss

### Формула ECL по IFRS 9

$$ ECL = PD \times LGD \times EAD $$

где:
- $$ECL$$ — Expected Credit Loss (ожидаемые кредитные потери)[4][1]
- $$PD$$ — вероятность дефолта за период (12 месяцев или lifetime)[1]
- $$LGD$$ — потери при дефолте (%)[1]
- $$EAD$$ — задолженность на момент дефолта[1]

### Пример расчета

Заемщик должен 1,000,000 руб. с вероятностью дефолта 20% и ожидаемыми потерями при дефолте 70%:[4]

$$ ECL = 0.20 \times 0.70 \times 1,000,000 = 140,000 \text{ руб.} $$

## Triangle formula



## Еще какая-то


# Reduced-form модели

## Связь с интенсивностью дефолта

В reduced-form моделях используется **hazard rate** (интенсивность дефолта) $$\lambda(t)$$:[1]

$$ S(t) = e^{-\int_0^t \lambda(u) du} $$

где $$S(t)$$ — survival probability (вероятность выживания до момента $$t$$).[1]

---
Как выводятся данные формулы ?

**Функция выживания** $$S(t)$$ — вероятность того, что заемщик НЕ дефолтнет до момента $$t$$:

$$ S(t) = P(\tau > t) $$

где $$\tau$$ — случайное время дефолта.[1][2]

**Интенсивность дефолта** $$\lambda(t)$$ — мгновенная условная вероятность дефолта:

$$ \lambda(t) = \lim_{\Delta t \to 0} \frac{P(t < \tau \leq t + \Delta t \mid \tau > t)}{\Delta t} $$

### 1: Связь через условную вероятность

Рассмотрим вероятность выживания в момент $$t + \Delta t$$:

$$ S(t + \Delta t) = P(\tau > t + \Delta t) $$

Эту вероятность можно разложить через условную вероятность:

$$ S(t + \Delta t) = P(\tau > t + \Delta t \mid \tau > t) \cdot P(\tau > t) $$

$$ S(t + \Delta t) = P(\tau > t + \Delta t \mid \tau > t) \cdot S(t) $$

### 2: Выражение через интенсивность

Условная вероятность **НЕ** дефолтнуть на интервале $$[t, t + \Delta t]$$:

$$ P(\tau > t + \Delta t \mid \tau > t) = 1 - P(t < \tau \leq t + \Delta t \mid \tau > t) $$

По определению интенсивности:

$$ P(t < \tau \leq t + \Delta t \mid \tau > t) = \lambda(t) \Delta t + o(\Delta t) $$

где $$o(\Delta t)$$ — член высшего порядка малости.

Следовательно:

$$ P(\tau > t + \Delta t \mid \tau > t) = 1 - \lambda(t) \Delta t + o(\Delta t) $$

### 3: Подстановка в уравнение

$$ S(t + \Delta t) = [1 - \lambda(t) \Delta t] \cdot S(t) + o(\Delta t) $$

$$ S(t + \Delta t) - S(t) = -\lambda(t) S(t) \Delta t + o(\Delta t) $$

Разделим на $$\Delta t$$ и устремим $$\Delta t \to 0$$:

$$ \frac{S(t + \Delta t) - S(t)}{\Delta t} = -\lambda(t) S(t) $$

По определению производной:

$$ \frac{dS(t)}{dt} = -\lambda(t) S(t) $$

### Решение дифференциального уравнения

Это обыкновенное дифференциальное уравнение (ОДУ) с разделяющимися переменными:

$$ \frac{dS(t)}{S(t)} = -\lambda(t) dt $$

Интегрируем обе части от 0 до $$t$$:

$$ \int_0^t \frac{dS(u)}{S(u)} = -\int_0^t \lambda(u) du $$

$$ \ln S(t) - \ln S(0) = -\int_0^t \lambda(u) du $$

Начальное условие: $$S(0) = 1$$ (в начальный момент заемщик точно жив, т.е. не дефолтнул):

$$ \ln S(t) = -\int_0^t \lambda(u) du $$

Потенцируя обе части:

$$ S(t) = \exp\left(-\int_0^t \lambda(u) du\right) $$

Что мы и получили в начале

## Интуитивное объяснение

### Аналогия с радиоактивным распадом

Формула $$S(t) = e^{-\lambda t}$$ (для постоянной $$\lambda$$) **идентична закону радиоактивного распада**:

$$ N(t) = N_0 e^{-\lambda t} $$

**Физический смысл**: В каждый момент времени фиксированная доля атомов (или заемщиков) распадается (дефолтит) с интенсивностью $$\lambda$$. Чем выше интенсивность, тем быстрее убывает количество "выживших".



## Связь с вероятностью дефолта

**Функция распределения времени дефолта** (cumulative distribution function):

$$ F(t) = P(\tau \leq t) = 1 - S(t) $$

**Плотность распределения времени дефолта**:

$$ f(t) = \frac{dF(t)}{dt} = -\frac{dS(t)}{dt} = \lambda(t) S(t) $$

Это показывает, что интенсивность $$\lambda(t)$$ связана с плотностью дефолта через:

$$ \lambda(t) = \frac{f(t)}{S(t)} = \frac{f(t)}{1 - F(t)} $$

Это называется **hazard function** в теории надёжности и survival analysis.

---




Безусловная вероятность дефолта:

$$ PD(t) = 1 - S(t) $$

## Трехстадийная модель IFRS 9

**Stage 1**: Нормальное качество кредита — провизии на основе 12-месячной ECL.[8][9]

**Stage 2**: Существенное ухудшение кредитного качества (SICR — Significant Increase in Credit Risk) — провизии на основе lifetime ECL.[9][8]

**Stage 3**: Дефолт или credit-impaired — провизии на основе lifetime ECL с учетом фактических потерь.[8][9]

## Ключевые источники и стандарты

### IFRS 9 (2014, вступил в силу 2018)

Международный стандарт финансовой отчетности, требующий:
- Forward-looking оценку кредитных потерь [1]
- Point-in-time параметры PD, LGD, EAD [1]
- Трехстадийную модель обесценения [8]

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

