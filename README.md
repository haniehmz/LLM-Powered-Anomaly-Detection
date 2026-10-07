# LLM-Powered Network Anomaly Detection

A cybersecurity project that combines **classical machine learning** and **Large Language Models (LLMs)** to detect, analyze, and explain anomalous network traffic.

The system uses an **Isolation Forest** model for unsupervised anomaly detection and integrates an LLM to generate human-readable explanations whenever suspicious network activity is detected.

This project was developed as an academic exercise for applying **machine learning and artificial intelligence techniques to cybersecurity**.

---

## Overview

Network traffic contains patterns that can indicate normal communication as well as potentially suspicious or abnormal behavior.

This project implements a lightweight real-time anomaly detection pipeline that:

1. Generates simulated network traffic.
2. Sends the traffic continuously through a TCP socket.
3. Preprocesses incoming network data.
4. Uses an **Isolation Forest** model to detect anomalies.
5. Calculates an anomaly confidence score.
6. Sends detected anomalies to an **LLM-based cybersecurity analyst**.
7. Generates a short anomaly label and detailed explanation.
8. Logs detected anomalies and their explanations into a CSV file.

The overall architecture combines:

```text
Network Traffic
      │
      ▼
   TCP Server
      │
      ▼
   TCP Client
      │
      ▼
Data Preprocessing
      │
      ▼
Isolation Forest
      │
      ├───────────────┐
      │               │
    Normal          Anomaly
      │               │
      ▼               ▼
 Continue        LLM Analysis
                    │
                    ▼
             Label + Explanation
                    │
                    ▼
              Anomaly Log
```
---

## Key Features

* Unsupervised network anomaly detection
* Isolation Forest machine learning model
* Synthetic network traffic generation
* Real-time TCP communication
* Feature preprocessing and normalization
* Anomaly confidence scoring
* LLM-powered anomaly analysis
* Automatic anomaly labeling
* Human-readable explanations
* CSV-based anomaly logging
* PCA visualization of detected anomalies
* Separation between traffic generation and anomaly analysis

---

## Network Traffic Features

Each network traffic record contains the following features:

| Feature       | Description                   |
| ------------- | ----------------------------- |
| `src_port`    | Source port of the connection |
| `dst_port`    | Destination port              |
| `packet_size` | Packet size in bytes          |
| `duration_ms` | Communication duration        |
| `protocol`    | Network protocol              |

---

## Simulated Traffic Generation

The project includes a TCP server that continuously generates simulated network traffic.

Most generated traffic represents normal behavior, while a smaller percentage is intentionally generated as anomalous traffic.

The server generates approximately:

* **80% normal traffic**
* **20% anomalous traffic**

### Normal Traffic

Normal traffic uses:

* Common source ports such as `80`, `443`, `22`, and `8080`
* Packet sizes between `100` and `1500` bytes
* Communication durations between `50` and `500` milliseconds
* `TCP` or `UDP` protocols

### Anomalous Traffic

Several anomaly patterns are simulated.

#### Suspicious Port

```text
Source port → 1337, 9999, or 6666
```

#### Abnormally Large Packet

```text
Packet size → 2000–10000 bytes
```

#### Abnormally Long Communication

```text
Duration → 2000–5000 ms
```

#### Unknown Protocol

```text
Protocol → UNKNOWN
```

This allows the system to simulate different types of suspicious network behavior.

---

## Machine Learning Model

The project uses the **Isolation Forest** algorithm from Scikit-learn.

Isolation Forest is an unsupervised anomaly detection algorithm that identifies observations that are easier to isolate from the rest of the dataset.


### Why Isolation Forest?

Isolation Forest is suitable for this project because:

* It does not require labeled anomaly data.
* It is computationally efficient.
* It works well with numerical network features.
* It can identify unusual observations based on feature distributions.
* It is commonly applicable to anomaly detection problems.

---

## Data Preprocessing

The `protocol` feature is categorical and therefore needs to be converted into numerical form.

The numerical features are then standardized using `StandardScaler`.

The preprocessing pipeline is:

```text
Raw Network Data
       │
       ▼
Categorical Encoding
       │
       ▼
Numerical Features
       │
       ▼
StandardScaler
       │
       ▼
Isolation Forest
```

The fitted scaler is saved as:

```text
scaler.joblib
```

This ensures that the same preprocessing transformation can be reused during real-time inference.

---

## Model Training

The training dataset is generated programmatically and stored in:

```text
dataset/training_data.json
```

The trained Isolation Forest model is serialized using Joblib:

```text
anomaly_model.joblib
```

The saved model can then be loaded by the real-time client without retraining.

---

## Anomaly Scoring

In addition to the binary prediction, the system calculates an anomaly score using:

```python
confidence_score = -model.score_samples(processed_data)[0]
```

Higher values indicate observations that are more unusual according to the model.

The system uses:

```text
1  → Normal
-1 → Anomaly
```

The anomaly score is stored together with the detected network traffic.

> The `confidence_score` should be interpreted as an anomaly-related score rather than a calibrated probability of malicious activity.

---

## PCA Visualization

Principal Component Analysis (PCA) is used to project the feature space into two dimensions.

This provides a visual representation of the relationship between normal and anomalous observations.

```text
Original Feature Space
        │
        ▼
       PCA
        │
        ▼
    PC1 + PC2
        │
        ▼
2D Visualization
```

The visualization helps demonstrate how the Isolation Forest separates unusual observations from normal traffic patterns.

---

## LLM-Powered Security Analysis

One of the main features of this project is the integration of a Large Language Model.

The LLM is invoked only when the machine learning model identifies an anomaly.

The current implementation uses:

```text
Llama 3.3 70B Instruct Turbo Free
```

through the Together AI API.

The LLM receives:

* Source port
* Destination port
* Packet size
* Communication duration
* Protocol
* Expected normal behavior

It is instructed to act as a cybersecurity analyst and return:

```text
Label: <short anomaly label>
Reason: <detailed explanation>
```

For example, a suspicious record could be interpreted as:

```text
Label: Abnormally Long Connection

Reason: The communication duration is significantly higher than
the expected normal range, which may indicate suspicious or
persistent network activity.
```

This allows the system to move beyond simply saying:

```text
Anomaly = -1
```

and instead provide an explanation that is easier for a human analyst to understand.

---

## Hybrid AI Architecture

The most important concept demonstrated by this project is the combination of **traditional machine learning and generative AI**.

### Stage 1 — Detection

Isolation Forest determines whether the network traffic is anomalous.

### Stage 2 — Reasoning

The LLM analyzes the suspicious traffic and provides an explanation.

```text
              Network Traffic
                     │
                     ▼
              Isolation Forest
                     │
          ┌──────────┴──────────┐
          │                     │
       Normal                 Anomaly
          │                     │
          ▼                     ▼
      No Alert             LLM Analysis
                                │
                                ▼
                      Security Explanation
                                │
                                ▼
                         Anomaly Logging
```

This architecture separates **machine-based detection** from **language-based interpretation**.

---

## Real-Time Communication

The system consists of two main processes.

### Server

`server.py`

The server:

* Opens a TCP socket.
* Waits for a client connection.
* Generates network traffic.
* Serializes each record as JSON.
* Sends records continuously to the client.

Traffic is transmitted approximately every two seconds.

### Client

`client.py`

The client:

* Connects to the server.
* Receives JSON network records.
* Preprocesses the data.
* Loads the trained Isolation Forest model.
* Performs anomaly detection.
* Calculates the anomaly score.
* Sends anomalies to the LLM.
* Logs the results.

---

## Configuration


Use your api key this section.

```text
TOGETHER_API_KEY=your_api_key_
```

---

## Running the Project

### Step 1 — Train the Model

Run:

```text
train_model.ipynb
```

This will:

1. Generate the training dataset.
2. Preprocess the network traffic.
3. Train the Isolation Forest model.
4. Save `anomaly_model.joblib`.
5. Save `scaler.joblib`.
6. Generate anomaly predictions.
7. Calculate anomaly scores.
8. Create PCA visualizations.

---

### Step 2 — Start the Server

Run:

```bash
python server.py
```

The server starts listening on:

```text
localhost:9999
```

It then continuously generates and sends network traffic.

---

### Step 3 — Start the Client

In a second terminal:

```bash
python client.py
```

The client connects to the server and begins analyzing incoming traffic.

For normal traffic:

```text
Data is normal.
```

For anomalous traffic:

```text
🚨 Anomaly Detected!

Label: ...
Reason: ...

Confidence Score: ...
```

---

## Anomaly Logging

Detected anomalies are stored in:

```text
anomalies_log.csv
```

The log contains:

| Field              | Description                    |
| ------------------ | ------------------------------ |
| `src_port`         | Source port                    |
| `dst_port`         | Destination port               |
| `packet_size`      | Packet size                    |
| `duration_ms`      | Communication duration         |
| `protocol`         | Network protocol               |
| `label`            | LLM-generated anomaly label    |
| `reason`           | LLM-generated explanation      |
| `confidence_score` | Isolation Forest anomaly score |

This creates a simple security event log that can later be used for further analysis.

---

