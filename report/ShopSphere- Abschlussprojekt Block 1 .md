**ShopSphere: Abschlussprojekt**

Block 1 — SQL: Datenvorbereitung

*Zusatzdatei: Schritt-für-Schritt-Analyse der Syntax jeder Abfrage*

# 1.1. Umsatz, Bestellanzahl und durchschnittlicher Bestellwert nach Region und Jahr

Ziel: zeigen, wie das Geschäft nach Region × Jahr wächst – die Grundlage für Grafik 2.4 (Regionsentwicklung) und die KPI-Karten des Dashboards.

SQL-Abfrage  
SELECT c.region,  
       o.order\_year,  
       ROUND(SUM(o.net\_amount), 2\) AS total\_net\_revenue,  
       COUNT(o.order\_id) AS order\_count,  
       ROUND(AVG(o.net\_amount), 2\) AS avg\_order\_value  
FROM orders o  
JOIN customers c ON o.customer\_id \= c.customer\_id  
GROUP BY c.region, o.order\_year  
ORDER BY c.region, o.order\_year;

### Ergebnis (Ausschnitt)

| region | order\_year | total\_net\_revenue | order\_count | avg\_order\_value |
| :---- | :---- | :---- | :---- | :---- |
| North America | 2022 | 80 204,08 | 254 | 315,76 |
| North America | 2023 | 235 736,97 | 848 | 277,99 |
| North America | 2024 | 718 726,68 | 2632 | 273,07 |
| Southeast Asia | 2022 | 12 721,48 | 44 | 289,12 |
| Southeast Asia | 2024 | 613 904,12 | 2029 | 302,56 |

*Business-Fazit: Alle 5 Regionen wachsen von Jahr zu Jahr, jedoch mit unterschiedlicher Geschwindigkeit. Southeast Asia und Latin America wachsen relativ am schnellsten (ausgehend von einer sehr kleinen Basis), während North America und Europe in absoluten Umsatzzahlen am größten bleiben. Die vollständige Tabelle ist in der Datei 1\_1\_region\_year.csv für Tableau gespeichert.*

# 1.2. Top-10-Kunden nach Gesamtausgaben

Ziel: die wertvollsten Kunden für Frage 9 (Top 5 %) identifizieren und verstehen, woher sie kommen (Akquisitionskanal) – Grundlage für die Bindung von VIP-Kunden.

SQL-Abfrage

SELECT o.customer\_id,

       c.region,

       c.acquisition\_channel,

       COUNT(o.order\_id) AS orders\_count,

       ROUND(SUM(o.net\_amount),2) AS total\_spent

FROM orders o

JOIN customers c ON o.customer\_id \= c.customer\_id

GROUP BY o.customer\_id, c.region, c.acquisition\_channel

ORDER BY total\_spent DESC

LIMIT 10;

| customer\_id | region | acquisition\_channel | orders\_count | total\_spent |
| :---- | :---- | :---- | :---- | :---- |
| 12348 | Europe | Influencer | 43 | 30 360,93 |
| 11131 | Europe | Influencer | 41 | 22 546,65 |
| 12723 | North America | Referral | 43 | 19 561,76 |
| 10995 | Europe | Influencer | 32 | 19 496,63 |
| 11387 | Southeast Asia | Influencer | 35 | 18 730,10 |
| 11835 | Southeast Asia | Referral | 31 | 18 114,42 |
| 11046 | Europe | Organic | 37 | 17 960,45 |
| 10837 | North America | Influencer | 27 | 17 822,03 |
| 12965 | Europe | Email | 32 | 16 732,93 |
| 12478 | North America | Referral | 32 | 16 412,50 |

*Business-Fazit: 6 der 10 wertvollsten Kunden kamen über den Influencer-Kanal – ein starkes Signal für Frage 4 (LTV nach Akquisitionskanal).*

# 1.3. Umsatz, Marge und Rücksendeanteil nach Kategorien

Ziel: erkennen, welche Kategorie Volumen liefert und welche echten Gewinn bringt. Grundlage für das Streudiagramm 2.3 und Case B (Block 4).

SQL-Abfrage (mit CTE – einer temporären „Untertabelle“)

WITH cat\_orders AS (

  SELECT DISTINCT oi.category, oi.order\_id, o.is\_returned

  FROM order\_items oi

  JOIN orders o ON oi.order\_id \= o.order\_id

)

SELECT oi.category,

       ROUND(SUM(oi.line\_total), 2\) AS total\_revenue,

       ROUND(AVG(p.margin\_pct), 2\) AS avg\_margin\_pct,

       (SELECT ROUND(100.0 \* SUM(is\_returned) / COUNT(\*), 2\)

        FROM cat\_orders co WHERE co.category \= oi.category) AS return\_rate\_pct

FROM order\_items oi

JOIN products p ON oi.product\_id \= p.product\_id

GROUP BY oi.category

ORDER BY total\_revenue DESC;

### Ergebnis (alle 7 Kategorien)

| category | total\_revenue | avg\_margin\_pct | return\_rate\_pct |
| :---- | :---- | :---- | :---- |
| Electronics | 2 097 901,06 | 12,0 | 15,97 |
| Home & Kitchen | 576 134,75 | 35,0 | 10,27 |
| Sports | 343 114,98 | 30,0 | 8,40 |
| Clothing | 248 601,48 | 45,0 | 16,00 |
| Beauty | 168 624,42 | 55,0 | 9,97 |
| Toys | 140 505,55 | 40,0 | 8,98 |
| Books | 90 757,82 | 25,0 | 8,13 |

*Business-Fazit: Electronics erwirtschaftet 60 % des gesamten Umsatzes, hat aber unter den großen Kategorien die niedrigste Marge (12 %) und den höchsten Rücksendeanteil (16 %) – eine klassische „Volumen-Illusion“ (siehe Frage 6). Beauty ist das Gegenteil: geringer Umsatz, aber 55 % Marge und niedrige Rücksendequote – ein „verborgener Diamant“ (Frage 7).*

# 1.4. Kunden mit überdurchschnittlichen Ausgaben (Unterabfrage)

Ziel: prüfen, wie stark der Unternehmensumsatz von einer Minderheit an Kunden abhängt, die mehr ausgeben als der „typische“ Kunde.

### SQL-Abfrage

WITH customer\_totals AS (

    SELECT 

        customer\_id,

        SUM(net\_amount) AS total\_spent

    FROM orders

    GROUP BY customer\_id

),

avg\_spending AS (

    SELECT AVG(total\_spent) AS avg\_total\_spent

    FROM customer\_totals

),

above\_avg AS (

    SELECT \*

    FROM customer\_totals ct

    CROSS JOIN avg\_spending a

    WHERE ct.total\_spent \> a.avg\_total\_spent

)

SELECT

    COUNT(\*) AS above\_avg\_customers,

    ROUND(SUM(total\_spent), 2\) AS above\_avg\_total\_revenue,

    \-- Gesamtumsatz aller Kunden  \-- \-- Загальний обсяг продажів усіх клієнтів

    (SELECT SUM(total\_spent) FROM customer\_totals) AS grand\_total\_revenue,

    \-- Umsatzanteil   \-- Частка у виручці

    ROUND(

        SUM(total\_spent) \* 100.0 /

        (SELECT SUM(total\_spent) FROM customer\_totals)

    , 2\) AS share\_of\_revenue\_pct,

    \-- Gesamtzahl der Kunden    \-- \-- Загальна кількість клієнтів

    (SELECT COUNT(\*) FROM customer\_totals) AS total\_customers,

     \-- Kundenanteil    \-- Частка клієнтів

    ROUND(

        COUNT(\*) \* 100.0 /

        (SELECT COUNT(\*) FROM customer\_totals)

    , 2\) AS share\_of\_customers\_pct

FROM above\_avg;

### Ergebnis

| above\_avg\_customers | above\_avg\_total\_revenue | grand\_total\_revenue | share\_of\_revenue\_pct | total\_customers | share\_of\_customers\_pct |
| :---- | :---- | :---- | :---- | :---- | :---- |
| 862 | 2 524 753,93 | 3 474 016,03 | 72,68 | 3000 | 28,73 |

*Business-Fazit: Nur 28,7 % der Kunden (862 von 3000\) geben überdurchschnittlich viel aus, erwirtschaften jedoch 72,7 % des gesamten Umsatzes. Das bestätigt eine starke Wertkonzentration (Pareto-Prinzip) und bereitet unmittelbar den Boden für Frage 9 (Top-5%-Kunden).*

# 1.5. ROI der Marketingkanäle

Ziel: berechnen, welcher Marketingkanal das Budget effizienter in Umsatz umwandelt – Schlüsseldaten für Case A (Frage 3).

SQL-Abfrage

SELECT channel,

       ROUND(SUM(budget), 2\) AS total\_budget,

       ROUND(SUM(attributed\_revenue), 2\) AS total\_attributed\_revenue,

       ROUND(SUM(attributed\_revenue) / SUM(budget), 2\) AS roi

FROM marketing

GROUP BY channel

ORDER BY roi DESC;

### Ergebnis (alle 6 Kanäle)

| channel | total\_budget | total\_attributed\_revenue | roi |
| :---- | :---- | :---- | :---- |
| Organic | 20 364,00 | 163 398,00 | 8,0 |
| Email | 37 468,00 | 243 610,00 | 6,0 |
| Influencer | 112 337,00 | 519 453,00 | 4,0 |
| Referral | 73 766,00 | 263 536,00 | 3,0 |
| Social Ads | 286 488,00 | 589 544,00 | 2,0 |
| Paid Search | 450 959,00 | 598 703,00 | 1,0 |

*Business-Fazit: Paid Search erhält das größte Budget (450 959 $ – 45 % aller Marketingausgaben), hat aber den niedrigsten ROI (1,0). Organic und Email sind am effizientesten, erhalten aber nur einen Bruchteil des Budgets. Eine detaillierte Analyse und Empfehlung folgt in den Fragen 3–5.*

**Frage 1\. Logik der Dashboard-Komposition**

Das Dashboard ist nach dem klassischen Trichterprinzip **„vom Allgemeinen zum Konkreten“** (Overview → Zoom → Details) aufgebaut. Die Leserichtung von oben nach unten entspricht dem Gedankengang eines Analysten, der sich Schritt für Schritt dieselben Fragen stellt.

**1\. Titel \+ KPI-Karten (Total Revenue, Order Count, Average Order Value, Return Rate)**

Dies beantwortet die erste Frage jeder Führungskraft: **„Wie steht das Unternehmen insgesamt da?“**

Die vier Kennzahlen bilden den **„Gesundheitscheck“** des Unternehmens für den gesamten Analysezeitraum:

* **3,47 Mio. USD Umsatz**  
* **12.274 Bestellungen**  
* **283 USD durchschnittlicher Bestellwert**  
* **9,77 % Retourenquote**

Ohne zeitlichen oder regionalen Kontext liefern diese Kennzahlen zunächst einen schnellen Überblick über die Gesamtsituation.

**2\. Umsatz-Saisonalität (Liniendiagramm nach Monaten, 2022–2024)**

Der nächste logische Schritt lautet: **„Wann passiert das?“**

Hier wird die zeitliche Dimension eingeführt. Direkt unter der Überschrift wird bereits die wichtigste Erkenntnis zusammengefasst: **Der Umsatz erreicht seinen Höhepunkt im November und Dezember und fällt in den Wintermonaten deutlich ab.**

Damit wird die wichtigste zeitliche Entwicklung sofort sichtbar.

**3\. Marketing: Budget vs. ROI \+ Regionen im Wandel**

Nun folgt die Frage: **„Warum passiert das und wo wächst das Unternehmen?“**

Diese Ebene erklärt die wichtigsten Wachstumstreiber:

* Links wird gezeigt, welche Marketingkanäle den höchsten Return on Investment (ROI) erzielen.  
* Rechts wird dargestellt, welche Regionen zwischen 2022 und 2024 das stärkste Wachstum verzeichnen.

Die beiden Diagramme sind bewusst nebeneinander angeordnet: Das linke erklärt, **woher die Kunden kommen**, das rechte zeigt, **wo das Umsatzwachstum geografisch stattfindet**.

**4\. Kategorien: Volumen vs. Rentabilität**

Die letzte Ebene beantwortet die Frage: **„Welche konkreten Maßnahmen sollten ergriffen werden?“**

Das Quadranten-Bubble-Diagramm zeigt die Produktkategorien anhand von Verkaufsvolumen und Gewinnmarge. Durch die Aufteilung in vier Quadranten lassen sich Wachstumschancen und Problemfelder schnell erkennen.

Im Gegensatz zu den vorherigen Diagrammen beantwortet dieses nicht die Frage **„Was ist passiert?“**, sondern **„Wo sollten Maßnahmen ergriffen werden?“**

Die Reihenfolge der Visualisierungen wurde bewusst gewählt, da jede Ebene den Fokus weiter eingrenzt und zusätzliche Informationen liefert, ohne bereits Gezeigtes zu wiederholen:

* zuerst der Gesamtüberblick (**Wie viel?**),  
* danach die zeitliche Entwicklung (**Wann?**),  
* anschließend Ursachen und regionale Unterschiede (**Warum? Wo?**),  
* und schließlich konkrete Handlungsempfehlungen (**Was sollte optimiert werden?**).

Genau in dieser Reihenfolge sucht der Betrachter eines Dashboards normalerweise nach Antworten.

---

**Frage 2\. Drei Erkenntnisse in den ersten 30 Sekunden**

**1\. Das Unternehmen erzielt hohe Umsätze, weist jedoch eine vergleichsweise hohe Retourenquote auf.**

Mit **3,47 Mio. USD Umsatz**, **12.274 Bestellungen** und einem durchschnittlichen Bestellwert von rund **283 USD** wirkt das Geschäft insgesamt erfolgreich.

Allerdings liegt die **Retourenquote bei 9,77 %**. Das bedeutet, dass nahezu jede zehnte Bestellung zurückgesendet wird. Für den E-Commerce ist dies ein deutlicher Hinweis auf mögliche Probleme, beispielsweise bei der Produktqualität, den Größenangaben oder den Produktbeschreibungen.

**2\. Der Umsatz ist stark saisonabhängig.**

Sowohl im Untertitel als auch im Liniendiagramm ist deutlich zu erkennen, dass der Umsatz jedes Jahr während der **Black-Friday- und Weihnachtszeit** seinen Höchststand erreicht und in den Wintermonaten anschließend deutlich zurückgeht.

Diese Erkenntnis ist besonders wichtig für die Planung von Lagerbeständen und Marketingbudgets.

**3\. Die Regionen entwickeln sich unterschiedlich schnell.**

Das Diagramm **„Regionen im Wandel“** zeigt, dass die türkise und die grüne Region bis 2024 auf **718.727** bzw. **613.904 USD** Umsatz wachsen.

Die orange und die rote Region erreichen dagegen lediglich **281.391** bzw. **321.391 USD**.

Damit hat sich der Abstand zwischen der stärksten und der schwächsten Region innerhalb von drei Jahren mehr als verdoppelt. Dies deutet darauf hin, dass Investitionen und Ressourcen künftig gezielter auf die wachstumsstarken Regionen verteilt werden sollten.

