# US-Macro-Factor-Model

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

Note: Our data must begin in 2007 due to the inception of our UUP and HYG proxies; unfortunately, I was not able to find any data that would provide a representation of these factors with an earlier date. 

Correlated factors like Credit or Commodities will be residualized against Equity/Interest Rates to isolate pure exposures using simple regression models. 

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
It can be at times difficult to label PCs even though we can see the weights, perhaps we can imply what a PC could be but we should always take it with a grain of salt. I am going to suggest what the PCs could be based on what the weights are although they could be wrong:
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
By design PCA makes the feature set orthogonal. Plotting the matrix including the original US equity factor, we can see that correlations are weak, but present, interestingly enough the only properties with a notable correlation are the noise components, which makes sense as they explain less variance, suggesting that they are closer to the original feature than others. 

![PC Correlations](https://github.com/jack-bell1/US-Macro-Factor-Model/blob/main/pca_correlations.png)
