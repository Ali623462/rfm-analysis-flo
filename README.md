
# 📊 RFM-Kundenanalyse mit FLO-Daten

Dieses Projekt verwendet eine RFM-Analyse (Recency, Frequency, Monetary), um Kunden von FLO, einem türkischen Einzelhändler, zu segmentieren. Ziel ist es, Kundenverhalten besser zu verstehen und gezielte Marketingstrategien zu entwickeln.

---

## 🔧 Methodik

### RFM-Metriken:

- **Recency**: Anzahl der Tage seit dem letzten Kauf
- **Frequency**: Anzahl der Käufe
- **Monetary**: Gesamtumsatz des Kunden

---

## 📥 Datenimport & Vorbereitung

```python
import pandas as pd
df = pd.read_csv("flo_data_20k.csv")
df["order_num_total"] = df["order_num_total_ever_online"] + df["order_num_total_ever_offline"]
df["customer_value_total"] = df["customer_value_total_ever_offline"] + df["customer_value_total_ever_online"]
date_columns = df.columns[df.columns.str.contains("date")]
df[date_columns] = df[date_columns].apply(pd.to_datetime)
```

---

## 📅 RFM-Tabelle erstellen

```python
import datetime as dt
analysis_date = dt.datetime(2021, 6, 1)

rfm = pd.DataFrame()
rfm["customer_id"] = df["master_id"]
rfm["recency"] = (analysis_date - df["last_order_date"]).dt.days
rfm["frequency"] = df["order_num_total"]
rfm["monetary"] = df["customer_value_total"]
```

---

## 🔢 RFM-Scores berechnen

```python
rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1])
rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1, 2, 3, 4, 5])
rfm["monetary_score"] = pd.qcut(rfm['monetary'], 5, labels=[1, 2, 3, 4, 5])

rfm["RF_SCORE"] = rfm["recency_score"].astype(str) + rfm["frequency_score"].astype(str)
rfm["RFM_SCORE"] = rfm["recency_score"].astype(str) + rfm["frequency_score"].astype(str) + rfm["monetary_score"].astype(str)
```

---

## 🧠 Kundensegmentierung

```python
seg_map = {
    r'[1-2][1-2]': 'hibernating',
    r'[1-2][3-4]': 'at_Risk',
    r'[1-2]5': 'cant_loose',
    r'3[1-2]': 'about_to_sleep',
    r'33': 'need_attention',
    r'[3-4][4-5]': 'loyal_customers',
    r'41': 'promising',
    r'51': 'new_customers',
    r'[4-5][2-3]': 'potential_loyalists',
    r'5[4-5]': 'champions'
}
rfm['segment'] = rfm['RF_SCORE'].replace(seg_map, regex=True)
```

---

## 📊 Segmentübersicht (Beispiel)

| Segment             | Recency ⌀ | Frequency ⌀ | Umsatz ⌀ |
|---------------------|-----------|-------------|-----------|
| champions           | 17 Tage   | 8,93 Käufe   | 1406,63 ₺  |
| loyal_customers     | 82 Tage   | 8,37 Käufe   | 1216,82 ₺  |
| at_Risk             | 241 Tage  | 4,47 Käufe   | 646,61 ₺   |
| hibernating         | 247 Tage  | 2,39 Käufe   | 366,27 ₺   |

---

## 🎯 Zielgruppenexport

### 🎯 1. Treue Kunden – interessiert an „KADIN“ (Damenmode)

```python
target_segments_customer_ids = rfm[rfm["segment"].isin(["champions", "loyal_customers"])]["customer_id"]
cust_ids = df[(df["master_id"].isin(target_segments_customer_ids)) & 
              (df["interested_in_categories_12"].str.contains("KADIN"))]["master_id"]
cust_ids.to_csv("yeni_marka_hedef_müşteri_id.csv", index=False)
```

### 🎯 2. Passive Männer-/Kinderkunden – für Rabattaktionen

```python
target_segments_customer_ids = rfm[rfm["segment"].isin(["cant_loose", "hibernating", "new_customers"])]["customer_id"]
cust_ids = df[(df["master_id"].isin(target_segments_customer_ids)) & 
              ((df["interested_in_categories_12"].str.contains("ERKEK")) |
               (df["interested_in_categories_12"].str.contains("COCUK")))]["master_id"]
cust_ids.to_csv("indirim_hedef_müşteri_ids.csv", index=False)
```

---

## 📌 Fazit

- Durch die RFM-Analyse konnten Kunden in sinnvolle Segmente eingeteilt werden.
- Diese Segmentierung ermöglicht gezielte Kampagnen für verschiedene Zielgruppen.
- Datengetriebenes Marketing kann Kundenbindung verbessern und Reaktivierungen fördern.

---

## 📚 Verwendete Bibliotheken

- pandas
- datetime
