# RebalanceAI

RebalanceAI is an interactive tool built in Google Colab (via an IPython notebook) for analyzing investment portfolios. It leverages Google's Gemini Generative AI (via LangChain), yFinance for stock data, and scientific libraries like SciPy for optimization. The tool fetches real-time stock data, computes key portfolio metrics (e.g., diversification, risk, returns), and uses AI to generate insightful analyses, health scores, risks, strengths, and personalized recommendations. It's designed for investors to evaluate portfolio health, simulate optimizations, and get AI-driven advice.

The notebook demonstrates end-to-end functionality: from data fetching and visualization to AI-powered portfolio assessment. It's ideal for personal finance enthusiasts, financial analysts, or anyone looking to build AI-enhanced investment tools.

## Features

- **Stock Data Fetching**: Retrieve historical and current stock prices, returns, and info using yFinance for any list of tickers.
- **Portfolio Metrics Calculation**:
  - Total value, asset weights, sector breakdowns.
  - Diversification score (Herfindahl-Hirschman Index).
  - Risk metrics: Volatility, Sharpe Ratio, Maximum Drawdown.
  - Performance: Unrealized P&L, top holdings/sector exposure.
- **Portfolio Optimization**: Use SciPy to suggest optimized asset weights for better risk-adjusted returns (e.g., minimizing volatility).
- **AI-Powered Analysis**: Integrate with Gemini GenAI to generate:
  - Health Score (0-100) with categorization (e.g., "Excellent", "Poor").
  - Risk assessment, strengths, and key risks.
  - Actionable recommendations with rationale and expected impact.
  - Stress test scenarios for different market conditions.
- **Visualizations**: Interactive plots for normalized stock prices, volatility comparisons, and portfolio breakdowns (using Matplotlib and Plotly).
- **User Input Support**: Upload your own portfolio CSV or use a sample; add custom concerns/goals for personalized AI analysis.
- **Export Results**: Save analysis to JSON for easy sharing or further use.

## Requirements

- Python 3.10+
- Google Colab environment (recommended for easy setup and API key management).
- Libraries (installed via pip in the notebook):
  - `langchain` and `langchain-google-genai` for AI integration.
  - `yfinance` for stock data.
  - `pandas`, `numpy`, `matplotlib`, `plotly`, `scipy` for data processing and visualization.
  - Google Gemini API key (free tier available; sign up at [Google AI Studio](https://aistudio.google.com/)).

## Installation

1. **Clone the Repository** (if using locally):
```bash
   git clone https://github.com/your-username/rebalance-ai.git
   cd rebalance-ai
```

2. **Install Dependencies**:
   Run the following in your terminal or directly in the notebook's first cell:
```bash
   pip install -q langchain langchain-google-genai yfinance pandas numpy matplotlib plotly scipy
```

3. **Set Up API Key**:
   - Obtain a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey).
   - In Colab: Add it as a secret named `Gemini_api_key` (via the key icon in the left sidebar).
   - Alternatively, paste it when prompted in the notebook.

4. **Open the Notebook**:
   - Upload `gemini.ipynb` to Google Colab and run the cells sequentially.

## Usage

1. **Run the Notebook**:
   - Execute cells step-by-step:
     - Install packages and import libraries.
     - Configure Gemini API.
     - Fetch sample stock data and visualize.
     - Upload your portfolio CSV (format below) or use the provided sample.
     - Provide optional user context (e.g., "I'm risk-averse and focused on long-term growth").
     - Run the analysis to get AI-generated insights.

2. **Portfolio CSV Format**:
   - Required columns: `ticker` (stock symbol), `quantity` (shares owned), `current_price` (current market price), `sector` (e.g., "Technology").
   - Optional: `avg_cost` (average purchase price for P&L calculations).
   - Example (`sample_portfolio.csv` included):
```csv
     ticker,quantity,avg_cost,current_price,sector
     AAPL,50,120.0,170.0,Technology
     MSFT,20,210.0,330.0,Technology
     TSLA,10,400.0,220.0,Automotive
```

3. **Run Analysis**:
   - The notebook will compute metrics, generate an AI prompt, and query Gemini.
   - Output includes: Health score, risks, strengths, recommendations, and detailed narrative.
   - Results are printed and saved as `portfolio_analysis.json`.

4. **Customization**:
   - Modify tickers or periods in `StockDataFetcher`.
   - Adjust AI prompt in `create_genai_prompt` for specific focuses (e.g., ESG factors).
   - Extend with more tools (e.g., news integration via NewsAPI).

## Methodology

This tool follows a structured approach to portfolio analysis, combining financial data processing, quantitative metrics, optimization techniques, and AI-driven qualitative insights. The methodology is divided into key stages:

### 1. Data Acquisition
- **Stock Data Fetching**: Historical price data (e.g., closing prices, returns) is retrieved using the `yfinance` library for specified tickers over a user-defined period (default: 1 year). This includes daily returns calculated as percentage changes in closing prices.
- **Portfolio Input**: User uploads a CSV with holdings details (ticker, quantity, avg_cost, current_price, sector). Current prices can be updated dynamically via yFinance if not provided.
- **Assumptions**: Data is annualized assuming 252 trading days per year. Risk-free rate defaults to 0% for simplicity (can be customized).

### 2. Metric Calculations
- **Portfolio Composition**:
  - Total value: Sum of (quantity × current_price) across holdings.
  - Weights: Individual asset value divided by total portfolio value.
  - Sector Breakdown: Aggregated weights by sector.
- **Diversification**:
  - Herfindahl-Hirschman Index (HHI): Sum of squared weights (∑ w_i²). Lower values indicate better diversification (e.g., <0.1 is well-diversified).
  - Top holding/sector exposure: Identifies concentration risks.
- **Performance Metrics**:
  - Unrealized P&L: ((Current Value - Cost Basis) / Cost Basis) × 100.
  - Mean Return: Annualized average daily return.
- **Risk Metrics**:
  - Volatility: Annualized standard deviation of returns (std_dev × √252).
  - Sharpe Ratio: (Mean Return - Risk-Free Rate) / Volatility.
  - Maximum Drawdown: Largest peak-to-trough decline in portfolio value over the period.
- **Implementation**: Metrics are computed using Pandas for data manipulation and NumPy for numerical operations.

### 3. Portfolio Optimization
- **Objective**: Minimize portfolio volatility subject to constraints (e.g., weights sum to 1, no short-selling).
- **Method**: Mean-Variance Optimization using SciPy's `minimize` function from the `scipy.optimize` module.
  - Covariance Matrix: Derived from historical returns to model asset correlations.
  - Optimization: Uses Sequential Least Squares Programming (SLSQP) solver.
- **Output**: Suggested rebalanced weights to achieve efficient frontier positioning.
- **Limitations**: Assumes historical data predicts future risk/return; does not account for transaction costs or taxes.

### 4. AI-Powered Analysis
- **Prompt Engineering**: A detailed prompt is constructed with computed metrics (e.g., HHI, Sharpe) and user context. It instructs Gemini to evaluate health score, risks, strengths, recommendations, and stress scenarios.
- **Model Configuration**: Uses `gemini-2.0-flash-exp` (or similar) with temperature=0.7 for balanced creativity and accuracy.
- **Response Parsing**: AI output is parsed as JSON for structured results (e.g., health_score, recommendations).
- **Stress Testing**: AI simulates scenarios (e.g., market crash, inflation) based on metrics, without explicit quantitative modeling.

### 5. Visualization and Reporting
- **Plots**: Matplotlib for static charts (e.g., price trends, volatility bars); Plotly for interactive elements.
- **Output**: Console prints, JSON export. AI narrative provides interpretive insights.

This methodology ensures a comprehensive, data-driven evaluation while leveraging AI for nuanced recommendations. It adheres to modern portfolio theory principles but is for educational/informational purposes only—not financial advice.

## Example
```python
# Sample usage in the notebook
test_tickers = ['AAPL', 'MSFT', 'GOOGL', 'NVDA', 'TSLA']
fetcher = StockDataFetcher(test_tickers, period='1y')
stock_data = fetcher.fetch_data()

# Portfolio analysis
holdings_df = pd.read_csv('sample_portfolio.csv')
analysis = analyze_portfolio_with_genai(holdings_df, user_context="Focus on diversification")
print(analysis['health_score'])  # e.g., 85
```

**Sample Output**:
- Health Score: 85/100 (Excellent)
- Risk Level: Moderate
- Key Risks: Technology sector concentration, top holding dependency.
- Recommendations: Rebalance sectors, add international exposure.

## Configuration

- **Gemini Model**: Defaults to `gemini-2.5-flash` (fast and cost-effective). Change in the notebook if needed.
- **Output Directory**: Results saved to `/content/portfolio_demo/` (customizable).
- **Error Handling**: The notebook includes try-except blocks for API failures and data issues.

