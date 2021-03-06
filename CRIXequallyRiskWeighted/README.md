[<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/banner.png" width="888" alt="Visit QuantNet">](http://quantlet.de/)

## [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **CRIXequallyRiskWeighted** [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/)

```yaml

Name of Quantlet : CRIXequallyRiskWeighted

Published in : Portfolio Optimization with crypto-currencies

Description : 'Cumulative return with the equally risk weighted strategy and SP500 core with a window of 100 days'

Keywords : tedas, ERC, risk weighted, portfolio, optimization, cryptocurrency, return

Author : SIVAGOUROU Dinesh

Submitted : 2018/08/24

Datafile :
- cryptos data
- SP500 core

Output :
- Plot of the cumulated return of the ERC strategy and core wealth.

```

![Picture1](CRIXequallyRiskWeighted.png)

### R Code
```r

# clear all variables
rm(list = ls(all = TRUE))
graphics.off()

# install and load packages
libraries = c("FRAPO", "quantreg", "plotly")
lapply(libraries, function(x) if (!(x %in% installed.packages())) {
  install.packages(x)
})
lapply(libraries, library, quietly = TRUE, character.only = TRUE)


### Initial Parameters 
tau=c(0.05, 0.15, 0.25, 0.35, 0.50) # <- Default setting
window_length = # <- to modify
  
### tau spine function
tau_spine=function(x,tau){
  return(min(tau[tau >= x]))
}  
  
### Function for plotting
plot_wealth = function(strategy_wealth,core,strategy_string,core_string,date_range,windowLength){
    
    core_wealth = cumsum(c(1,core[windowLength:(length(core))]))
    core_wealth  = core_wealth[-length(core_wealth)]
    
    plot_ly(x = date_range[windowLength: length(date_range)]) %>%
      add_lines(y = core_wealth[1 : (length(date_range)-windowLength+1) ], name = paste("Core ",core_string),mode="lines")%>%
      add_lines(y = strategy_wealth[1 :( length(date_range)-windowLength+1 )], name = strategy_string,line = list(mode = "lines"))%>%
      layout(title = paste("Cumulative return for",core_string, "with window length=",toString(windowLength),sep=" "))
    
    
    
}

    
### Function RISK_WEIGHTED_ALLOCATION
RISK_WEIGHTED_ALLOCATION = function(data_cryptos,l,core,tau){
  
  Wealth = c(1,rep(0,length(data_cryptos[,1])-l+1))
  
  
  for (i in l : length(data_cryptos[,1])){
    
    Fn=ecdf(core[(i-l+1):i])
    
    if( tau_spine(Fn(core[i]),tau) != Inf ){
      
      model =rq(formula = core[(i-l+1):i] ~ ., tau = tau_spine(Fn(core[i]),tau), data = data_cryptos[(i-l+1):i,])
      
      assets_beta_negative = model$coefficients[(model$coefficients < 0) & names(model$coefficients)!="(Intercept)"]
      
      selected_assets = data_cryptos[(i-l+1):i,names(assets_beta_negative)]
      
      cov_selected_assets = cov(selected_assets)
      
      if(!is.null(assets_beta_negative)){
        
        
        weight =  as.numeric(Weights(PERC(cov_selected_assets,percentage = FALSE)))
        
        X_i_plus_1 = data_cryptos[i+1,names(assets_beta_negative)[2:length(names(assets_beta_negative))]]
        
        
        Wealth[i-l+1 +1] = Wealth[i-l+1] + sum(weight*X_i_plus_1)
        
      }
      
      else{
        
        Wealth[i-l+ 2] = Wealth[i-l+1] 
        
      }
      
    }
    
    else  {
      
      
      Wealth[i-l+2] = Wealth[i-l+1] + core[i+1]
      
      
    }
    
  }
  
  return(Wealth[-length(Wealth)])
  
}


##################################                RESULTS            #########################################



Wealth_RISK_WEIGHTED = RISK_WEIGHTED_ALLOCATION(data_cryptos,window_length,SP500)
Wealth_RISK_WEIGHTED = RISK_WEIGHTED_ALLOCATION(data_cryptos,window_length,NASDAQ)
Wealth_RISK_WEIGHTED = RISK_WEIGHTED_ALLOCATION(data_cryptos,window_length,FTSE100)
Wealth_RISK_WEIGHTED = RISK_WEIGHTED_ALLOCATION(data_cryptos,window_length,SP400_TECH_DISTRIBUTRS)
Wealth_RISK_WEIGHTED = RISK_WEIGHTED_ALLOCATION(data_cryptos,window_length,SP_EURO_INFO_TECH)



plot_wealth(Wealth_RISK_WEIGHTED,SP500,"Wealth_RISK_WEIGHTED","SP500",Core$date,window_length)
plot_wealth(Wealth_RISK_WEIGHTED,NASDAQ,"Wealth_RISK_WEIGHTED","NASDAQ",Core$date,window_length)
plot_wealth(Wealth_RISK_WEIGHTED,FTSE100,"Wealth_RISK_WEIGHTED","FTSE100",Core$date,window_length)
plot_wealth(Wealth_RISK_WEIGHTED,DAX_30_PERFORMANCE,"Wealth_RISK_WEIGHTED","DAX_30_PERFORMANCE",Core$date,window_length)
plot_wealth(Wealth_RISK_WEIGHTED,NIKKEI_225_STOCK_AVERAGE,"Wealth_RISK_WEIGHTED","NIKKEI_225_STOCK_AVERAGE",Core$date,window_length)


```

automatically created on 2018-09-04