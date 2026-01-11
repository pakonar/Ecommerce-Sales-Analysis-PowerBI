# Dokumentacja Miar DAX (DAX Measures Documentation)

Poniższy dokument zawiera kompletny zestaw miar języka DAX wykorzystanych w projekcie. Kod został podzielony na obszary analityczne.

## 1. Analiza i Lojalność Klientów (Customer Analysis & Loyalty)

**Returning Customers (Powracający Klienci)**
Oblicza liczbę klientów, którzy dokonali zakupu w bieżącym miesiącu ORAZ dokonali zakupu kiedykolwiek wcześniej.
```
Returning Customers =
VAR CurrentCustomers = VALUES('Online Retail'[CustomerID])
-- Tworzy listę unikalnych ID klientów, którzy kupili coś w OBECNIE wybranym okresie (np. w Listopadzie 2011).

VAR CurrentMonthStart = MIN('Calendar'[Date])
-- Pobiera pierwszą datę z obecnie wybranego okresu (służy jako punkt odcięcia "przeszłości").

VAR PastCustomers =
    CALCULATETABLE(
        -- Tworzy wirtualną tabelę w zmienionym kontekście filtrów.
        VALUES('Online Retail'[CustomerID]),
        -- Pobiera listę unikalnych klientów...
        ALL('Calendar'),
        -- ...ale ignoruje wszystkie filtry nałożone na kalendarz (patrzy na całą historię)...
        'Calendar'[Date] < CurrentMonthStart
        -- ...pod warunkiem, że data zakupu jest wcześniejsza niż początek obecnego okresu.
    )

RETURN
COUNTROWS(
    -- Zlicza wiersze w tabeli wynikowej.
    INTERSECT(CurrentCustomers, PastCustomers)
    -- Funkcja INTERSECT zwraca część wspólną dwóch list: klientów, którzy są na liście "Obecni" I na liście "Przeszli".
)
```
**One-time Buyers (Jednorazowi Klienci)**
Liczy klientów, którzy w całej historii sklepu zrobili zakupy tylko raz.
```
One-time Buyers =
COUNTROWS(
    -- Zlicza wiersze tabeli, która powstanie w wyniku funkcji FILTER.
    FILTER(
        VALUES('Online Retail'[CustomerID]),
        -- Iteruje (przechodzi) przez każdego unikalnego klienta widocznego w obecnym kontekście.

        CALCULATE(
            DISTINCTCOUNT('Online Retail'[InvoiceNo]),
            -- Liczy unikalne numery faktur dla danego klienta...
            ALL('Calendar')
            -- ...ale ignoruje wybrany okres czasu! Sprawdza całą historię życia klienta.
        ) = 1
        -- Warunek: Jeśli liczba faktur w całej historii wynosi dokładnie 1, zachowaj tego klienta.
    )
)
```
**New Customers (Nowi Klienci)**
Liczba klientów, którzy kupili coś w tym okresie, ale nie są powracający (czyli kupują pierwszy raz).
```
New Customers =
[Total Customers] - [Returning Customers]
```
Retention Rate % (Wskaźnik Retencji)
Jaki procent obecnych klientów to klienci lojalni (powracający).
```
Retention Rate % =
DIVIDE([Returning Customers], [Total Customers], 0)
-- Dzieli liczbę powracających przez ogólną liczbę klientów.
```
Lifetime Retention Rate % (Wskaźnik Powracających Klientów - Całkowity)
Określa, jaki procent klientów widocznych w danym okresie to osoby, które w całej historii współpracy z firmą dokonały więcej niż jednego zakupu. Miara ta rozwiązuje problem analizy w krótkich oknach czasowych, patrząc na "cykl życia" klienta.
```
Lifetime Retention Rate % =
VAR TotalCust = [Total Customers]
-- Pobiera liczbę wszystkich klientów aktywnych w wybranym kontekście (np. w danym roku).

VAR OneTimeCust = [One-time Buyers]
-- Pobiera liczbę klientów, którzy w CAŁEJ historii (dzięki funkcji ALL w mierze bazowej) kupili tylko raz.

VAR RepeatCust = TotalCust - OneTimeCust
-- Oblicza resztę, czyli klientów, którzy nie są jednorazowi (zatem musieli kupić min. 2 razy).

RETURN
DIVIDE(RepeatCust, TotalCust, 0)
-- Dzieli liczbę powracających przez ogół klientów, dając procent lojalnej bazy (np. 70%).
```
Purchase Frequency (Częstotliwość Zakupów)
Mówi nam, ile średnio transakcji dokonuje jeden klient. Jest to kluczowy wskaźnik lojalności – im wyższy wynik, tym silniejszy nawyk zakupowy mają klienci.
```
Purchase Frequency =
DIVIDE(
    [Transaction Count],
    -- Liczba unikalnych transakcji sprzedaży (faktur).
    [Total Customers],
    -- Liczba unikalnych klientów, którzy dokonali tych zakupów.
    0
    -- Zabezpieczenie przed dzieleniem przez zero.
)
```
2. Analiza Sprzedaży i Produktów (Sales Performance)
% of GT Net Revenue (Udział w Całkowitym Przychodzie)
Pokazuje, jaki procent całkowitej sprzedaży firmy stanowi sprzedaż danego produktu (lub kategorii). Kluczowe do analizy Pareto.
```
% of GT Net Revenue =
VAR CurrentProductRevenue = [Net Revenue (Inc. VAT)]
-- Przypisuje do zmiennej wartość sprzedaży dla bieżącego wiersza (np. konkretnego produktu na wykresie).

VAR TotalGlobalRevenue =
    CALCULATE(
        [Net Revenue (Inc. VAT)],
        -- Oblicza tę samą miarę przychodu...
        REMOVEFILTERS('Online Retail'[StockCode], 'Online Retail'[Description])
        -- ...ale usuwa filtry z kodu i opisu produktu. Dzięki temu otrzymujemy sumę sprzedaży WSZYSTKICH produktów (mianownik).
    )

RETURN
DIVIDE(CurrentProductRevenue, TotalGlobalRevenue)
-- Bezpieczne dzielenie: (Sprzedaż Produktu) / (Sprzedaż Całkowita).
```
Average Order Value (AOV - Średnia Wartość Zamówienia)
Średnia wartość koszyka zakupowego (tylko dla transakcji sprzedaży, bez zwrotów).
```
Average Order Value =
VAR TotalRevenueSales =
    CALCULATE(
        [Net Revenue (Inc. VAT)],
        -- Bierze przychód netto...
        'Online Retail'[Quantity] > 0
        -- ...ale tylko dla wierszy, gdzie ilość jest dodatnia (czyli to sprzedaż, a nie zwrot).
    )

VAR TotalInvoicesSales =
    CALCULATE(
        DISTINCTCOUNT('Online Retail'[InvoiceNo]),
        -- Liczy unikalne numery faktur...
        'Online Retail'[Quantity] > 0
        -- ...również tylko dla transakcji sprzedaży (żeby nie liczyć faktur korygujących jako zamówień).
    )

RETURN
DIVIDE(TotalRevenueSales, TotalInvoicesSales, 0)
-- Dzieli przychód przez liczbę zamówień. Jeśli brak zamówień, zwraca 0.
```
Average Units per Order (Średnia liczba sztuk w zamówieniu)
Ile jest średnio produktów w koszyku.
```
Average Units per Order =
VAR TotalUnitsSales =
    CALCULATE(
        SUM('Online Retail'[Quantity]),
        -- Sumuje ilość sztuk...
        'Online Retail'[Quantity] > 0
        -- ...tylko dla sprzedaży (ignoruje ujemne ilości ze zwrotów).
    )

VAR TotalInvoicesSales =
    CALCULATE(
        DISTINCTCOUNT('Online Retail'[InvoiceNo]),
        -- Liczy liczbę zamówień (tak samo jak w AOV)...
        'Online Retail'[Quantity] > 0
    )

RETURN
DIVIDE(TotalUnitsSales, TotalInvoicesSales, 0)
-- Dzieli liczbę sztuk przez liczbę zamówień.
```
Gross Revenue (Przychód Brutto)
Całkowita wartość sprzedaży przed odjęciem zwrotów.
```
Gross Revenue =
CALCULATE(
    SUM('Online Retail'[Amount]),
    -- Sumuje kolumnę z kwotami...
    'Online Retail'[TransactionType] = "Sale"
    -- ...biorąc pod uwagę tylko wiersze oznaczone jako "Sale" (Sprzedaż).
)
```
3. Analiza Zwrotów (Returns Analysis)

Return Value (Wartość Zwrotów)
Łączna kwota zwróconych towarów.
```
Return Value =
CALCULATE(
    SUM('Online Retail'[Amount]),
    -- Sumuje kolumnę z kwotami...
    'Online Retail'[TransactionType] = "Return"
    -- ...biorąc pod uwagę tylko wiersze oznaczone jako "Return" (Zwrot).
)
```
Return Rate % (Wskaźnik Zwrotów)
Jaki procent przychodu brutto tracimy przez zwroty.
```
Return Rate % =
DIVIDE(
    ABS([Return Value]),
    -- Bierze wartość bezwzględną zwrotów (zamienia liczbę ujemną na dodatnią, np. -100 na 100)...
    [Gross Revenue],
    -- ...i dzieli przez przychód brutto.
    0
    -- W razie błędu zwraca 0.
)
```
4. Podstawowe Liczniki (Base Measures)
Transaction Count (Liczba Transakcji)
```
Transaction Count =
CALCULATE(
    DISTINCTCOUNT('Online Retail'[InvoiceNo]),
    -- Liczy unikalne numery faktur...
    'Online Retail'[TransactionType] = "Sale"
    -- ...tylko dla sprzedaży (ignoruje korekty).
)
```
**Total Customers (Liczba Klientów)**
```
Total Customers =
DISTINCTCOUNT('Online Retail'[CustomerID])
-- Liczy unikalne ID klientów w bieżącym kontekście.
```
**Total Products (Liczba Produktów)**
```
Total Products =
DISTINCTCOUNTNOBLANK('Online Retail'[StockCode])
-- Liczy unikalne kody produktów, ignorując puste wartości (BLANK).
```
##5. Słownik użytych funkcji DAX
CALCULATE: Najważniejsza funkcja w DAX. Pozwala zmienić kontekst obliczeń (np. "policz sprzedaż, ale tylko dla typu 'Sale'").
CALCULATETABLE: To samo co CALCULATE, ale zwraca całą tabelę, a nie jedną liczbę. Użyte do stworzenia listy PastCustomers.
VAR ... RETURN: Zmienne. Pozwalają zapisać wynik obliczenia, nazwać go i użyć później. Poprawiają czytelność i wydajność kodu.
ALL: Funkcja "usuwająca filtry". Mówi: "Ignoruj to, co użytkownik wybrał we fragmentatorze (np. datę)".
REMOVEFILTERS: Nowocześniejsza, bardziej precyzyjna wersja ALL służąca do usuwania filtrów z konkretnych kolumn (użyta w % of GT).
VALUES: Zwraca listę unikalnych wartości z kolumny, widocznych w obecnym filtrze.
INTERSECT: Funkcja zbiorów. Porównuje dwie listy i zwraca tylko te elementy, które występują w obu. Kluczowa do analizy powracających klientów.
DIVIDE: "Bezpieczne dzielenie". Chroni przed błędem "dzielenie przez zero" (np. gdy w danym dniu nie było sprzedaży).
ABS (Absolute): Wartość bezwzględna. Zamienia liczby ujemne na dodatnie (przydatne przy zwrotach).
DISTINCTCOUNT / DISTINCTCOUNTNOBLANK: Zlicza unikalne wystąpienia. Wersja NOBLANK jest lepsza, gdy w danych mogą być puste wiersze.
