# Executive Sales Dashboard 

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-00758F?style=for-the-badge&logo=microsoft&logoColor=white)
![Data Analysis](https://img.shields.io/badge/Data_Analysis-Success?style=for-the-badge)

## 1. Cel projektu
Projekt ma na celu opracowanie interaktywnego systemu analitycznego, który przekształca surowe dane sprzedażowe w czytelne informacje biznesowe. System umożliwia podejmowanie trafniejszych, opartych na danych decyzji strategicznych w obszarze przychodów, rynków, klientów oraz rentowności produktów.

---

## 2. Źródło i struktura danych
Dane pochodzą z **Online Retail Dataset**, udostępnionego przez UCI Machine Learning Repository. Zestaw obejmuje transakcje sprzedaży sklepu internetowego w różnych krajach w okresie 2010–2011.

**Kluczowe kolumny:**
*   `InvoiceNo` – numer faktury
*   `StockCode` – kod produktu
*   `Description` – nazwa/opis produktu
*   `Quantity` – liczba sprzedanych sztuk (ujemne wartości oznaczają zwroty)
*   `InvoiceDate` – data i godzina transakcji
*   `UnitPrice` – cena jednostkowa
*   `CustomerID` – identyfikator klienta
*   `Country` – kraj klienta

---

## 3. Wykorzystane narzędzia i techniki
W projekcie zastosowano **Microsoft Power BI** z naciskiem na przygotowanie danych (ETL), aby zapewnić rzetelne analizy.

### A. Profilowanie i korekta danych
*   Analiza jakości danych i identyfikacja braków (null-e) w celu zapewnienia spójności.
*   Korekta typów danych (np. waluta, liczby całkowite).
*   Ujednolicenie nazw produktów (*Capitalize Each Word*) i rynków.

### B. Czyszczenie i filtrowanie (Data Cleansing)
*   Usunięcie rekordów technicznych (korekty księgowe, opłaty bankowe, testy).
*   Obsługa braków danych: niezalogowani klienci oznaczeni jako „Guest”.
*   Dynamiczne filtrowanie wierszy w Power Query.

### C. Obsługa Outlierów (Wartości Skrajnych)
*   Wykrycie i usunięcie nierealnych transakcji (np. zakupy powyżej 80 tys. sztuk), co zapobiegło zafałszowaniu średniej wartości koszyka (AOV).
*   Zachowanie `StockCode` jako głównego identyfikatora produktów.

### D. Modelowanie Danych i DAX
*   Stworzenie kolumn obliczeniowych: `Revenue = Quantity * UnitPrice`.
*   Przygotowanie zaawansowanych miar DAX (m.in. analiza kohortowa, Pareto).

**[Zobacz pełną dokumentację miar DAX](DAX_Documentation.md)**

---

## 4. Analiza Biznesowa – Pytania, Wnioski i Rekomendacje

### I. Kondycja Finansowa i Sezonowość
![Sales Overview Dashboard](SalesOverview.png)

**a. Pytania Biznesowe:**
*   Jak kształtują się przychody w czasie i czy widoczna jest sezonowość sprzedaży?
*   Czy spadek sprzedaży w grudniu 2011 jest realnym trendem, czy wynika z niepełnych danych?

**b. Wizualizacja:**
*   *Sales Overview* -> **Area Chart** (Gross & Net Revenue by Month)

**c. Wnioski (Insights):**
*   Firma wykazuje silny trend wzrostowy w IV kwartale (Q4), z wyraźnym pikiem sprzedaży w **listopadzie** (przygotowania do świąt).
*   Widoczny na wykresie drastyczny spadek w grudniu 2011 jest pozorny i wynika z zakończenia zbioru danych w dniu 9 grudnia (niepełny miesiąc).

**d. Rekomendacja:**
*   **Planowanie Logistyki:** Kluczowe działania mające na celu **zapewnienie pełnych stanów magazynowych** muszą zakończyć się w październiku, aby obsłużyć listopadowy szczyt sprzedażowy.
*   **Raportowanie:** W raportach zarządczych należy wyraźnie oznaczać niepełne okresy, aby uniknąć błędnych decyzji biznesowych opartych na fałszywych spadkach.

---

### II. Efektywność Produktowa i Zasada Pareto
![Product Performance Dashboard](Product Performance - strona 3.png)

**a. Pytania Biznesowe:**
*   Czy występuje koncentracja przychodów (Zasada Pareto)?
*   Które bestsellery generują ukryte koszty poprzez wysoki wskaźnik zwrotów?
*   Które produkty należą do "Długiego Ogona" i generują koszty magazynowania?

**b. Wizualizacja:**
*   *Product Performance* -> **Bar Chart** (Top 50 Products)
*   *Product Performance* -> **Scatter Plot** (Revenue vs. Volume Analysis)

**c. Wnioski (Insights):**
*   **Zasada Pareto:** Analiza potwierdza klasyczny rozkład – **20% produktów (763 produkty) generuje aż 77% całkowitego przychodu**. Pozostałe ~3000 produktów to tzw. "Długi Ogon".
*   **Wyzwanie Jakościowe:** Mimo że "White Hanging Heart" sprzedał się w liczbie **35 313** egzemplarzy (znacznie więcej niż lider rankingu, który ma **13 022 szt.**), jego rentowność cierpi przez jakość.
                             Wskaźnik zwrotów na poziomie 6.23% (przy średniej rynkowej 2.40%) oznacza, że tysiące sztuk wracają do magazynu, generując straty.
*   **Lider Wartości:** Produkt *"Regency Cakestand 3 Tier"* jest liderem przychodowym (£164k), mimo mniejszego wolumenu niż inne bestsellery, co oznacza wysoką marżę jednostkową.

**d. Rekomendacja:**
*   **Audyt Jakości:** Weryfikacja partii produktu *"White Hanging Heart"* oraz sposobu jego pakowania.
*   **Optymalizacja Magazynu:** Wdrożenie polityki ciągłej dostępności dla top 20% produktów oraz rozważenie **wycofania ze sprzedaży** produktów z końcówki "Długiego Ogona".

---

### III. Profil i Lojalność Klienta
![Customer Behavior Dashboard](Customer Behavior & Market Insights - strona 2.png)

**a. Pytania Biznesowe:**
*   Czy wzrost firmy opiera się na pozyskiwaniu nowych klientów, czy na utrzymaniu obecnych?
*   Jaki jest profil klienta (B2B vs B2C) biorąc pod uwagę częstotliwość zakupów i wartość koszyka?

**b. Wizualizacja:**
*   *Customer Behavior* -> **Stacked Column Chart** (Returning vs New Customers)
*   *Customer Behavior* -> **KPI Cards** (Purchase Frequency, AOV)
*   *Customer Behavior* -> **Pie Chart** (Repeat Buyers)

**c. Wnioski (Insights):**
*   **Wysoka Lojalność:** Baza klientów jest bardzo stabilna. Wskaźnik `Lifetime Retention Rate` wynosi blisko **70%**, a wykres czasowy pokazuje, że w 2011 roku wzrost przychodów był napędzany głównie przez klientów powracających.
*   **Charakterystyka B2B:** Średnia częstotliwość zakupów to **4.53** transakcji na klienta, a średnia wartość koszyka (AOV) to **£506**. Sugeruje to dominację klientów hurtowych/biznesowych, a nie detalicznych.

**d. Rekomendacja:**
*   **Zmiana Strategii:** Przesunięcie budżetu marketingowego z **pozyskiwania nowych klientów** na **utrzymanie relacji z obecnymi**.
*   **Program Lojalnościowy:** Wdrożenie progów rabatowych opartych na rocznym wolumenie zakupów, aby nagradzać lojalnych klientów B2B i zachęcać do konsolidacji zamówień.

---

### IV. Strategia Rynkowa i Geografia
![Sales Overview Dashboard](Sales Overview - strona 1.png)
![Customer Behavior Dashboard](Customer Behavior & Market Insights - strona 2.png)

**a. Pytania Biznesowe:**
*   Czy firma jest uzależniona od jednego rynku (Ryzyko Koncentracji)?
*   Które rynki zagraniczne charakteryzują się najwyższą wartością koszyka?

**b. Wizualizacja:**
*   *Sales Overview* -> **Bar Chart** (Top 5 Markets)
*   *Customer Behavior* -> **Scatter Plot** (AOV vs Transaction Count)
*   *Customer Behavior* -> **Tree Map** (Top 10 Markets by AOV)

**c. Wnioski (Insights):**
*   **Dominacja Rynku Lokalnego:** Wielka Brytania generuje aż **85%** całkowitego przychodu. Tak silne uzależnienie od jednego kraju stanowi istotne ryzyko dla stabilności firmy. Ewentualne problemy na tym rynku miałyby krytyczny wpływ na wyniki całej firmy.
*   **Rynki Premium:** Rynki takie jak Australia, Japonia i Holandia charakteryzują się znacznie wyższą średnią wartością koszyka (**AOV > £2000**) niż UK (~£400).

**d. Rekomendacja:**
*   **Strategia dla Rynków Zagranicznych:** Traktowanie rynków eksportowych jako segmentu "Premium Wholesale". Wysokie koszty wysyłki nie są tam barierą, ponieważ są amortyzowane przez bardzo dużą wartość zamówień.
*   **Dywersyfikacja:** Ekspansja na rynki UE (np. Holandia - nr 2 w Europie) w celu zmniejszenia zależności od UK.

---
## 5. Kontakt
[www.linkedin.com/in/patryk-konarzewski-1a1b45182]
