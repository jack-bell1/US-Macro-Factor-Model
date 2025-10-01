# US-Macro-Factor-Model
### Intro:
##### Should you wish to view the code and see in greater detail what was done here, please refer to![my Factor Model](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/macro_factor_model.ipynb) in the repo titled `macro_factor_model.ipynb`. 
##### I have included a TL;DR section to provide an overview of what the project was, with some sample results. The .ipynb file is a lot more vocal in explaining my thought processes and analyses, should you be interested!
---
## Tech Stack:
The project was completed in Python; the libraries used were as follows:
- Pandas
- NumPy
- Scikit-learn
- statsmodels
- seaborn
- matplotlib
- plotly
- scipy
- yfinance

## Skills Utilized:
- Regression Modeling (Supervised Learning)
- Regression Analysis
- Principal Component Analysis (PCA - Unsupervised Learning)
- Time Series Analysis
- Correlation Analysis
- Residual Analysis
- Distribution Analysis
- Statistical Tests (Ljung-Box, Jarque-Bera)
- Exploratory Data Analysis (EDA)
- Data Visualization

---

### Motivations:
Inspired by [Two Sigma Factor Lens](https://www.twosigma.com/articles/thematic-research-introducing-the-two-sigma-factor-lens), which models broad macro risk drivers to reduce correlation and improve portfolio construction. The paper uses tradeable proxies to enable a portfolio construction from the factor model. However, my approach will be slightly different.


### Project goals:
Build a factor model to represent major US macro risks. Use it to detect market regimes for better trading signals. Market regimes are purely statistical methods to identify whether or not we are in a BULL / BEAR / SIDEWAYS market. Perhaps there are more than these regimes in the market; we will explore this after we have constructed our factor model. 
#####
### Factor Structure:
- Core Factors: Dominant macro drivers explaining most variance across asset classes. 
- Secondary Factors: Specialized risks with smaller, but still meaningful, influence.


| Factor            | Type       | Proxy Ticker(s) | Description                                | Inception Date |
|-------------------|------------|-----------------|--------------------------------------------|----------------|
| US Equity (Target)| Core       | SPY             | S&P 500 ETF                                | 1993           |
| Interest Rates    | Core       | IEF             | 7–10 Year US Treasury Bonds                | 2002           |
| Credit            | Core       | LQD, HYG        | IG & High Yield Credit                     | 2002 / 2007    |
| Commodities       | Core       | DBC             | Broad Commodities Index ETF                | 2006           |
| USD FX Strength   | Secondary  | UUP             | US Dollar Index ETF                        | 2007           |
| Local Inflation   | Secondary  | TIP             | US TIPS ETF                                | 2003           |


### Factor motivations : 
- US Equity: We want to predict regimes inside this proxy. We still want to residualize to get the 'pure effect' of US Equities. 
- We are using the other factors to interpret SPY's behaviour - this will become apparent in the correlation analysis

Note: Our data must begin in 2007, as this is the year our UUP and HYG proxies were introduced; unfortunately, I was unable to find any data that would provide a representation of these factors before this date. 

Correlated factors, such as credit or Commodities, will be residualized against equity or interest rates to isolate pure exposures using simple regression models. 

### Methodology - Why a simple regression Model?

I will not be using any more complex models such as ARIMA, SARIMAX, LSTM, Ridge, Lasso, Elastic Net, etc. This is because our goal is not to capture variance using the model, but instead to extract the residuals from each regression we do. 

In layman's terms, we are not looking to predict with this model; instead, we are looking to explain.

By design, this means we are technically violating Gauss-Markov assumptions as we are purposefully looking for exogeneity, so that we can extract it. This would mean our residual extraction will have noise, but I am assuming it isn't very important. In financial time series, we often find that the assumptions are generally violated, resulting in heteroskedastic residuals. Depending on the data used, we are likely to encounter serial correlation. 



## TL;DR:
Built a macro factor model using tradeable proxies and residualized exposures, which will be extracted as a feature set to statistically identify US market regimes (bull, bear, sideways) for trading signal generation.

---

# Sample Results:
## Regression Results:
#### Rolling R-Squared for all factors on US Equity:
After making the EW rolling OLS regression, I plotted the rolling $R^2$ across the time series to get a feel for how the model performed. Note that we do have autocorrelation and heteroskedastic residuals, so by Gauss-Markov, OLS here is not BLUE, but it is the best thing we have. This was completed for each factor, where we had average rolling $R^2$ values of at least 0.65, except for the USD Strength and Commodities Factors, which I imagine have more global exposures. 
Two Sigma made their model represent global markets, where they included aspects such as Emerging Markets (EM) factors. I would imagine they would have a stronger fit in this area by using these external features. 
![Rolling R-Squared](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/R2_rolling.png)

## PCA Results
#### Explained Variance:
I ran PCA, hoping to reduce the size of the feature set. I have 7 factors, and I wanted to see if it was possible to compress the dataset for better interpretation. It turned out that 3 PCs explained around 70% of the variance amongst the 7 factors, and if we add PC4 to that, it will cumulatively explain around 80% variance, where the explained variance begins to diminish.
![Scree Plot displaying explained variance on an individual + cumulative basis](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/pca_scree_plt.png)

#### PCA Loadings:
It can be at times difficult to label PCs, even though we can see the weights; perhaps we can imply what a PC could be, but we should always take it with a grain of salt. I am going to suggest what the PCs could be based on what the weights are, although they could be wrong:
- PC1 - Macroeconomic Risk Premium
- PC2 - Equity vs Credit
- PC3 - USD Strength & Commodities combined (interesting point here is that both of these had the largest residual bandwidth on average
- PC4 - USD & Inflation
- PC5 - Pure Combined Credit Factor?
- PC6/7 - Reject, probably noise

![PCA Factor Loadings](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/pca_loadings.png)

#### Main PC Contributors Plotted with a Time Gradient
Given that the first 3 PCs explain 70% variance, I thought it could be interesting to plot their behaviour in 3D space. We see that through time, the PCs tend to cluster together. My thoughts are that this is because of our EW component in the regression. If we prioritize recent data, then the more recent it is, the more it can cluster together. 

![Main PC Contributors Plotted with a Time Gradient](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/3_pcs_time_gradient.gif)

#### Main PC Contributors Plotted with a Volatility Gradient
If we look at how these clusters change against volatility, we can see here that as volatility increases, the observations tend to deviate more from the cluster. This makes sense because at certain times, we can expect the factors to somewhat converge and exhibit similar behaviour. 

![Main PC Contributors Plotted with a Volatility Gradient](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/3_pcs_vol_gradient.gif)

#### PC Correlations + Original US Equity Factor
By design, PCA makes the feature set orthogonal. Plotting the matrix including the original US equity factor, we can see that correlations are weak, but present; interestingly enough, the only properties with a notable correlation are the noise components, which makes sense as they explain less variance, suggesting that they are closer to the original feature than others. 

![PC Correlations](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/pca_correlations.png)

#### Thoughts on the PCA Features:
I will save all 7 PCs and test how they work on regime models, then I will compare the tests to situations where I use only PCs 1-5, and another test using PCs 1-4, etc. This is because I am unsure how much explained variance we can lose. For example, I have read that 95% explained variance is sometimes a rule of thumb, but the problem with that is that I believe this would include noise in the regime model. 

Then again, we see evidently that the PCs explained variance diminishes after PC4, so if we only use PCs 1-4, our cumulative variance explained will be 80% of our original raw feature set, and if we include PC5, then it would explain 90%. 
Given I am unsure which path to take, it could be worth testing all to see if there are any notable differences in regime detection.  


---

# Closing Remarks 

I am not entirely sure if this factor model will serve as a solid representation of the U.S. equity market. My initial hypothesis was that the US Market Regimes are simply defined by volatility and returns. This is theoretically all the things we would need to determine a 'bull', 'bear', or 'sideways market. 

That being said, this factor model could have the potential to make this approach slightly more comprehensive. I am expecting to see more detailed features than the simple aforementioned labels. Perhaps there are periods of 'Crisis' which are different from a period of 'stress', perhaps there are times when we should be cautious in our trading, but not necessarily be in a crisis state. 

I will save all dataframes that I think could be useful in the regime model.

Perhaps we will also try to test this factor model against a simple baseline of VIX + returns as features to represent the SPY regimes. I also wish to test the raw residuals and the raw returns against our PCs to determine which feature set is optimal.

## Looking Forward

I am going to use these features to make a market regime detection system. I will also make many regime models using these different datasets, and possibly make new datasets to compare the regime detections. Then I plan on doing a comprehensive backtest of all the regime models to see which one is the most effective.

### Learning:

I have learned a lot in this project, getting a hands-on deep dive on building a simple model and residualizing the factors, this is an approach I had never considered before undertaking this project. I was also able to do a deeper dive on PCA, which has greatly fortified my knowledge in this area. I also really enjoyed analyzing the data from a static and rolling standpoint, where I found interesting relationships and insights throughout. 

Looking forward to the next step.
