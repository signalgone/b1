+++
title = 'Prediction of next day realized volatility from option information (Implied Volatility, Skewness and Greeks)'
date = 2026-10-24T20:48:51+08:00
draft = true
+++

# Introduction
Options tend to have information that are related to how trader perceived the current 
state of volatility outlook into the future and it allows players to hedge their 
risk as well. Puts option tend to be more skew when compare to calls option, 
which then formed the volatility smile as puts is able to provide the downside 
protection. The aim was to uses put option close price and volatility informations 
to predict the next day realized volatility (of rolling 30 trading days). 
The prediction of realized volatility can be used for portfolio optimization 
and risk management.

# Black Scholes Implied Volatility (IV) and Realized Volatility
The Black Scholes formula is 
$$dS_{t}=rS_{t}dt+\sigma S_{t}dW_{t}$$

So, if we have the current stock price $S$,  strike price $K$, risk-free interest 
rate $r$ and maturity $t$, we could reverse the formula to obtained $\sigma$ by 
Brent method.(We can use Brent's method to optimized ?????) The $\sigma$ here is the
implied volatility of the option price.

In Black Scholes formula, $\sigma$ is assumed to be remain constant through all 
strike price and all maturity. However, in reality, when we obtained the implied 
volatility through different strike price and maturity, we will see that they are 
not constant at all. They will varies, that is why we can build a volatility surface
 across strike prices and maturity.
 


Realized Volatility ????

why i use 30 day???

# Feature engineering
I managed to feature engineered a few features from the option closed price, like implied volatility, volatility skew (mixon skew), implied probability, option greeks, put-call volume ratios and discount rate.

Mixon skew is one of the volatility skew measure. It measures how the volatility changes across the strike. Mathematically, it is 
$$\dfrac{\text{25 Delta put IV}-\text{ 25 Delta call IV}}{\text{50 Delta put IV}}$$ 
This skew will allow us to understand how the volatility smile is being formed, as out of money puts are usually priced higher which resulted in a higher IV than out of money calls. 

For implied probability, I uses Breeden-Litzenberger formula, which is able to convert puts prices into an implied probability density function. We first have to find the implied volatility, as mention before, and spline using gaussian spline to smooth the volatility surface across strikes. We uses the implied volatility to find the implied probability ?????

For interest rate, I was aware of how OptionMetrics does the zero rate curve by bootstrapping to liquid money market instruments. However, I do not have access to them, so I will be using treasury bills (1 month, 3 month, 6 month, 9 month, 12 months) and uses spline to form my zero rates curve.

I have also uses put-call volume ratios as one of the feature. This was done by using put volumes divided by call volumes.

As for the option greeks, I will be using delta, gamma, vega, theta, rho, vomma, charm, ?????

??? correlation between features

# Train and test set



# The Prediction Models
## Model 1 - Ordinary Least Squares (OLS): 
OLS is mostly being used in the financial quant space. As it is fast, easily understand and able to have non linearity effect by squaring the same features or multiple by two or more features.
For Ordinary Least Squares, I uses a partial regression concept to find variables that are suitable to fit into OLS. So, the idea is very similar to FWL Theorem, where by first, I have the full model, $$\hat{Y}=\hat{\beta}_{0}+\hat{\beta}_{1}^{ols}X_{1}+\hat{\beta}_{2}^{ols}X_{2}$$ where $X_{2}$ is the interest of variable here. 
So, I drop it and find the new $\hat{Y}_{a}$ which is 
$$\hat{Y}_{a}=\hat{\beta}_{0}+\hat{\beta}_{1}^{ols}X_{1}$$ 

Next, I will find the residual of this partial regression, $Y_{res}=Y-\hat{Y}_{a}$, I uses this residual $Y_{res}$ and regress on $X_{2}$. So if the coefficient of $X_{2}$ is statistically significant (p-value $<0.05$), it will remain in the full model. 
As from from the full model of $Y$ with $\hat{Y_{a}}$ and $Y_{res}$, 
$$Y=\hat{Y_{a}}+Y_{res}=\underbrace{\hat{\beta}_{0}+\hat{\beta}_{1}^{ols}X_{1}}_{\text{correlated part to Y}}+\underbrace{Y_{res}}_{\text{the uncorrelated part to Y}}$$

So, when we tested the new variable against $Y_{res}$, we are checking if this new variable is able to explained the uncorrelated part that has not explained by any variables that we included. However, this method tend to have create multicollinearity, we will have to apply principal component analysis (PCA) on the selected variables to orthogonalized them.
so I add one variable by one variable, starting from the highest correlated variable to the target and tested up till all interaction terms ($X_{1}X_{2}$) and squared terms of itself ($X_{1}^{2}$). 

<mention the variable selected??>

I had also run OLS on all variables, excluding interaction terms and square of the same variables.


## Model 2 - Ridge regression:
Ridge regression is a penalized linear regression.
As ridge regression introduce $L_2$ norm penalty to the covariance matrix ($X^{T}X$).
$$\beta^{\text{ridge}}=(X^{T}X+I\lambda)^{-1}XY$$
It will shrink the coefficients of the regression when compare to OLS. 

I uses the same variables that were selected by the same partial regression concept from OLS. However, we did not use PCA here as Ridge regression is able to handle multicollinearity.
The model is optimized by 5 cross fold to find the alpha values in the statsmodel package. I would also run Ridge regression on all variables, excluding interaction terms and square of the same variables.



## Model 3 - Long-Short Term Memory (LSTM) :
For LSTM, I aggregate the data twice, once weighted all the features with moneyness, and then after that I weighted all the features with inverse of log time $(1/\log(t))$. This is to reduce the weightage for those options that have a longer duration, as the option prices in those might not be accurate, thus, affecting the implied volatility and other option information.

The model is fitted with 3 layers of LSTM and 1 dense layer. It is hyperparameter tuned by optuna to find the optimized parameters.

## Model 4 - LightGBM:
LightGBM will be able to extract out the non linearity relationship between the features and the realized volatility as we have shown in the our OLS that there are some nonlinearity.

I have LightGBM ran on both full data and the aggregated dataset that LSTM was ran on.

It is hyperparameter tuned by optuna to find the optimized parameters.

# Results



# Conclusion



# Reference



