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
-- CREATE TRAINING DATA
# time interval: last 12 months + current month
# training_data: last 12 months excluse current month
# evaluate_data: current_month

DECLARE training_start DATE DEFAULT DATE_SUB(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 12 MONTH);
DECLARE training_end DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

set training_start = '2025-07-01';
set training_end = '2025-07-23';

CREATE OR REPLACE TABLE `tom-moretti.nameless_analytics.ml_users_churn_training_data`
  PARTITION BY user_acquisition_date
  CLUSTER BY data_type
  AS (
    SELECT
      user_date AS user_acquisition_date,
      client_id, 
      user_id,
      sessions,
      days_from_last_visit,
      session_duration_sec,
      purchase,
      purchase_revenue,
      refund,
      refund_revenue,
      CAST(FLOOR(RAND() * 2) AS INT64) AS is_churned, -- Only for testing
      CASE
        WHEN user_date >= training_start AND user_date < training_end THEN "training_data"
        WHEN user_date >= training_end AND user_date <= CURRENT_DATE() THEN "evaluate_data"
        ELSE NULL
      END AS data_type
    FROM `tom-moretti.nameless_analytics.users`(training_start, CURRENT_DATE())
  );


-- CREATE MODEL
CREATE OR REPLACE MODEL `tom-moretti.nameless_analytics.ml_users_churn_model` OPTIONS(
  model_type = 'LOGISTIC_REG',
  -- model_type = 'BOOSTED_TREE_CLASSIFIER',
  input_label_cols = ['is_churned']
) AS (
  SELECT
    * EXCEPT(data_type)
  FROM `tom-moretti.nameless_analytics.ml_users_churn_training_data`
  WHERE data_type = 'training_data'
);


-- EVALUATE
SELECT
  *
FROM ML.EVALUATE(
  MODEL `tom-moretti.nameless_analytics.ml_users_churn_model`, (
    SELECT
      * EXCEPT(data_type)
    FROM `tom-moretti.nameless_analytics.ml_users_churn_training_data`
    WHERE data_type = 'evaluate_data'
  )
);


-- PREDICT
SELECT
  * EXCEPT (predicted_is_churned_probs, predicted_is_churned),
  predicted_is_churned
FROM ML.PREDICT(
  MODEL `tom-moretti.nameless_analytics.ml_users_churn_model`, (
    SELECT
      * EXCEPT(data_type)
    FROM `tom-moretti.nameless_analytics.ml_users_churn_training_data`
    WHERE data_type = 'evaluate_data'
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


## Test A/B e inferenze causali
Modello: ML.TRIAL_RESULT (usato con ML.HYPERPARAMETERS)
Cosa fa: Aiuta a capire quale variante funziona meglio tra due o più opzioni (es. email A vs B).

Utilità:
- Ottimizzare messaggi, landing page
- Automatizzare test su ampie basi utenti


## Lookalike modeling
Modello: LOGISTIC_REG, BOOSTED_TREE_CLASSIFIER
Cosa fa: Trova nuovi utenti simili a un gruppo che ha già convertito (o che ha un comportamento desiderato).

Input tipici:
- Dati dei clienti "ideali"
- Comportamenti utenti anonimi
- Demografica, comportamento navigazione

Utilità:
- Espansione audience per pubblicità mirata

---

Reach me at: [Email](mailto:hello@tommasomoretti.com) | [Website](https://tommasomoretti.com/?utm_source=github.com&utm_medium=referral&utm_campaign=nameless_analytics) | [Twitter](https://twitter.com/tommoretti88) | [Linkedin](https://www.linkedin.com/in/tommasomoretti/)
