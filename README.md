<img src="https://github.com/user-attachments/assets/93640f49-d8fb-45cf-925e-6b7075f83927#gh-light-mode-only" alt="Light Mode" />
<img src="https://github.com/user-attachments/assets/71380a65-3419-41f4-ba29-2b74c7e6a66b#gh-dark-mode-only" alt="Dark Mode" />

---

# ML queries
The Nameless Analytics ML query is a set of BigQuery ML query that makes marketing predictions and analysis.

For an overview of how Nameless Analytics works [start from here](https://github.com/tommasomoretti/nameless-analytics/).

Table of contents:
- Tables
  - [Events raw table](#events-raw-table)
  - [Users raw changelog table](#users-raw-changelog-table)
  - [Dates table](#dates-table)
 

# Lorem ipsum
## Previsione del churn (abbandono cliente)
Modello: LOGISTIC_REG o BOOSTED_TREE_CLASSIFIER

Cosa fa: Predice la probabilità che un cliente abbandoni il servizio o smetta di acquistare.

Input tipici:
- Frequenza di acquisto
- Ultimo acquisto
- Utilizzo del servizio
- Supporto ricevuto
- Interazioni recenti con campagne

Utilità:
- Segmentare i clienti a rischio
- Attivare campagne di retention automatiche

```sql
# NAMELESS ANALYTICS

# Create training dataset
DECLARE training_start_date DATE;
DECLARE training_end_date DATE;
DECLARE churn_days_from_last_visit INT64;
DECLARE churn_days_from_last_purchase INT64;

set training_start_date = '2025-01-01'; -- Set start date for training dataset
set training_end_date = '2025-06-30'; -- Set end date for training dataset
set churn_days_from_last_visit = 0; -- Set days_from_last_visit churn threshold for training dataset
set churn_days_from_last_purchase = 1; -- Set days_from_last_purchase churn threshold for training dataset

CREATE OR REPLACE TABLE `tom-moretti.nameless_analytics_ml.training_data_users_churn`
  PARTITION BY user_acquisition_date
  AS (
    SELECT
      user_date AS user_acquisition_date,
      client_id, 
      user_id,
      sessions,
      days_from_last_visit,
      days_from_last_purchase,
      session_duration_sec,
      purchase,
      purchase_revenue,
      refund,
      refund_revenue,
      CASE
        when days_from_last_visit >= churn_days_from_last_visit and days_from_last_purchase >= churn_days_from_last_purchase then 'Yes'
        else 'No'
      END is_churned
    FROM `tom-moretti.nameless_analytics.users`(training_start_date, training_end_date)
    -- WHERE is_customer = 'Customer'
  );

# -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Create model
CREATE OR REPLACE MODEL `tom-moretti.nameless_analytics_ml.ml_model_users_churn` OPTIONS(
  model_type = 'LOGISTIC_REG', -- LOGISTIC_REG or BOOSTED_TREE_CLASSIFIER,
  input_label_cols = ['is_churned'],
  auto_class_weights = TRUE
) AS (
  SELECT
    *
  FROM `tom-moretti.nameless_analytics_ml.training_data_users_churn`
);


# Evaluate model
SELECT
  *
FROM ML.EVALUATE(
  MODEL `tom-moretti.nameless_analytics_ml.ml_model_users_churn`, (
    SELECT
      *
    FROM `tom-moretti.nameless_analytics_ml.training_data_users_churn`
  )
);

# -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Predict
create or replace table function `tom-moretti.nameless_analytics_ml.users_churn` (start_date DATE, end_date DATE) as (
  SELECT
    * EXCEPT (predicted_is_churned_probs, predicted_is_churned),
    predicted_is_churned as is_churned
  FROM ML.PREDICT(
    MODEL `tom-moretti.nameless_analytics_ml.ml_model_users_churn`, (
      SELECT
        user_date AS user_acquisition_date,
        client_id, 
        user_id,
        sessions,
        days_from_last_visit,
        days_from_last_purchase,
        session_duration_sec,
        purchase,
        purchase_revenue,
        refund,
        refund_revenue,
      FROM `tom-moretti.nameless_analytics.users`(start_date, end_date)
    )
  )
);
```


## Segmentazione clienti
Modello: KMEANS

Cosa fa: Raggruppa i clienti in cluster basati su caratteristiche simili.

Input tipici:
- Spesa media
- Frequenza di acquisto
- Interazioni con il sito/app
- Localizzazione, età, canale preferito

Utilità:
- Personalizzazione di campagne
- Creazione di buyer personas

```sql
```


## Previsione delle vendite / forecasting
Modello: ARIMA_PLUS o ML.FORECAST

Cosa fa: Prevede il valore futuro di una metrica (es. vendite, revenue, iscrizioni) su base temporale.

Input tipici:
- Serie temporali delle vendite
- Eventi stagionali/promozioni
- Indicatori economici o click pubblicitari

Utilità:
- Ottimizzare scorte, promozioni, marketing budget
- Pianificazione data-driven

```sql
```


## Ottimizzazione delle campagne pubblicitarie
Modello: LINEAR_REG, BOOSTED_TREE_REGRESSOR

Cosa fa: Prevede il ritorno di una campagna, es. conversioni o ROI.

Input tipici:
- Budget speso
- Canale (Google Ads, Meta, Email)
- Target demografico
- Tempo di esposizione

Utilità:
- Trovare il canale più performante
- Prevedere il ritorno su una campagna prima di lanciarla

```sql
```


## Raccomandazioni prodotto
Modello: Custom (via MATRIX_FACTORIZATION o esportando embedding)

Cosa fa: Suggerisce prodotti basati su comportamento passato (tipo Netflix/Amazon).

Input tipici:
- Cronologia acquisti o visite
- Rating (se disponibili)
- Tempo trascorso su prodotto

Utilità:
- Aumentare AOV (average order value)
- Personalizzazione esperienze

```sql
```


## Predizione della probabilità di conversione
Modello: LOGISTIC_REG, BOOSTED_TREE_CLASSIFIER

Cosa fa: Stima la probabilità che un utente compia una conversione (es. acquisto, iscrizione).

Input tipici:
- Click su annunci
- Tempo sulla pagina
- Tipo di dispositivo
- Fonte di traffico

Utilità:
- Ottimizzazione funnel
- Retargeting personalizzato

```sql
```


## Analisi di sentiment su recensioni o feedback
Modello: via REMOTE_MODEL o AutoML (testo)

Cosa fa: Analizza testi (recensioni, email, commenti) per determinare sentiment positivo/negativo/neutro.

Input tipici:
- Recensioni prodotto
- Risposte sondaggi
- Commenti su social o moduli

Utilità:
- Comprendere come i clienti percepiscono il brand
- Intervenire in caso di feedback negativo

```sql
```


## Test A/B e inferenze causali
Modello: ML.TRIAL_RESULT (usato con ML.HYPERPARAMETERS)

Cosa fa: Aiuta a capire quale variante funziona meglio tra due o più opzioni (es. email A vs B).

Utilità:
- Ottimizzare messaggi, landing page
- Automatizzare test su ampie basi utenti

```sql
```


## Lookalike modeling
Modello: LOGISTIC_REG, BOOSTED_TREE_CLASSIFIER

Cosa fa: Trova nuovi utenti simili a un gruppo che ha già convertito (o che ha un comportamento desiderato).

Input tipici:
- Dati dei clienti "ideali"
- Comportamenti utenti anonimi
- Demografica, comportamento navigazione

Utilità:
- Espansione audience per pubblicità mirata

```sql
```

---

Reach me at: [Email](mailto:hello@tommasomoretti.com) | [Website](https://tommasomoretti.com/?utm_source=github.com&utm_medium=referral&utm_campaign=nameless_analytics) | [Twitter](https://twitter.com/tommoretti88) | [Linkedin](https://www.linkedin.com/in/tommasomoretti/)
