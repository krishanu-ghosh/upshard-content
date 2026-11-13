---
title: Time Series Fundamentals
description: Trend, seasonality, and stationarity are the three ideas that determine which models and which mistakes apply to time-ordered data.
roadmap: ai-data-scientist
node: time-series-fundamentals
group: econometrics-and-experimentation
video:
  provider: youtube
  id: "GE3JOFwTWVM"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 448
level: intermediate
prerequisites: [designing-ab-tests]
takeaways:
  - "Any time series can be decomposed into trend, seasonality, and a residual component, and most forecasting mistakes come from treating one component as if it were another."
  - A series is stationary when its statistical properties, mean, variance, and autocorrelation, don't depend on when you measure it, and most classical models require this to work correctly.
  - Differencing, subtracting each value from the one before it, is the standard way to turn a trending or seasonal series into a stationary one.
  - Random walk behavior, where the best forecast for tomorrow is simply today's value, is common in financial and other time series and looks deceptively like a trend.
check_questions:
  - q: What are the three components a time series is typically decomposed into, and what does each capture?
    a: Trend captures the long-run direction the series is moving in, seasonality captures a pattern that repeats at a fixed, known period such as weekly or yearly, and the residual is whatever's left after removing both, ideally close to random noise.
  - q: "Why does stationarity matter for classical time-series models, and what does \"stationary\" mean?"
    a: A stationary series has a mean, variance, and autocorrelation structure that don't change over time, and most classical forecasting models assume this because they estimate a single set of parameters meant to apply across the whole series, which only makes sense if the series behaves consistently throughout.
  - q: How does differencing help turn a non-stationary series into a stationary one?
    a: Differencing replaces each observation with the change from the previous observation, which removes a steady trend (since the change stays roughly constant even as the level rises) and can also remove certain kinds of non-constant variance tied to the trend.
deeper:
  - label: "Time Series Talk: Stationarity"
    url: "https://www.youtube.com/watch?v=oY-j2Wof51c"
  - label: "Time Series Talk: What is Seasonality?"
    url: "https://www.youtube.com/watch?v=4hrMdu9CSQs"
related_formulas: []
---

## Why order matters

Most data science techniques assume your rows are exchangeable — shuffle them and nothing about the analysis changes. Time series data breaks that assumption on purpose: the order of observations carries information, and shuffling a sequence of monthly revenue figures destroys exactly the structure you're trying to model. This single difference is why time series gets its own toolkit instead of just reusing standard regression. The first step in working with any such series is learning to see it as a sum of simpler pieces, because most of the modeling decisions that follow depend on correctly identifying which piece you're looking at.

## Decomposing a series into trend, seasonality, and noise

Take monthly ice cream sales for a chain of stores over five years. Plot it and you'll see sales climbing overall as the chain opens new stores (trend), a sharp jump every June through August and a dip every winter (seasonality), and smaller month-to-month wiggles that don't fit either pattern (the residual). Formally, a time series is treated as trend plus seasonality plus residual (or their product, in a multiplicative decomposition, when the seasonal swings grow proportionally with the trend rather than staying a fixed size). Trend is the slow-moving, long-run direction, estimated by smoothing out the short-term noise. Seasonality is a pattern that repeats at a fixed, known period, a week, a month, a year. The residual is whatever's left after you subtract both out, and in a well-fitted decomposition it should look close to unpredictable noise with no obvious pattern remaining. Most forecasting errors trace back to confusing these components: treating a seasonal summer bump as if it were a permanent trend leads you to overbuild for December, while treating true trend as noise leads you to miss that the business is structurally growing.

## Stationarity: the assumption models need to be told exists

A series is stationary when its statistical properties don't depend on where in time you look: the mean, the variance, and the way observations correlate with earlier observations stay the same whether you examine the first third of the data or the last third. Why does this matter? Classical models like ARIMA estimate one set of parameters meant to describe the whole series. If the series' behavior is fundamentally different in January 2020 than in January 2024 — a different average level, a different amount of volatility — a single set of parameters can't honestly describe both periods, and forecasts built on that mismatch will be systematically wrong, usually in the direction of underestimating how much the recent regime has already shifted. A rising trend is the most common source of non-stationarity: the mean keeps increasing, so "the average value" is a different number depending on which window you ask about. Strong seasonality is another: the variance and local mean both swing with the season.

## Differencing: the standard fix

The standard way to convert a trending series into a stationary one is differencing: instead of modeling the raw level of the series, you model the change from one period to the next. If sales are climbing steadily by about 200 units a month, the raw series has a rising mean, but the differenced series — this month's sales minus last month's — hovers around 200 with no trend of its own, because the climb has been absorbed into a roughly constant step size. Seasonal differencing works the same way but subtracts the value from the same period a year (or a cycle) earlier, which can remove seasonality the same way ordinary differencing removes trend. You can check whether differencing worked using a formal test for stationarity (the Augmented Dickey-Fuller test is the standard one), or informally by checking that the differenced series' mean and spread look stable across the whole window rather than drifting.

## The random walk trap

One pattern deserves special attention because it looks like a trend but behaves completely differently: a random walk, where each value is the previous value plus an unpredictable random shock, with no real underlying trend at all. Stock prices are the canonical example. Looking at a chart of a random walk that happened to drift upward over a year, it's tempting to fit a trend line and extrapolate it forward, but a random walk's defining property is that the best forecast for tomorrow is simply today's value, not today's value plus the recent slope — the series has no memory of a direction, only of its last position. Differencing a true random walk once produces pure noise, which is itself a useful diagnostic: if first-differencing your series leaves behind something that looks like unpredictable noise with no remaining structure, you may be looking at a random walk rather than a trend worth forecasting forward, and that distinction should change how much confidence you put in any extrapolation.
