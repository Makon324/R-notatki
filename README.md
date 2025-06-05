**ROZSZERZONA NOTATKA DO KOLOKWIUM: FUNKCJE W JĘZYKU R I SZCZEGÓŁOWY OPIS PARAMETRÓW**

---

## Spis treści

1. [Funkcje do estymacji punktowej i opis ich parametrów](#estymatory-punktowe)
   1.1. `mean()`
   1.2. `median()`
   1.3. `var()`
   1.4. `sd()`
   1.5. `quantile()`
2. [Testowanie hipotez i pobieranie p‑value](#testowanie-hipotez)
   2.1. `t.test()` – test t‑Studenta
   2.2. `prop.test()` – test jednej lub dwóch proporcji
   2.3. `chisq.test()` – test chi‑kwadrat
3. [Przedziały ufności i ich parametry](#przedzialy-ufnosci)
   3.1. `t.test(..., conf.level = …)` – przedział ufności dla średniej
   3.2. `prop.test(..., conf.level = …)` – przedział ufności dla proporcji
   3.3. `varTest()` (pakiet **EnvStats**) – przedział ufności dla wariancji
4. [Estymacja metodą największej wiarygodności (MLE)](#mle)
   4.1. `mle()` (pakiet **stats4**) – MLE dla pełnych danych
   4.2. `fitdistcens()` (pakiet **fitdistrplus**) – MLE dla danych pogrupowanych/cenzurowanych
5. [Dodatkowe uwagi i przykłady](#dodatkowe-uwagi)

---

<a name="estymatory-punktowe"></a>

## 1. Estymatory punktowe i opis funkcji

W tej sekcji opiszemy pięć podstawowych funkcji w R, służących do oszacowania parametrów populacji na podstawie próby.

### 1.1. `mean(x, trim = 0, na.rm = FALSE, ...)`

* **Zastosowanie:** Estymuje średnią arytmetyczną $\bar{x}$ próby.
* **Podstawowa wersja:**

  ```r
  mean(dane)
  ```
* \*\*Główne argumenty:

  * `x` – wektor liczb (numeryczny lub logiczny).
  * `trim` – ułamek (między 0 i 0.5) określający, jaką część skrajnych obserwacji odciąć przed liczeniem średniej (tzw. średnia przycięta).

    * Domyślnie `trim = 0`, czyli standardowa średnia bez przycięcia.
    * Przykład: `mean(x, trim = 0.1)` oznacza odrzucenie 10% najmniejszych i 10% największych obserwacji.
  * `na.rm` – logiczna wartość:

    * `FALSE` (domyślnie): jeżeli wektor `x` zawiera jakiekolwiek `NA`, zwracane jest `NA`.
    * `TRUE`: przed obliczeniem średniej R usunie wartości `NA`.
* **Zwracana wartość:** Jedna liczba – średnia arytmetyczna (lub przycięta, jeśli użyto `trim`).

---

### 1.2. `median(x, na.rm = FALSE)`

* **Zastosowanie:** Oblicza medianę próby, czyli wartość środkową.
* \*\*Argumenty:

  * `x` – wektor liczb.
  * `na.rm` – jeśli `TRUE`, R usunie wartości `NA` przed obliczeniem.
* **Zwracana wartość:** Jedna liczba – mediana.
* **Uwagi:** Mediana jest miarą odporności (robust) wobec wartości odstających („outliers”), w przeciwieństwie do średniej arytmetycznej.

---

### 1.3. `var(x, y = NULL, na.rm = FALSE, use)`

* **Zastosowanie:** Oblicza wariancję próby $\displaystyle s^2 = \frac{1}{n-1} \sum_{i=1}^n (x_i - \bar{x})^2$.
* \*\*Argumenty:

  * `x` – wektor liczb (lub macierz/ramka danych, gdy porównujemy dwie kolumny).
  * `y` – opcjonalny drugi wektor tej samej długości, pozwalający na obliczenie kowariancji między `x` i `y` (jeśli `y` ≠ `NULL`).
  * `na.rm` – jeżeli `TRUE`, usuwamy wszystkie pary `(x_i, y_i)`, gdzie występuje `NA`.
  * `use` – parametr bardziej zaawansowany, kontroluje, jak traktować pary z `NA` (wartość domyślna `"everything"`).
* **Zwracana wartość:** Jedna liczba – wariancja próby (domyślnie z dzieleniem przez `n − 1`).

---

### 1.4. `sd(x, na.rm = FALSE)`

* **Zastosowanie:** Oblicza odchylenie standardowe próby $\displaystyle s = \sqrt{\text{var}(x)}$.
* \*\*Argumenty:

  * `x` – wektor liczb.
  * `na.rm` – jeśli `TRUE`, usuwa `NA` przed obliczeniem.
* **Zwracana wartość:** Jedna liczba – odchylenie standardowe (próby).

---

### 1.5. `quantile(x, probs = seq(0, 1, 0.25), type = 7, na.rm = FALSE)`

* **Zastosowanie:** Oblicza wybrane kwartyle (lub dowolne percentyle) wektora `x`.
* \*\*Argumenty:

  * `x` – wektor liczb.
  * `probs` – wektor wartości z przedziału $[0,1]$ określających, jakie kwantyle policzyć. Przykład:

    * `probs = c(0.25, 0.5, 0.75)` → pierwszy kwartyl (25 %), mediana (50 %), trzeci kwartyl (75 %).
    * `probs = 0.95` → 95‑y percentyl.
  * `type` – liczba od 1 do 9 określająca metodę interpolacji kwantyli (w R domyślny `type = 7` odpowiada definicji z S‑Plus).
  * `na.rm` – jeżeli `TRUE`, usuwa wartości `NA`.
* **Zwracana wartość:** Wektor kwantyli o takiej samej długości jak `probs`.

---

<a name="testowanie-hipotez"></a>

## 2. Testowanie hipotez i pobieranie p‑value

W tej części omówimy najważniejsze funkcje w R do testowania hipotez: `t.test()`, `prop.test()` oraz `chisq.test()`. Dla każdej funkcji podamy wszystkie kluczowe argumenty i szczegóły ich działania.

---

### 2.1. `t.test(x, y = NULL, alternative = c("two.sided","less","greater"), mu = 0, paired = FALSE, var.equal = FALSE, conf.level = 0.95, ...)`

#### Cel:

* **Test t‑Studenta** służy do testowania hipotez o średnich w następujących wariantach:

  1. **Jedna próba**: czy średnia w populacji $\mu$ jest równa zadanej wartości `mu`.
  2. **Dwie niezależne próby**: porównanie średnich dwóch różnych grup.
  3. **Dwie sparowane próby** (paired): porównanie średnich w parach (np. przed i po interwencji).

#### Szczegółowy opis argumentów:

1. **`x`** – wektor numeryczny z danymi pierwszej grupy lub z próby w przypadku testu jednej próby.
2. **`y`** – (opcjonalnie) wektor numeryczny z danymi drugiej grupy (test dla dwóch niezależnych próbek).

   * Jeśli `y = NULL`, R zakłada **test dla jednej próby**.
   * Jeżeli `paired = TRUE`, to `x` i `y` muszą być tej samej długości i będą traktowane jako pary (test sparowany).
3. **`alternative`** – typ testu:

   * `"two.sided"` (domyślnie) – test dwustronny, $H_1: \mu \neq \mu_0$ lub w wariancie dwóch próbek $H_1: \mu_x \neq \mu_y$.
   * `"less"` – test jednostronny, $H_1: \mu < \mu_0$ lub $H_1: \mu_x < \mu_y$.
   * `"greater"` – test jednostronny, $H_1: \mu > \mu_0$ lub $H_1: \mu_x > \mu_y$.
4. **`mu`** – wartość oczekiwana $\mu_0$ pod hipotezą zerową w teście jednej próby (lub w wariancie dwóch próbek traktowanych jako test różnicy od `mu` (zwykle przestawiana jest różnica średnich, ale rzadko jest używana w teście na dwóch próbach)). Domyślnie `mu = 0`.
5. **`paired`** – logiczna flaga (`TRUE`/`FALSE`).

   * `FALSE` (domyślnie): test niezależnych prób.
   * `TRUE`: test sparowany (porównujemy „przed vs. po” lub „para A vs. para B” w `x` i `y`).
6. **`var.equal`** – logiczna flaga (`TRUE`/`FALSE`). Dotyczy testu dwóch niezależnych prób:

   * `FALSE` (domyślnie): używa testu Welcha, który nie zakłada równości wariancji w obu grupach.
   * `TRUE`: używa klasycznego testu t‑Studenta dla dwóch grup, zakładając równość wariancji (połączoną wariancję).
7. **`conf.level`** – poziom ufności dla przedziału ufności (domyślnie `0.95`). Przyjmuje wartości z przedziału $(0,1)$.
8. **`...`** – dodatkowe argumenty (np. `subset` przy pracy z obiektami typu `formula`).
9. **Zwracane elementy w wyniku** (obiekt klasy `"htest"`):

   * `statistic` – wartość statystyki testowej t.
   * `parameter` – liczba stopni swobody (df).
   * `p.value` – wartość p.
   * `conf.int` – wektor długości 2: dolna i górna granica przedziału ufności dla średniej (jedna próba) lub różnicy średnich (dwie próby).
   * `estimate` – estymowana średnia (jedna próba) lub estymowane średnie w grupach i/lub różnica średnich (dwie próby).
   * `null.value` – wartość hipotezy zerowej (`mu`).
   * `alternative` – jakim testem (dwustronny/jednostronny).
   * `method` – tekstowy opis użytej procedury (np. `"One Sample t-test"` lub `"Welch Two Sample t-test"`).
   * `data.name` – nazwa obiektu wektora lub formuły użytej w teście.

#### Przykłady użycia:

1. **Test t dla jednej próby**

   ```r
   dane <- c(5.1, 4.9, 5.0, 5.2, 5.1)
   # Sprawdzamy H0: mu = 5, test dwustronny, poziom ufności 95%
   wynik <- t.test(x = dane, mu = 5, alternative = "two.sided", conf.level = 0.95)
   # Odczyty:
   wynik$statistic  # statystyka t
   wynik$parameter  # df = n - 1
   wynik$p.value    # wartość p
   wynik$conf.int   # 95% CI dla mu
   wynik$estimate   # estymowana srednia próby
   ```
2. **Test t dla dwóch niezależnych prób (zakładamy równość wariancji)**

   ```r
   grupa1 <- c(5.1, 4.9, 5.0, 5.2, 5.1)
   grupa2 <- c(5.3, 5.4, 5.2, 5.5, 5.3)
   wynik2 <- t.test(x = grupa1,
                    y = grupa2,
                    alternative = "two.sided",
                    var.equal = TRUE,
                    conf.level = 0.95)
   # Odczyty:
   wynik2$statistic   # t
   wynik2$parameter   # df = n1 + n2 - 2
   wynik2$p.value     # p-value
   wynik2$conf.int    # 95% CI dla (mu1 - mu2)
   wynik2$estimate    # estymowane srednie w grupie1 i grupie2
   ```
3. **Test t sparowany**

   ```r
   # Przykładowe pomiary przed i po jakiejś interwencji
   przed <- c(100, 102,  98,  97, 101)
   po    <- c(102, 103, 100,  99, 102)
   wynik_paired <- t.test(x = przed,
                          y = po,
                          paired = TRUE,
                          alternative = "two.sided",
                          conf.level = 0.95)
   # Tu p-value dotyczy hipotezy H0: średnia różnica = 0
   wynik_paired$p.value
   wynik_paired$conf.int  # 95% CI dla średniej różnicy (przed - po)
   ```

---

### 2.2. `prop.test(x, n, p = NULL, alternative = c("two.sided","less","greater"), conf.level = 0.95, correct = TRUE)`

#### Cel:

* **Test proporcji** w następujących wariantach:

  1. **Jedna próba**: sprawdzenie, czy proporcja sukcesów $p$ w populacji równa jest zadanej wartości `p0`.
  2. **Dwie próby**: test, czy proporcje sukcesów w dwóch próbach są równe ($p_1 = p_2$).

#### Szczegółowy opis argumentów:

1. **`x`** – liczba sukcesów (lub wektor liczności sukcesów w obu próbach, jeśli testujemy dwie proporcje).

   * Dla jednej próby: pojedyncze $x$, np. `x = 45` (45 sukcesów).
   * Dla dwóch prób: wektor dwuelementowy, np. `x = c(45, 30)` (45 sukcesów w próbie 1, 30 sukcesów w próbie 2).
2. **`n`** – łączna liczba prób (lub wektor dwuelementowy, gdy są dwie próby).

   * One-sample: `n = 100` (100 obserwacji, w tym `x` sukcesów).
   * Two-sample: `n = c(100, 120)` → w próbie 1 jest 100 osób, w próbie 2 jest 120 osób.
3. **`p`** – wartość pod hipotezą zerową w teście jednej próby (np. `p = 0.5`).

   * Jeśli `p = NULL` (domyślnie), a `length(x) = 1`, R testuje $H_0: p = 0.5$.
   * Jeśli `length(x) = 1` i chcemy innej wartości, podajemy `p = 0.6` → testuje $H_0: p = 0.6$.
   * Jeśli `length(x) = 2` (dwie próby), argument `p` jest ignorowany.
4. **`alternative`** – typ testu:

   * `"two.sided"` – dwustronny (domyślnie).
   * `"less"` – jednostronny, $H_1: p < p_0$ (lub $p_1 < p_2$ w wariancie dwóch prób).
   * `"greater"` – jednostronny, $H_1: p > p_0$ (lub $p_1 > p_2$).
5. **`conf.level`** – poziom ufności dla przedziału ufności (domyślnie `0.95`).
6. **`correct`** – logiczna flaga (`TRUE`/`FALSE`).

   * `TRUE` (domyślnie): R stosuje korekcję Yatesa (continuity correction) dla dwuprocentowego testu chi‑kwadrat, co często zaleca się przy małych wartościach $n$.
   * `FALSE`: brak korekcji Yatesa (test dokładny bazuje na przybliżeniu Gaussa‑Laplace’a).
7. **Zwracana wartość (obiekt klasy `"htest"`)**:

   * `statistic` – wartość statystyki testowej (zazwyczaj chi‑kwadrat$^2$ z 1 df lub znormalizowana statystyka z przybliżeniem normalnym).
   * `parameter` – liczba stopni swobody (df).
   * `p.value` – wartość p.
   * `conf.int` – wektor długości 2: dolna i górna granica przedziału ufności dla prawdziwej proporcji (jedna próba) lub dla różnicy proporcji (dwie próby).
   * `estimate` – estymowana wartość $\hat p$ (jedna próba) lub wektor $\hat p_1, \hat p_2$ (dwie próby).
   * `null.value` – wartość hipotetyczna $p_0$ (jedna próba).
   * `alternative` – rodzaj testu (dwustronny/jednostronny).
   * `method` – np. `"1-sample proportions test with continuity correction"` albo `"2-sample test for equality of proportions with continuity correction"`.
   * `data.name` – nazwy obiektów użytych jako `x` i `n`.

#### Przykłady:

1. **Test jednej proporcji**

   ```r
   # 78 sukcesów na 120 obserwacji; H0: p = 0.6; test dwustronny; 95% CI
   wynik_p1 <- prop.test(x = 78, n = 120, p = 0.6, alternative = "two.sided", conf.level = 0.95, correct = TRUE)
   wynik_p1$estimate    # estymowana proporcja ~ 78/120
   wynik_p1$p.value      # p-value
   wynik_p1$conf.int     # 95% CI dla p
   ```
2. **Test porównania dwóch proporcji**

   ```r
   # Próbka 1: 78/120; Próbka 2: 45/100; H0: p1 = p2
   wynik_p2 <- prop.test(x = c(78, 45),
                         n = c(120, 100),
                         alternative = "two.sided",
                         conf.level = 0.95,
                         correct = FALSE)
   wynik_p2$estimate    # estymowane p1 i p2
   wynik_p2$p.value      # p-value testu
   wynik_p2$conf.int     # 95% CI dla (p1 - p2)
   ```

---

### 2.3. `chisq.test(x, y = NULL, correct = TRUE, p = rep(1/length(x), length(x)), rescale.p = FALSE, simulate.p.value = FALSE, B = 2000, ...)`

#### Cel:

* **Test chi‑kwadrat** ma kilka zastosowań:

  1. **Test niezależności** w tabeli kontyngencji.
  2. **Test zgodności rozkładu** (goodness of fit).

#### Szczegółowy opis argumentów:

1. **`x`** –

   * W przypadku testu niezależności: macierz lub ramka danych, gdzie w komórkach są liczności (np. liczba obserwacji w każdej kategorii dwóch zmiennych).
   * W przypadku testu zgodności (jednowymiarowego): wektor liczności obserwacji kategorii (np. ile razy wystąpiła kategoria 1, kategoria 2, kategoria 3).
2. **`y`** – opcjonalne podanie drugiego wektora kategorii, wtedy R przekształca je automatycznie w tabelę kontyngencji i wykonuje test niezależności.
3. **`correct`** – logiczna flaga, czy stosować korekcję Yatesa (continuity correction) dla tablic 2×2 (domyślnie `TRUE`).

   * `TRUE`: w tabelach 2×2 R odejmuje 0,5 od różnicy między zaobserwowanymi a oczekiwanymi licznościami, by skorygować błąd przy małych próbach.
   * `FALSE`: brak korekcji.
4. **`p`** – wektor prawdopodobieństw oczekiwanej dystrybucji kategorii w przypadku testu zgodności.

   * Domyślnie `p = rep(1/length(x), length(x))`, czyli wszystkie kategorie są jednakowo prawdopodobne.
   * Można podać własne $p_j$, pod którymi test ma sprawdzić, czy obserwacje w `x` odpowiadają dystrybucji $p$.
5. **`rescale.p`** – logiczna flaga (domyślnie `FALSE`).

   * Jeżeli `TRUE`, R automatycznie przeliczy sumę `p` do wartości 1 (przydatne, gdy `p` nie sumuje się dokładnie do 1).
6. **`simulate.p.value`** – logiczna wartość (domyślnie `FALSE`).

   * `TRUE`: w przypadku dużych tabel (np. krosowanie wielu kategorii) R wygeneruje empiryczną wartość p przez symulację Monte Carlo.
   * `FALSE`: bazuje na przybliżeniu chi‑kwadrat asymptotycznym.
7. **`B`** – liczba iteracji w symulacji Monte Carlo (jeśli `simulate.p.value = TRUE`).
8. **`...`** – dodatkowe argumenty (np. `subset` przy użyciu formuły).
9. **Zwracany obiekt (`"htest"` lub `"htest"` podobny):**

   * `statistic` – wartość statystyki chi‑kwadrat.
   * `parameter` – stopnie swobody:

     * Dla testu niezależności w tabeli $r \times c$: $(r-1)(c-1)$.
     * Dla testu zgodności: $(k-1)$ (gdzie $k$ to liczba kategorii).
   * `p.value` – wartość p.
   * `observed` – macierz lub wektor zaobserwowanych liczności.
   * `expected` – macierz lub wektor liczności oczekiwanych przy $H_0$.
   * `residuals` – macierz lub wektor reszt (standardizowanych odchyleń $(O - E)/\sqrt{E}$).
   * `method` – np. `"Pearson's Chi-squared test"`.
   * `data.name` – nazwa obiektu wejściowego.

#### Przykłady:

1. **Test niezależności w tabeli 2×2**

   ```r
   # Załóżmy, że badamy związek płci (M/F) i decyzji (TAK/NIE)
   tab <- matrix(c(20, 15,
                   25, 10), nrow = 2, byrow = TRUE)
   # Wiersze: płeć (M, F); kolumny: TAK, NIE
   dimnames(tab) <- list(Płeć = c("M", "F"),
                         Decyzja = c("TAK", "NIE"))
   wynik_chi <- chisq.test(x = tab, correct = TRUE)
   wynik_chi$statistic   # chi-squared
   wynik_chi$parameter   # df = (2-1)*(2-1) = 1
   wynik_chi$p.value     # p-value
   wynik_chi$expected    # oczekiwane liczności
   wynik_chi$residuals   # reszty standaryzowane
   ```
2. **Test zgodności (goodness of fit)**

   ```r
   # Zbadamy, czy obserwowany rozkład w 4 kategoriach odpowiada rozkładowi [0.25, 0.25, 0.25, 0.25]
   obserwacje <- c(30, 20, 25, 25)  
   # 100 obserwacji w sumie, zamierzamy sprawdzić, czy każda kategoria
   # powinna się pojawić z prawdopodobieństwem 0.25
   wynik_gof <- chisq.test(x = obserwacje,
                          p = c(0.25, 0.25, 0.25, 0.25),
                          rescale.p = TRUE,
                          correct = FALSE)
   wynik_gof$statistic   # chi-squared
   wynik_gof$parameter   # df = 4 - 1 = 3
   wynik_gof$p.value     # p-value
   wynik_gof$expected    # oczekiwane liczności (również ~25, 25, 25, 25)
   ```

---

<a name="przedzialy-ufnosci"></a>

## 3. Przedziały ufności w R

Powiązane są one z testami istotności (np. `t.test()` i `prop.test()`) oraz z dedykowanymi funkcjami do wariancji.

---

### 3.1. `t.test(..., conf.level = 0.95)`

* **Zwraca w wyniku** element `conf.int` – przedział ufności dla średniej (jedna próba) lub różnicy średnich (dw i prób).
* **Argument `conf.level`** – wartość z przedziału $(0,1)$, określająca poziom ufności (np. `0.99` dla 99 % CI, `0.90` dla 90 % CI).
* **Interpretacja w wyniku `conf.int`:**

  * Dla testu jednej próby:

    $$
    \bigl[\bar{x} - t_{\alpha/2,\,n-1} \cdot \tfrac{s}{\sqrt{n}},\;\bar{x} + t_{\alpha/2,\,n-1} \cdot \tfrac{s}{\sqrt{n}}\bigr]
    $$
  * Dla testu dwóch niezależnych prób (var.equal = TRUE):

    $$
    \;(\bar{x}_1 - \bar{x}_2)\; \pm\; t_{\alpha/2,\,n_1+n_2-2} \,\sqrt{\,s_p^2\bigl(\tfrac{1}{n_1} + \tfrac{1}{n_2}\bigr)\,},
    $$

    gdzie $s_p^2$ to skojarzona wariancja w obu grupach.
  * Dla testu obu prób (var.equal = FALSE, Welch): R oblicza granice CI wykorzystując przybliżone df wg formuły Welcha.

---

### 3.2. `prop.test(..., conf.level = 0.95)`

* **Zwraca w wyniku** element `conf.int` – przedział ufności dla proporcji (jedna próba) lub różnicy proporcji (dwie próby).
* **Argument `conf.level`** – analogicznie jak w `t.test`, ustala poziom ufności.
* **Metoda liczenia CI:**

  * Przy dużych próbach R stosuje przybliżenie normalne (jeśli `correct = FALSE`).
  * Jeśli `correct = TRUE`, jest stosowana korekcja ciągłości Yatesa, co wpływa na granice CI (zwykle trochę bliżej estymaty).
  * Dla małych prób standardowe przybliżenie może być niewystarczające; wówczas lepiej stosować np. dokładny przedział Cloppera–Pearsona (funkcja `binom.test` w R).

Przykład (jedna próba):

```r
wynik_p <- prop.test(x = 45, n = 100, conf.level = 0.95, correct = FALSE)
wynik_p$conf.int  # [0.350..., 0.549...]
```

Przykład (dwie próby):

```r
wynik_p2 <- prop.test(x = c(50, 30), n = c(200, 150), conf.level = 0.95, correct = TRUE)
wynik_p2$conf.int  # CI dla p1 - p2
```

---

### 3.3. `varTest(x, sigma.squared = NULL, alternative = c("two.sided","less","greater"), conf.level = 0.95)`

> **Uwaga:** `varTest()` pochodzi z pakietu **EnvStats**. Aby z niej skorzystać, należy uprzednio zainstalować i załadować pakiet:
>
> ```r
> install.packages("EnvStats")
> library(EnvStats)
> ```

#### Cel:

* Obliczenie przedziału ufności dla wariancji populacji $\sigma^2$ (test jednej próby) oraz (opcjonalnie) przetestowanie hipotezy $H_0: \sigma^2 = \sigma_0^2$.

#### Opis argumentów:

1. **`x`** – wektor liczb (dane z próby).
2. **`sigma.squared`** – wartość $\sigma_0^2$ pod hipotezą zerową (jeśli chcemy wykonać test).

   * Jeśli `sigma.squared = NULL` (domyślnie), `varTest` oblicza tylko przedział ufności.
   * Jeśli `sigma.squared = s0` (liczba), R wykonuje również test $H_0: \sigma^2 = s0$.
3. **`alternative`** – kierunek testu w przypadku `sigma.squared` nie–`NULL`:

   * `"two.sided"` (domyślnie): $H_1: \sigma^2 \neq \sigma_0^2$.
   * `"less"`: $H_1: \sigma^2 < \sigma_0^2$.
   * `"greater"`: $H_1: \sigma^2 > \sigma_0^2$.
4. **`conf.level`** – poziom ufności dla przedziału, domyślnie `0.95`.
5. **Zwracana wartość (obiekt klasy `"htest"` lub podobny)**:

   * `statistic` – wartość statystyki $\chi^2 = \tfrac{(n-1) s^2}{\sigma_0^2}$ (jeśli `sigma.squared` podane) lub `NA` (jeśli tylko CI).
   * `parameter` – df = $n-1$.
   * `p.value` – wartość p (jeśli przeprowadzany jest test).
   * `conf.int` – wektor długości 2: granice przedziału ufności dla $\sigma^2$.
   * `estimate` – estymowana wariancja próby $s^2$.
   * `null.value` – wartość \$\sigma^2\_0\$, jeśli test.
   * `alternative` – typ testu.
   * `method` – np. `"Chi-square Test of Variance"`.
   * `data.name` – nazwa wektora `x`.

#### Formuła przedziału ufności 1−α:

$$
\Bigl[
\frac{(n-1)\,s^2}{\chi^2_{1-\alpha/2,\,n-1}},\quad
\frac{(n-1)\,s^2}{\chi^2_{\alpha/2,\,n-1}}
\Bigr].
$$

#### Przykład:

```r
library(EnvStats)
dane <- c(5.1, 4.9, 5.0, 5.2, 5.1)
# Przedział ufności 95% dla wariancji
wynik_var <- varTest(x = dane, conf.level = 0.95)  
wynik_var$estimate   # s^2
wynik_var$conf.int   # 95% CI dla sigma^2
```

---

<a name="mle"></a>

## 4. Estymacja metodą największej wiarygodności (MLE)

Estymacja MLE w R może być wykonana albo poprzez funkcje z pakietu **stats4**, albo – w przypadku danych pogrupowanych/cenzurowanych – za pomocą funkcji `fitdistcens()` z pakietu **fitdistrplus**.

---

### 4.1. `mle(minuslogl, start, method = "BFGS", lower = -Inf, upper = Inf, control = list(), ...)`

> **Uwaga:** `mle()` pochodzi z pakietu **stats4**. Aby z niej skorzystać, należy wpisać:
>
> ```r
> library(stats4)
> ```

#### Cel:

* Znalezienie wartości parametrów, które maksymalizują funkcję wiarygodności (lub minimalizują funkcję `minuslogl`, czyli ujemny log‑likelihood).

#### Kluczowe argumenty:

1. **`minuslogl`** – funkcja R przyjmująca argumenty będące parametrami do estymacji.

   * Funkcja ta dla zadanych wartości parametrów liczy ujemny log‑likelihood na podstawie wektora danych (który może być przekazany globalnie albo przez zamknięcie).
   * Zwracany wynik musi być jedną liczbą (ujemny log‑likelihood).
2. **`start`** – lista wartości początkowych (nazwy muszą się zgadzać z argumentami funkcji `minuslogl`).

   * Przykład: `start = list(mu = 0, sigma = 1)`.
3. **`method`** – algorytm optymalizacji (domyślnie `"BFGS"`). Inne możliwe: `"Nelder-Mead"`, `"CG"`, `"L-BFGS-B"` (umożliwia ograniczenia `lower` i `upper`).
4. **`lower`**, **`upper`** – wektory określające ograniczenia dla parametrów (używane, gdy `method = "L-BFGS-B"`).
5. **`control`** – lista opcji sterujących algorytmem (np. `list(maxit = 1000)` zmienia maksymalną liczbę iteracji).
6. **`...`** – dodatkowe argumenty przekazywane do algorytmu optymalizacji.
7. **Zwracana wartość (obiekt klasy `"mle"`)**:

   * `coef(fit)` – wektor estymowanych parametrów $\hat\theta$.
   * `vcov(fit)` – macierz kowariancji estymatorów (przybliżenie Fishera).
   * `summary(fit)` – podsumowanie z estymatami, odchyleniami standardowymi, wartością log‑likelihood, wartością AIC, itp.

#### Przykład: Estymacja MLE dla rozkładu normalnego

Zakładamy, że `dane` to wektor liczb. Chcemy estymować $\mu$ i $\sigma$ w normalnym $\mathcal{N}(\mu,\sigma^2)$.

```r
library(stats4)

# Definicja funkcji minusloglikelihood
loglik_norm <- function(mu, sigma) {
  if (sigma <= 0) return(Inf)  # wymóg sigma > 0
  # 'dane' traktujemy jako zmienną globalną lub domknięcie
  -sum(dnorm(dane, mean = mu, sd = sigma, log = TRUE))
}

# Wartości początkowe: średnia próby i odchylenie
start_vals <- list(mu = mean(dane), sigma = sd(dane))

# Uruchamiamy optymalizację
fit_norm <- mle(minuslogl = loglik_norm,
                start = start_vals,
                method = "L-BFGS-B",
                lower = c(mu = -Inf, sigma = 1e-8),  # sigma > 0
                upper = c(mu = Inf,  sigma = Inf))

# Wyniki:
coef(fit_norm)    # wektor [mu = ..., sigma = ...]
summary(fit_norm) # szczegółowe podsumowanie z SE, AIC, logLik
```

---

### 4.2. `fitdistcens(censdata, distr, start, ...)`

> **Uwaga:** `fitdistcens()` pochodzi z pakietu **fitdistrplus**. Aby z niej skorzystać, należy wpisać:
>
> ```r
> install.packages("fitdistrplus")
> library(fitdistrplus)
> ```

#### Cel:

* Dopasowanie rozkładu ciągłego do danych cenzurowanych lub pogrupowanych – w szczególności można dopasować rozkład normalny, log‑normalny, gamma itp., mając tylko przedziały i liczności (nie znając indywidualnych wartości w przedziałach).

#### Kluczowe argumenty:

1. **`censdata`** – ramka danych (`data.frame`) z co najmniej trzema kolumnami:

   * `left` – lewa granica przedziału (może być `-Inf` dla obserwacji lewostronnie cenzurowanych / „>=”).
   * `right` – prawa granica przedziału (może być `Inf` dla obserwacji prawostronnie cenzurowanych / „<=”).
   * `freq` – częstość (liczba obserwacji) w danym przedziale $[left, right)$.
   * Przykład:

     | left | right | freq  |
     | ---- | ----- | ----- |
     | 0    | 10    |   7   |
     | 10   | 20    |  15   |
     | 20   | 30    |   3   |
2. **`distr`** – ciąg znaków, nazwa rozkładu do dopasowania, np. `"norm"` (reguły: nazwa tak, jak w funkcjach `dnorm`/`pnorm`/`qnorm` itp.).
3. **`start`** – lista wartości początkowych dla parametrów rozkładu.

   * Dla rozkładu normalnego podajemy np. `list(mean = 13, sd = 5)`.
   * Dla rozkładu gamma: `list(shape = 2, rate = 0.5)`.
4. **`...`** – dodatkowe opcje przekazywane do wewnętrznego `mledist()`, np.:

   * `optim.method` – jaki algorytm optymalizacji zastosować (`"BFGS"`, `"L-BFGS-B"`, `"Nelder-Mead"`, itp.).
   * `lower`, `upper` – ograniczenia na parametry (tylko przy metodzie `"L-BFGS-B"`).
   * `control` – lista opcji dla optymalizatora (np. `list(maxit = 2000)`).
5. **Zwracana wartość (klasa `"fitdist"` z atrybutem `censdata`)**:

   * `estimate` – wektor estymowanych parametrów $\hat\theta$.
   * `sd` – przybliżone odchylenia standardowe estymatorów (z macierzy kowariancji).
   * `loglik` – wartość log‑likelihood w estymacie MLE.
   * `aic`, `bic` – informacyjne kryteria AIC, BIC.
   * Może się pojawić również `convergence` – kod zwrócony przez optymalizator (0 = konwergencja, >0 = niepoprawnie zakończone).
   * W atrybucie `censdata` znajduje się oryginalna ramka danych.

#### Przykład dopasowania rozkładu normalnego do danych pogrupowanych:

```r
library(fitdistrplus)

# Dane pogrupowane: przedziały 0-10, 10-20, 20-30 i liczności 7, 15, 3
df_cens <- data.frame(
  left  = c( 0, 10, 20),
  right = c(10, 20, 30),
  freq  = c( 7, 15,  3)
)

# Wartości początkowe: średnia około 13.4, sd około 8
fit_norm_grup <- fitdistcens(censdata = df_cens,
                             distr    = "norm",
                             start    = list(mean = 13.4, sd = 8),
                             optim.method = "L-BFGS-B",
                             lower = c(mean = -Inf, sd = 1e-6),
                             upper = c(mean = Inf,  sd = Inf))

# Wyniki:
fit_norm_grup$estimate   # szacowane mean i sd
fit_norm_grup$sd         # ich odchylenia standardowe
fit_norm_grup$loglik     # wartość log-likelihood
fit_norm_grup$aic        # AIC
```

---

<a name="dodatkowe-uwagi"></a>

## 5. Dodatkowe uwagi i przykłady praktyczne

### 5.1. Porównanie argumentów podobnych funkcji

| Funkcja                      | Główny cel                                 | Kluczowy argument „data”                                                      | Kluczowe argumenty „hipoteza”                                                                      | Argument budujący CI                                                      |
| ---------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `mean(x, ...)`               | Oblicza średnią arytmetyczną               | `x` – wektor numeryczny                                                       | brak (obliczamy estymator)                                                                         | brak (nie zwraca CI, do CI służy `t.test`)                                |
| `median(x, ...)`             | Oblicza medianę                            | `x` – wektor numeryczny                                                       | brak                                                                                               | brak                                                                      |
| `var(x, ...)`                | Oblicza wariancję próby                    | `x` (oraz opcjonalnie `y` dla kowariancji)                                    | brak                                                                                               | brak                                                                      |
| `sd(x)`                      | Oblicza odchylenie standardowe             | `x` – wektor numeryczny                                                       | brak                                                                                               | brak                                                                      |
| `quantile(x, ...)`           | Oblicza kwantyle                           | `x` – wektor numeryczny                                                       | brak                                                                                               | brak                                                                      |
| `t.test(x, ...)`             | Test t‑Studenta oraz CI dla średnich       | `x` (jedna próba) lub `x`,`y` (dwie próby)                                    | `mu` – wartość oczekiwana (jedna próba), `paired` – czy sparowane, `var.equal` – równość wariancji | `conf.level`                                                              |
| `prop.test(x, n, ...)`       | Test proporcji i CI dla proporcji          | `x` – liczba sukcesów lub wektor sukcesów, `n` – liczba prób (albo wektor)    | `p` – wartość proporcji pod H₀ (jedna próba), `alternative` – kierunek testu                       | `conf.level`                                                              |
| `chisq.test(x, ...)`         | Test chi‑kwadrat (niezależność/zgodność)   | `x` – macierz lub wektor liczności, `y` – (opcjonalny) drugi wektor kategorii | `p` – wektor oczekiwanych prawdopodobieństw (test zgodności), `correct` – korekcja Yatesa          | brak (choć `chisq.test` zwraca CI dla rem->?)                             |
| `varTest(x, ...)` (EnvStats) | CI i test dla wariancji                    | `x` – wektor liczb                                                            | `sigma.squared` – zadana wartość $\sigma_0^2$, `alternative` – kierunek testu                      | `conf.level`                                                              |
| `mle(minuslogl, ...)`        | MLE – dopasowanie dowolnego modelu         | brak bezpośrednio (dane w `minuslogl`)                                        | lista wartości początkowych `start`, metoda `method`, ograniczenia `lower`,`upper`                 | brak (nie buduje CI, ale można je wyciągnąć z `vcov()`)                   |
| `fitdistcens(censdata, ...)` | MLE dla danych pogrupowanych/cenzurowanych | `censdata` – ramka (left,right,freq)                                          | `distr` – nazwa rozkładu, `start`, `optim.method`, `lower`,`upper`                                 | brak (ale zwraca `sd` estymatorów i AIC, które pozwalają porównać modele) |

---

### 5.2. Kilka przykładów „krok po kroku”

#### Przykład 1: Obliczenie średniej, wariancji i odchylenia standardowego wraz z obsługą `NA`

```r
# Przykładowe dane z brakującymi wartościami
dane <- c(4.8, 5.0, NA, 5.2, 4.9, 5.1, NA, 5.3)

# 1. Średnia arytmetyczna (domyślnie nie ignoruje NA)
mean(dane)            # zwraca NA, bo są braki danych
mean(dane, na.rm = TRUE)  # usuwa NA i liczy na wektorze (4.8,5.0,5.2,4.9,5.1,5.3) => ~4.8833

# 2. Mediana (ignorujemy NA)
median(dane, na.rm = TRUE)  # mediana spośród (4.8, 5.0, 5.2, 4.9, 5.1, 5.3) = 5.0

# 3. Wariancja (z dzieleniem przez n-1) (ignoruje NA)
var(dane, na.rm = TRUE)     # wariancja próby

# 4. Odchylenie standardowe
sd(dane, na.rm = TRUE)      # sqrt(var)

# 5. Kwantyle
quantile(dane, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)  # 25%, 50%, 75%
```

---

#### Przykład 2: Test t dla jednej próby z testem jednostronnym i dostosowanym poziomem ufności

```r
dane2 <- c(5.12, 4.98, 5.03, 5.07, 5.10, 5.15, 4.95)

# H0: mu = 5, H1: mu > 5 (test jednostronny „greater”), alfa = 0.01 (co daje CI 99%)
wynik2 <- t.test(x = dane2,
                 mu = 5,
                 alternative = "greater",
                 conf.level = 0.99)  
                
# Interpretacja:
wynik2$statistic   # wartość t
wynik2$parameter   # df = n - 1 = 6
wynik2$p.value     # p-value jednostronny
wynik2$conf.int    # 99% CI dla mu, ale w teście jednostronnym to w praktyce [dolnaGranica, Inf)
# Jeśli dolnaGranica > 5 => odrzucamy H0 przy α = 0.01
```

---

#### Przykład 3: Test dwóch proporcji z korekcją Yatesa i bez niej

```r
# Dane: w grupie A 60/120, w grupie B 40/100
# Test dwustronny; chcemy porównać pA i pB

# 1. Z korekcją Yatesa (domyślnie)
wynik_c1 <- prop.test(x = c(60, 40),
                      n = c(120, 100),
                      alternative = "two.sided",
                      conf.level = 0.95,
                      correct = TRUE)
wynik_c1$estimate   # [pA_hat = 0.5, pB_hat = 0.4]
wynik_c1$p.value     # p-value z korekcją
wynik_c1$conf.int    # 95% CI dla (pA - pB)

# 2. Bez korekcji Yatesa
wynik_c2 <- prop.test(x = c(60, 40),
                      n = c(120, 100),
                      alternative = "two.sided",
                      conf.level = 0.95,
                      correct = FALSE)
wynik_c2$p.value     # p-value bez korekcji
wynik_c2$conf.int    # 95% CI bez korekcji
```

---

#### Przykład 4: Test chi‑kwadrat ze symulowanym p‑value (Monte Carlo)

```r
# Tabela większa niż 2×2 – np. 3×3
tab33 <- matrix(c(25, 15, 10,
                  10, 20, 15,
                  5,  10, 20), nrow = 3, byrow = TRUE)
dimnames(tab33) <- list(Wiersze = c("A","B","C"),
                        Kolumny = c("X","Y","Z"))

# Standardowy test chi-kwadrat
wynik33 <- chisq.test(x = tab33, correct = FALSE)
wynik33$statistic
wynik33$p.value       # przybliżony asymptotycznie
wynik33$expected

# Test z symulacją Monte Carlo (B = 5000 iteracji)
wynik_sim <- chisq.test(x = tab33, simulate.p.value = TRUE, B = 5000)
wynik_sim$p.value     # empiryczne p uzyskane przez symulacje
```

---

#### Przykład 5: Estymacja MLE rozkładu Poissona

```r
library(stats4)

# Załóżmy, że 'dane_pois' to wektor liczb całkowitych (obserwacje Poissona).
dane_pois <- c(3, 1, 0, 4, 2, 1, 3, 2, 0, 1)

# Definiujemy funkcję minusloglikelihood dla Poissona
loglik_pois <- function(lambda) {
  if (lambda <= 0) return(Inf)
  -sum(dpois(dane_pois, lambda = lambda, log = TRUE))
}

# Startowa wartość dla lambda = średnia próby
start_pois <- list(lambda = mean(dane_pois))

# MLE
fit_pois <- mle(minuslogl = loglik_pois,
                start    = start_pois,
                method   = "BFGS")

# Wyniki
coef(fit_pois)      # estymowane lambda
summary(fit_pois)   # odchylenie standardowe estymatora, AIC, logLik
```

---

#### Przykład 6: Dopasowanie rozkładu gamma do danych pogrupowanych

```r
library(fitdistrplus)

# Dane: przedziały czasu reakcji w milisekundach i liczności obserwacji
df_gamma <- data.frame(
  left  = c( 100, 200, 300, 400),
  right = c(200, 300, 400, 500),
  freq  = c( 50,  80,  60,  40)
)

# Zakładamy, że dane w każdym przedziale poch. z rozkładu gamma(shape, rate)
# Wartości początkowe (np. shape = 2, rate = 0.01)
fit_gamma <- fitdistcens(censdata = df_gamma,
                         distr    = "gamma",
                         start    = list(shape = 2, rate = 0.01),
                         optim.method = "L-BFGS-B",
                         lower = c(shape = 1e-6, rate = 1e-6),
                         upper = c(shape = Inf,   rate = Inf))

# Wyniki
fit_gamma$estimate   # shape i rate
fit_gamma$sd         # odchylenia standardowe obu estymatorów
fit_gamma$loglik     # log-likelihood
fit_gamma$aic        # AIC
```

---

### 5.3. Podsumowanie kluczowych parametrów funkcji

#### `t.test()`

| Argument      | Opis                                                                 | Domyślnie     |
| ------------- | -------------------------------------------------------------------- | ------------- |
| `x`           | Wektor numeryczny dla jednej próby lub pierwsza grupa.               | (brak)        |
| `y`           | (Opcjonalnie) Wektor numeryczny dla drugiej grupy (dwie próby).      | `NULL`        |
| `alternative` | Typ testu: `"two.sided"`, `"less"`, `"greater"`.                     | `"two.sided"` |
| `mu`          | Wartość średniej w hipotezie zerowej (dla testu jednej próby).       | `0`           |
| `paired`      | Czy test jest sparowany? (`TRUE`/`FALSE`).                           | `FALSE`       |
| `var.equal`   | Czy założyć równość wariancji w teście dwóch prób? (`TRUE`/`FALSE`). | `FALSE`       |
| `conf.level`  | Poziom ufności (np. `0.95` dla 95 % CI).                             | `0.95`        |
| `...`         | Dodatkowe argumenty (np. `subset` przy formule).                     | –             |

#### `prop.test()`

| Argument      | Opis                                                              | Domyślnie          |
| ------------- | ----------------------------------------------------------------- | ------------------ |
| `x`           | Liczba sukcesów (jedna próba) lub wektor długości 2 (dwie próby). | (brak)             |
| `n`           | Liczba prób (jedna próba) lub wektor długości 2 (dwie próby).     | (brak)             |
| `p`           | Wartość proporcji pod H₀ (jedna próba).                           | `NULL` (czyli 0.5) |
| `alternative` | Typ testu: `"two.sided"`, `"less"`, `"greater"`.                  | `"two.sided"`      |
| `conf.level`  | Poziom ufności (np. `0.95`).                                      | `0.95`             |
| `correct`     | Czy stosować korekcję Yatesa (`TRUE`/`FALSE`).                    | `TRUE`             |

#### `chisq.test()`

| Argument           | Opis                                                                     | Domyślnie   |
| ------------------ | ------------------------------------------------------------------------ | ----------- |
| `x`                | Macierz lub wektor liczności (tabela kontyngencji lub wektor kategorii). | (brak)      |
| `y`                | (Opcjonalnie) Drugi wektor kategorii (jeśli `x` to wektor).              | `NULL`      |
| `correct`          | Korekcja Yatesa w tabelach 2×2 (`TRUE`/`FALSE`).                         | `TRUE`      |
| `p`                | Wektor oczekiwanych prawdopodobieństw (test zgodności).                  | R=jednakowe |
| `rescale.p`        | Przeskalowanie wektora `p` tak, aby suma == 1.                           | `FALSE`     |
| `simulate.p.value` | Czy symulować p‑value metodą Monte Carlo (`TRUE`/`FALSE`).               | `FALSE`     |
| `B`                | Liczba iteracji w symulacji (`simulate.p.value = TRUE`).                 | `2000`      |
| `...`              | Dodatkowe argumenty (np. `subset` przy formule).                         | –           |

#### `varTest()` (pakiet **EnvStats**)

| Argument        | Opis                                                                       | Domyślnie     |
| --------------- | -------------------------------------------------------------------------- | ------------- |
| `x`             | Wektor liczb (dane).                                                       | (brak)        |
| `sigma.squared` | Wartość $\sigma_0^2$ pod H₀ (jeśli wykonujemy test).                       | `NULL`        |
| `alternative`   | `"two.sided"`, `"less"`, `"greater"` (tylko gdy `sigma.squared` ≠ `NULL`). | `"two.sided"` |
| `conf.level`    | Poziom ufności (np. `0.95`).                                               | `0.95`        |

#### `mle()` (pakiet **stats4**)

| Argument         | Opis                                                                                            | Domyślnie                |
| ---------------- | ----------------------------------------------------------------------------------------------- | ------------------------ |
| `minuslogl`      | Funkcja zwracająca ujemny log‑likelihood na podstawie parametrów.                               | (brak; wymaga definicji) |
| `start`          | Lista wartości początkowych dla parametrów (nazwy muszą się zgadzać z argumentami `minuslogl`). | (brak)                   |
| `method`         | Metoda optymalizacji: `"BFGS"`, `"Nelder-Mead"`, `"CG"`, `"L-BFGS-B"`, itp.                     | `"BFGS"`                 |
| `lower`, `upper` | Wektory ograniczeń (tylko przy `method = "L-BFGS-B"`).                                          | `-Inf`, `Inf`            |
| `control`        | Lista dodatkowych ustawień dla optymalizatora (np. `list(maxit = 1000)`).                       | (pusta lista)            |
| `...`            | Dodatkowe argumenty (np. przekazywane do optymalizatora).                                       | –                        |

#### `fitdistcens()` (pakiet **fitdistrplus**)

| Argument         | Opis                                                                                                          | Domyślnie       |
| ---------------- | ------------------------------------------------------------------------------------------------------------- | --------------- |
| `censdata`       | Ramka danych z kolumnami `left`, `right`, `freq`, opisująca przedziały i częstości (dane cenzurowane).        | (brak)          |
| `distr`          | Nazwa rozkładu ciągłego, np. `"norm"`, `"gamma"`, `"lnorm"`.                                                  | (brak)          |
| `start`          | Lista wartości początkowych parametrów (nazwy zgodne z parametrami rozkładu: `mean`/`sd` dla `"norm"`, itp.). | (brak)          |
| `optim.method`   | Metoda optymalizacji: `"BFGS"`, `"Nelder-Mead"`, `"L-BFGS-B"`.                                                | `"Nelder-Mead"` |
| `lower`, `upper` | Ograniczenia dla parametrów (tylko przy `"L-BFGS-B"`).                                                        | `-Inf`, `Inf`   |
| `control`        | Lista dodatkowych opcji optymalizatora.                                                                       | (pusta lista)   |
| `...`            | Inne argumenty, np. `weights`, `fix.arg` (ustawianie wartości niektórych parametrów na stałe).                | –               |

---

## 6. Podsumowanie

* **Funkcje do estymacji punktowej** (`mean`, `median`, `var`, `sd`, `quantile`) są bardzo szybkie i intuicyjne, ale zwracają tylko pojedyncze liczby (lub wektory kwantyli).
* **`t.test()`** pozwala jednocześnie na przeprowadzenie testu istotności względem średniej (lub różnicy średnich) oraz na pobranie przedziału ufności metodą opartą na rozkładzie t‑Studenta.
* **`prop.test()`** analogicznie do `t.test`, ale dla proporcji – zarówno test, jak i przedział ufności.
* **`chisq.test()`** ma dwa tryby: test niezależności (kategoria × kategoria) lub test zgodności (jednowymiarowy rozkład).
* **Przedziały ufności**:

  * Dla średniej – `t.test(..., conf.level = 1 - α)$conf.int`.
  * Dla proporcji – `prop.test(..., conf.level = 1 - α)$conf.int`.
  * Dla wariancji – `varTest(..., conf.level = 1 - α)$conf.int` (pakiet **EnvStats**).
* **MLE**:

  * Gdy mamy pełne, nieskategoryzowane dane, standardowo używa się `mle()` z pakietu **stats4**.
  * Gdy dane są pogrupowane/cenzurowane, używa się `fitdistcens()` z pakietu **fitdistrplus**, przekazując ramkę (`left`, `right`, `freq`) oraz nazwę rozkładu.

---

**Życzę powodzenia na kolokwium i owocnej pracy z R!**
