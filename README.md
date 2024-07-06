# Stock Time Series Analysis

## Importing Libraries
import pandas as pd
import plotly.express as px
from plotly.subplots import make_subplots
import plotly.graph_objects as go

# Load the data
file_path = "C:/Users/sunny/Downloads/Stock Time series/stocks.csv"
stocks_data = pd.read_csv(file_path)

# Display the first few rows
print(stocks_data.head())

# Get information about the DataFrame
print(stocks_data.info())

# Count of unique tickers
print(stocks_data.Ticker.value_counts())

# Descriptive statistics for each Ticker
descriptive_stats = stocks_data.groupby('Ticker')['Close'].describe()
print(descriptive_stats)

# Time Series Analysis
stocks_data['Date'] = pd.to_datetime(stocks_data['Date'])
pivot_data = stocks_data.pivot(index='Date', columns='Ticker', values='Close')

fig = make_subplots(rows=1, cols=1)
fig.add_trace(go.Scatter(x=pivot_data.index, y=pivot_data['A'], name='A'))
fig.add_trace(go.Scatter(x=pivot_data.index, y=pivot_data['D'], name='D'))
fig.add_trace(go.Scatter(x=pivot_data.index, y=pivot_data['C'], name='C'))
fig.add_trace(go.Scatter(x=pivot_data.index, y=pivot_data['B'], name='B'))
fig.update_layout(
    title_text="Time Series of Closing Prices",
    xaxis_title='Date',
    yaxis_title='Closing Price',
    legend_title='Ticker',
    showlegend=True
)
fig.show()

# Volatility Analysis
volatility = pivot_data.std()
fig = px.bar(
    volatility,
    x=volatility.index,
    y=volatility.values,
    labels={'y': 'Standard Deviation', 'x': 'Ticker'},
    title='Volatility of Closing Prices (Standard Deviation)'
)
fig.show()
# It indicates that C and B stocks were more prone to price fluctuations during this period compared to A and D.

# Correlation Analysis
correlation_matrix = pivot_data.corr()
fig = go.Figure(
    data=go.Heatmap(
        z=correlation_matrix,
        x=correlation_matrix.columns,
        y=correlation_matrix.columns,
        colorscale='blues',
        colorbar=dict(title='correlation'),
        text=correlation_matrix.round(2).values,
        texttemplate="%{text}"
    )
)
fig.update_layout(
    title='Correlation Matrix of Closing Prices',
    xaxis_title="Ticker",
    yaxis_title="Ticker",
)
fig.show()
# Values close to +1 indicate a strong positive correlation, meaning that as one stock’s price increases, the other tends to increase as well.
# Values close to -1 indicate a strong negative correlation, where one stock’s price increase corresponds to a decrease in the other.
# Values around 0 indicate a lack of correlation.
# From the heatmap, we can observe that there are varying degrees of positive correlations between the stock prices, with some pairs showing stronger correlations than others. For instance, A and B seem to have a relatively higher positive correlation.

# Comparative Analysis
percentage_change = ((pivot_data.iloc[-1] - pivot_data.iloc[0]) / pivot_data.iloc[0]) * 100
fig = px.bar(
    percentage_change,
    x=percentage_change.index,
    y=percentage_change.values,
    labels={'y': 'Percentage Change (%)', 'x': 'Ticker'},
    title='Percentage Change in Closing Prices'
)
fig.show()
# B: The highest positive change of approximately 16.10%.
# A: Exhibited a positive change of approximately 12.23%. It indicates a solid performance, though slightly lower than B’s.
# D: Showed a slight negative change of about -1.69%. It indicates a minor decline in its stock price over the observed period.
# C: Experienced the most significant negative change, at approximately -11.07%. It suggests a notable decrease in its stock price during the period.

# Daily Risk Vs. Return Analysis
daily_returns = pivot_data.pct_change().dropna()
avg_daily_return = daily_returns.mean()
risk = daily_returns.std()
risk_return_df = pd.DataFrame({'Risk': risk, 'Average Daily Return': avg_daily_return})

fig = go.Figure()
fig.add_trace(
    go.Scatter(
        x=risk_return_df["Risk"],
        y=risk_return_df['Average Daily Return'],
        mode="markers+text",
        text=risk_return_df.index,
        textposition="top center",
        marker=dict(size=10)
    )
)
fig.update_layout(
    title='Risk vs. Return Analysis',
    xaxis_title='Risk (Standard Deviation)',
    yaxis_title='Average Daily Return',
    showlegend=False
)
fig.show()
# A shows the lowest risk combined with a positive average daily return, suggesting a more stable investment with consistent returns.
# D has higher volatility than A and, on average, a slightly negative daily return, indicating a riskier and less rewarding investment during this period.
# B shows moderate risk with the highest average daily return, suggesting a potentially more rewarding investment, although with higher volatility compared to A.
# C exhibits the highest risk and a negative average daily return, indicating it was the most volatile and least rewarding investment among these stocks over the analyzed period.
