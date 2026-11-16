# Next-day air quality forecast for Milan

**Track:** Climate, project 4.3.2 (Next-day air quality forecast)

**Team:** YONK by Gozzini Raffaele, Hnida Rayan, Singh Taranpreet

*This project is a prototype. It is not an official health or air-quality warning.*

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg))](https://colab.research.google.com/github/TaranAzrot-13/air-quality-forecast-milan/blob/main/air_quality_forecast.ipynb)

**Demo video:** [[video link]](https://drive.google.com/file/d/1S0xjB7F1OhJUhcBiH4DRJVHvtZHiY58F/view?usp=sharing)

## The problem

Fine particles in the air (PM2.5) can harm children and people with asthma or heart conditions. Milan often has days above 35 µg/m³, the 24-hour level used as a standard by the US EPA: in the four 90-day periods we tested, 135 of 360 days were above it. Schools and families need to know a day ahead whether to limit time outdoors.

## What we built

A notebook that downloads daily PM2.5 for a monitoring station in Milan plus daily weather, forecasts the next day's PM2.5, and turns it into a plain outlook: **Fine for most people**, **Be careful**, or **Limit outdoor time**, together with a likely range.

- **Goes in:** today's PM2.5 and weather, and the time of year.
- **Comes out:** a forecast for the next day, a range, and an outlook with simple advice.
- **A Gradio demo with two tabs:**
  - *Replay past days* shows days the model never saw during training, next to what actually happened. This is the part we evaluated.
  - *Latest forecast* is an add-on that uses the most recent readings and weather. We did not evaluate it separately, so it is a demonstration and not a validated service.

## How it works

**Data.** Daily PM2.5 from the OpenAQ station Porta Venezia (2.2 km from the city centre), 2024-11-14 to 2026-11-08, with daily weather for the same place from Open-Meteo (temperature, humidity, rain, wind, pressure). Hourly readings are averaged into days, short gaps of up to two days are filled by interpolation, and longer gaps are left out. The downloaded data is saved as a snapshot file so anyone can rerun the notebook and get our exact numbers without a key.

**Model.** Gradient boosting (trained to minimise absolute error) and a Ridge regression, using today's PM2.5, yesterday's, the 7-day average, the one-day change, today's weather and the time of year. We split by time: the most recent 120 days (2026-07-11 to 2026-11-07) are the test set and everything earlier is training data, so the model never learns from the future.

**Baseline.** "Tomorrow equals today", as the brief suggests.

**What we added of our own**
- A **weather honesty check**: tomorrow's real weather is the strongest clue but a real forecaster would only have a forecast, so we report the model that uses it separately as an *upper bound*.
- A **warning rule for bad days** (35 µg/m³ or more) whose level is chosen with an explicit cost for missed warnings versus false alarms, using training data only.
- A **forecast range**, with a check of how often it really contains the truth.
- A **stability check** across four earlier 90-day periods and a block-bootstrap check on the error difference.
- An analysis of **what drives pollution** (season, wind, which inputs the model uses).

## Results

| Next-day PM2.5 forecast | Mean absolute error (µg/m³) | Compared with the baseline |
|---|---|---|
| Baseline: tomorrow equals today | 6.70 | , |
| Our model | 5.68 | 15% lower |
| Upper bound: with tomorrow's real weather (not available to a real forecaster) | 4.85 | 28% lower |

The model beat the baseline in all 4 earlier 90-day periods.

**How often is the outlook right?** The outlook (Fine for most people, Be careful, Limit outdoor time) differed from what actually happened on 39 of the 120 test days, about a third.

**How sure are we?** On the final 120-day test alone, the improvement is 1.01 µg/m³ with a 95% range from -0.25 to +2.44, which includes zero, so we cannot rule out luck from that period alone. The consistent win in all 4 earlier periods supports it, but we call the evidence moderate, not conclusive. For the simpler Ridge model the improvement (0.34 µg/m³, range -0.57 to +1.58) is not clear either.

| "Limit outdoor time" warnings, pooled over 4 earlier periods (135 bad days) | Bad days caught | False alarms |
|---|---|---|
| Baseline: warn if today is bad | 113 | 23 |
| Our model | 115 | 33 |

**Where it fails, and why we think so**
- **As a yes/no warning it is not clearly better than the simple rule.** It caught 2 more bad days but raised 10 more false alarms. Bad days come in multi-day episodes, so "today is bad" already predicts tomorrow well. The model is better at the number than at the decision.
- **The forecast range is too cautious.** It contained the real value on 92% of test days, not the intended 80%. Our guess is that the training period contains more volatile winter days than the summer and autumn test period.
- **One station and about two years of data.** One station may not represent every neighbourhood, and two winters is a short record.
- **Only today's weather is used.** A real system would use a weather forecast, which has its own errors.
- **Fairness.** We have no data on different groups of people, so we could not check performance across groups. We checked it across four time periods instead, and we note that one station may serve some neighbourhoods better than others.
- **The latest-forecast tab is an add-on.** It forecasts the day after the latest complete day (usually today, from yesterday's readings) and needs an OpenAQ key. We have not evaluated it separately.

## What we'd do next

1. Replace today's weather with a real weather forecast, and measure how much of the gap to the upper bound we can close.
2. Add more stations and more years, and make the forecast range depend on the season. Then test the outlook with a school.

## Run it and repository contents

**To rerun**
1. Click the *Open in Colab* badge at the top, then choose Runtime, then Run all. It takes a few minutes.
2. No key is needed. The notebook loads the saved data snapshot from this repository, so it reproduces our exact numbers.
3. The last cell starts the demo with a temporary public link. The *Latest forecast* tab needs an OpenAQ key and shows a short message without one.
4. To download fresh data instead, add an OpenAQ key as a Colab Secret named `OPENAQ_API_KEY` and set `USE_SNAPSHOT = False` in the data cell. Results will then differ because the period moves forward.

**Files**
- `air_quality_forecast.ipynb`: the notebook (data, model, evaluation, demo).
- `air_quality_snapshot.json`: the saved daily PM2.5 and weather data behind all results.
- `README.md`: this write-up.

## Sources

- **OpenAQ**, open air-quality measurements: https://openaq.org
- **Open-Meteo**, historical weather, geocoding and forecast APIs (CC BY 4.0): https://open-meteo.com
- **US EPA** 24-hour PM2.5 standard (35 µg/m³) and the **WHO** 2021 air-quality guideline for PM2.5 (15 µg/m³), used as planning levels, not as health advice.
- **Libraries:** scikit-learn, pandas, NumPy, Matplotlib, Gradio.
- **AI tools:** Claude (Anthropic, https://claude.ai) was used by our team to help write and debug the notebook code and to draft this write-up. The results in this document come from running the notebook ourselves on our own data download.
