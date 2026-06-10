# Introduction
Injury predictors are divided into 2: internal and external predictors
Internal predictors include age,  gender, previous injury, heart rate variability (HRV), sleep quality, limb strength asymmetries, and fatigue.

## Internal predictors
**Age and gender** are prioritized as non-modifiable baseline variables (characteristics that cannot be changed or controlled through medical, surgical, or lifestyle interventions) because they dictate the structural and hormonal boundaries of an individual's load capacity and recovery trajectory. **Previous Injury** is often the most powerful internal indicator in a model. For example, research has shown that a prior hamstring strain can increase the risk of a future one by more than 11 times, even after accounting for other factors such as age. **Heart rate variability (HRV)** and **sleep quality** serve as essential dynamic biomarkers of autonomic nervous system readiness, providing real-time data on a person’s physiological resilience and their ability to absorb physical stress without breaking down. **Limb strength asymmetries** act as critical mechanical red flags, as significant imbalances in force distribution frequently lead to compensatory movement patterns that increase the risk of acute and overuse injuries. 
**Fatigue** is also a critical predictor because it directly degrades other internal safety markers, such as neuromuscular control, reaction time, and limb proprioception, thereby increasing the risk of acute injury during physical activity (e.g., a study examining the effect of fatigue on reactive postural control).

## External Predictors
External predictors focus on the physical demands placed on the individual and the environmental conditions under which they operate. Key statistical markers include **training load metrics, specifically the volume and intensity of activities like sprint distance or heavy lifting.**
Environmental data, such as **playing surface quality, altitude, and weather conditions**, are integrated to account for the physical stressors of the surrounding environment.
**Training load metrics** serve as the primary quantitative measure of the physical "work" performed, categorized into volume and intensity. **Volume** tracks the total amount of activity, such as total distance covered or the sum of weight lifted, while **intensity** measures the magnitude of that effort, such as top sprint speeds or the percentage of a maximum lift. These markers are essential because they allow models to identify when the body is being pushed beyond its current structural capacity, which is often the immediate catalyst for musculoskeletal failure.
**Environmental data** provides the context for how workload metrics are interpreted, as the same amount of effort requires more physiological resources under different external conditions. Factors such as high altitude or extreme heat and humidity act as force multipliers for fatigue, placing additional strain on the cardiovascular and thermoregulatory systems. Integrating these variables into a model ensures that the predicted risk score reflects not just the work being done, but the actual physical "cost" of performing that work in a specific setting.
**Surface quality and equipment conditions** account for the mechanical stressors that influence joint and ligament stability. Variables like the friction of a playing surface (e.g., turf vs. natural grass) or the degradation of footwear cushioning can alter biomechanical loading patterns in ways that internal data cannot capture.

## Algorithm Used by Researchers
Researchers and safety professionals use algorithms like **Random Forests, Gradient Boosting Decision Trees (GBDT), and XGBoost** to identify hidden patterns that traditional linear regressions might miss.

**Random Forests:** For injury prevention, the model processes various inputs such as HRV, sleep, and workload across hundreds of trees to determine the most likely outcome, such as "High Risk" or "Low Risk." To arrive at a final result, the algorithm uses majority voting, meaning the prediction supported by the highest number of individual trees becomes the official output of the model. This model is good at handling complex, non-linear relationships and remains accurate even when some data points, like a missed wearable sync, are missing. It can also tell exactly which variable (e.g., HRV vs. Training Load) is the strongest predictor of injury in your specific group.

**Gradient Boosting Decision Trees (GBDT):** It is a high-performance ensemble machine learning technique that builds a strong predictive model by sequentially training multiple "weak" decision trees to correct the errors of their predecessors. Unlike Random Forests, which build trees independently in parallel, GBDT works iteratively, with each new tree attempting to minimize the "residuals" or remaining errors from the previous stage. It is favored in injury modeling for its superior ability to identify hidden patterns in tabular datasets while natively handling missing sensor data. It also excels at modeling complex interactions, like how high training load combined with poor sleep exponentially increases risk.

**XGBoost (eXtreme Gradient Boosting):** It is a highly optimized version of the Gradient Boosting algorithm that uses an ensemble of decision trees to deliver superior speed and accuracy. In injury prevention modeling, it works by iteratively training "weak" decision trees, where each new tree specifically focuses on correcting the prediction errors made by all previous trees.  It utilizes a "sparsity-aware" algorithm that automatically learns how to handle missing values, which is critical for wearable data streams where a sensor might temporarily lose a signal. This allows researchers to see exactly which data points (e.g., heart rate, age, or specific gait variables) are the most significant "risk triggers," providing clinicians with clear, actionable insights for personalized intervention.


## Breakdown of Predictor Variables

| **Category** | **Variable Name** | **Scale / Measurement Unit** |
|---|---|---|
| **Internal (Static)** | x₁: Age | Years (Discrete) |
|  | x₂: Bio-Sex | Binary (0 = Male, 1 = Female) |
| **Internal (Moving)** | x₃: Injury Hist | Count of previous injuries in 24 months |
|  | x₄: HRV | RMSSD (milliseconds) |
|  | x₅: Sleep | $\frac {Total hours}{Sleep Efficiency \%}$ |
|  | x₆: Asymmetry | Strength Ratio (e.g., 1.0 = Symmetry, 1.15 = 15% Imbalance) |
|  | x₇: Fatigue | RPE Scale (1–10) or VAS Scale |
| **External** | x₈: Acute Load | AU (Arbitrary Units): Intensity × Duration |
|  | x₉: ACWR | Ratio: $\frac {Acute Load (7d)}{Chronic Load (28d)}$ |
|  | x₁₀: Environment | Heat Index (°C) or Friction Coefficient (u) |



| **Data Point (Variables)** | **Collection Frequency** | **How do we calculate/record** |
|---|---|---|
| **ACWR** | Daily | $\frac {Acute Load (7d)}{Chronic Load (28d)}$ |
| **Acute Load** | Daily | 7-day rolling average |
| **HRV** | Daily (Morning) | 7-day rolling average |
| **Sleep** | Daily | Total hours, Quality score (subjective scale of 1–4), and 7-day average of Quality score to smooth out the noise |
| **Fatigue** | Daily | Subjective 1–10 Scale (Morning) |
| **Environment** | Per session | Categorical (e.g., 0 = Indoor/Cool, 1 = Outdoor/Heat, 2 = Rainy/Slippery) |
| **Asymmetry** | Monthly | $\frac{Stronger − Weaker}{Stronger} × 100 $ |
| **Injury History** | Once (updated when injury occurs) | Binary (Yes or No), then we collect the number |
| **Age** | Yearly | Continuous |
| **Bio-sex** | Once | Categorically (Male or Female) |


Formula to calculate Risk (in %)
In a simplified linear version, the probability of injury (P) is:


P(x) = 𝝈(𝛃₀ + 𝛃₁x₁ + 𝛃₂x₃ + 𝛃₃x₆ + 𝛃₄x₉ + …)

```math
P(\text{injury}) = \frac{1}{1 + {e(p(x))}}
```

Risk = P(injury) * 100 

Where:
- 𝝈: The Logistic function (to keep the risk between 0 and 1).
- 𝛃₁: The Weighting Factor (the "extent" to which a factor matters).
- x₉ (ACWR): If x₉ >1.5, the weight B4 increases exponentially.

## Handling Missing Variable.

If a variable is missing, say “HRV (x₅)”. Instead of removing it, we insert the individual’s historical average. The Logic (the safest statistical bet) is that the person is at their normal baseline. This keeps “HRV (x₅)” active in the calculation without unfairly skewing the result toward "high risk". 

If a data point is missing and we have no historical average, we redistribute the weights so the total of our 𝛃 values is still equal to 1.0 (100%). 

```math
P(Injury) = \frac{\sigma\left(\sum (\beta_{\text{available}} \cdot X_{\text{available}})\right)}{\sum \beta_{\text{available}}}
```

**Baseline:**
30
The **baseline** is our zero point, or the normal value the model will use to decide whether today’s data is good or bad. I understand that from the start, we won't have a history/baseline.


I believe for a start, we can use medical standards (Safe averages) based on the user's age or sex. **After about 30 days**, we can replace the population data with the user's own average.


From the first 30 days, we weigh the user’s data against the medical standard (baseline value). We can use the Z-score Standardization. So, instead of using raw numbers, the formula converts everything into a "Difference from Baseline."

```math
X_{\text{input}} = \frac{\text{Today's Value} - \text{ Baseline Value/Average value of the user}}{\text{Variation (Standard Deviation)}}
```

- Day 1 Baseline = Population Average.
- Day 30: Baseline = User's Actual 30-day Average.


## How do I record moving data? Can I use data that was recorded at different times? 
Static and Moving Data: Static Data, such as Age, Sex, and Injury History (collected at sign-up and whenever the user updates it), is collected once during sign-up and stored in a library. Moving Data, like Sleep, HRV, and Fatigue, is collected every morning. Acute load is collected immediately after a workout, ACWR is recalculated instantly after every new acute load entry, and Asymmetry is updated only during a specific physical assessment or test.

Yes, we can use data recorded at different times using a step called "Snapshotting." It is a single, unified row of data that "freezes" a user’s status at a specific moment in time to run a calculation.

Every time the model needs to calculate a Risk Score (like right after a workout), it takes a "Snapshot" of the user. It grabs the Static data from the library and the latest Moving data from the log to create one single line of information, e.g., at 4:00 PM today, the app says: "Give me the user's Age (from 6 months ago), their Sleep (from this morning), and the Workout they just finished 5 minutes ago." The model treats them all as "The Current State" of the person.

The only issue with it is that the older our data, the lower the accuracy of our prediction. E.g., if HRV wasn’t recorded for over a year, the model treats it as static data and picks the last updated information. Its logic will be "I don't know how your heart feels today, so I will assume you are the same as you were 365 days ago."

It pulls the current information and the old HRV (now acting as Static) and puts them into the formula together. The impact still predicts general risk, but it might miss a "sudden snap" injury if it's out of date.


## How do we define injury risk?
- **Low-Risk Injury:** 0-10%
- **Mild Risk Injury:** 11-25%
- **Moderate Risk:** 26-50%
- **High Risk:** 51-75%
- **Critical Risk:** >75%

NB: A risk jump of >20% between two days is considered a "Medical Red Flag," regardless of which category the patient is in.
