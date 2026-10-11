Sales Forecasting for Retail Stores

Forecasting daily sales for 500 store-item series, 92 days ahead, with LightGBM. The model cuts forecast error by 28% compared with the best simple baseline.

![Dashboard](dashboard/dashboard_screenshot.png)

Problem

A retail business needs to plan stock and staffing in advance. This project forecasts daily sales for every store-item combination for the next 92 days (1 Oct to 31 Dec 2017), using only data available up to 30 Sep 2017.

Dataset

[Store Item Demand Forecasting Challenge](https://www.kaggle.com/c/demand-forecasting-kernels-only/data) on Kaggle.

- 10 stores x 50 items = 500 daily sales series
- About 5 years of history (2013 to 2017), no missing values
- Only `train.csv` is used. It is not included in this repository, see [`data/README.md`](data/README.md) for how to download it.

Approach

1. Explore the data. Trend, yearly and weekly seasonality, store and item differences.
2. Split by time, not randomly. Train on everything up to 30 Sep 2017 and validate on the last 92 days. A random split would let the model see the future.
3. Build baselines first. Seasonal naive (same weekday last year), 28-day moving average, and weekday average over the last 8 weeks. The model has to beat these.
4. Engineer features without leaking the future.
   - Calendar features: day of week, month, day, day of year, week of year, weekend flag
   - Lag features at 98, 182 and 364 days
   - Rolling averages (7, 28 and 91 days) computed on sales shifted by 98 days
   - The forecast horizon is 92 days, so every sales-based feature must be at least 92 days old. The notebook includes a check that fails if any validation row uses data after the cutoff.
5. Train LightGBM. One model for all 500 series, with store and item as categorical features. The number of trees was chosen with early stopping on a tuning window inside the training data, then the model was retrained and scored once on the validation period.
6. Evaluate and visualise. MAPE and RMSE on the same validation period for every model, plus error by store and item.
7. Build a dashboard. An Excel dashboard for a planning manager (see below).

Results

Validation period: 1 Oct to 31 Dec 2017 (92 days, 46,000 store-item-day forecasts).

| Model | MAPE | RMSE |
|---|---|---|
| LightGBM | 12.69% | 7.74 |
| Seasonal naive (same weekday last year) | 17.69% | 10.90 |
| Weekday average (last 8 weeks) | 26.91% | 15.74 |
| 28-day moving average | 27.30% | 15.58 |

- LightGBM reduces MAPE by about 28% versus the best baseline (seasonal naive), and RMSE from 10.90 to 7.74.
- Where the model struggles: stores 7, 5 and 6 have the highest error (15.2%, 14.6% and 14.3% MAPE) and also sell the least. Across items, sales volume and percentage error have a correlation of about -0.94, so the less an item sells, the larger its percentage error. This is largely a small-numbers effect: missing by 3 units matters more on 19 units than on 70.
- Bias: the model under-forecasts by about 2.2% overall, fairly evenly across stores. In practice that suggests a small safety-stock buffer.

 Dashboard

[`dashboard/Sales_Forecast_Dashboard.xlsx`](dashboard/Sales_Forecast_Dashboard.xlsx) is built from `outputs/forecast_results.csv`. It shows overall MAPE, RMSE and bias, actual vs forecast over time, error by store, the hardest items and stores to forecast, and a short call-out for managers on what to do about them. All figures are Excel formulas on the data sheet.

 Repository structure

```
sales-forecasting/
├── README.md
├── requirements.txt
├── .gitignore
├── sales_forecasting_guide.ipynb     # full analysis, with explanations
├── data/
│   └── README.md                     # how to download train.csv
├── outputs/
│   ├── forecast_results.csv          # actual vs forecast per store, item and day
│   └── model_comparison.csv          # MAPE and RMSE for each model
└── dashboard/
    ├── Sales_Forecast_Dashboard.xlsx
    └── dashboard_screenshot.png
```

How to run it

1. Clone the repository:
   ```bash
   git clone https://github.com/chan12333/sales-forecasting.git
   cd sales-forecasting
   ```
2. Install the libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Download `train.csv` from Kaggle and put it in the `data/` folder (see `data/README.md`).
4. Open `sales_forecasting_guide.ipynb` in Jupyter, VS Code or Google Colab and run all cells from the top. The results are written to `outputs/`.

Limitations

- Results come from a single validation window (Oct to Dec 2017). Repeating the evaluation over several time windows would give a more reliable estimate.
- No promotion, price or weather data is available, so the model can only learn from past sales and the calendar.
- Hyperparameters were not searched beyond early stopping.
- MAPE is inflated for low-volume stores and items, so RMSE and units-based errors are better guides for those series.
- A public dataset on one retailer: results may not carry over to other businesses.

Possible next steps

- Evaluate with several rolling time windows (`TimeSeriesSplit`)
- Tune hyperparameters and compare more models
- Add prediction intervals, so planners can see the range, not just one number

Author

Chandan Sharma, Data Analyst | [LinkedIn](https://www.linkedin.com/in/chandan-sharma-it) | chandansubhashsharma111@gmail.com
