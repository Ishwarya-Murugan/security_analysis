# <center> Dimensional Model for Security Analysis </center> 
The repository is part of the Security Analysis project.

The repository has the code and design to store the security information for Stocks, ETFs and Mutual Funds, I'm currently tracking manually for security analysis.

## Data Model Design Overview

![image](https://user-images.githubusercontent.com/36078795/138581501-3b65cbbe-de65-44c8-86dd-2cb0ee876787.png)


### Business Process
Create a data model to store the information on Securities traded in US stock market for analysis. The securities could be of different type like Stocks, ETFs or Mutual Funds etc.

### Dimensional Tables
The below will be the conforming dimensions which would be used to drill-down and roll-up the details on all the fact tables in the Security Analysis project.
  - date_dim 
  - security_dim
  - account_dim
  
### Fact Tables
The below fact tables will get loaded at 05:00 PM EST of the last day of the month irrespective of the holiday or weekend.
  - security_fct (periodic snapshot)
  - account_trade_fct (periodic snapshot)
  - account_cost_basis_fct (accumulating snapshot)
  - security_portfolio_fct (periodic_snapshot)
  
The below is the transactional fact table which will get a record loaded whenever the trade is made
  - account_trade_fct (transactional)

## Dimensional Table Design
### Date Dimension
**Database name:** `security_analysis`  

**Table Name:** `date_dim`      
*reference: https://www.codeproject.com/Articles/647950/Create-and-Populate-Date-Dimension-for-Data-Wareho*

<b>Columns:</b>
<pre>
date_key         INT   primary key    
date             DATETIME    
day_of_month     VARCHAR(2)
day_name         VARCHAR(9)
day_of_quarter   VARCHAR(3)
day_of_year      VARCHAR(3)
month            VARCHAR(2)
month_name       VARCHAR(9)
month_of_quarter VARCHAR(2)
quarter          CHAR(1)
quarter_name     VARCHAR(9)
year             CHAR(4)
year_name        CHAR(7)
month_year       CHAR(10)
mmyyyy           CHAR(6)
</pre>

**Table Usage:** The table will act as a role playing dimensions in the below fields.
1) `ipo_date` and `inception_date` in `security_dim` table
2) `record_date` in the `security_fct` table

### Security Dimension
**Database name:** `security_analysis`  

**Table Name:** `security_dim`      

<b>Columns:</b>
<pre>
symbol           VARCAHR(8)   primary key    
name             VARCAHR(30)
security_type    VARCHAR(10)
category         VARCHAR(30)
industry         VARCHAR(20)
index_tracked    VARCHAR(30)
inception_date   INT  references date_dim(date_key)
ipo_date         INT  references date_dim(date_key)
fund_family      VARCHAR(30)
trading_flag     CHAR(1)
</pre>

**Table Usage:** The table will store the attributes for the different securites (stocks, ETF and mutual fund).

**Slowly Changing Dimension**
The below attribure could change over time. The table will follow 'Type-1' update for the updated records and overwite the old with new record. 
1) `industry` - could be updated due to merger
2) `trading_flag` - could change if the company become private or merged

### Account Dimension
**Database name:** `security_analysis`  

**Table Name:** `account_dim`      

<b>Columns:</b>
<pre>
account_id          VARCHAR(10)   primary key
firm_name           VARCAHR(30)     
account_type        VARCAHR(20)  
account_open_date   INT  references date_dim(date_key) 
account_close_date  INT  references date_dim(date_key) 
account_status      VARCHAR(10)
retirement_flag     VARCHAR(20)
roth_flag           CHAR(1)
tax_deferred        CHAR(1)
hsa_flag            CHAR(1)
401k_flag           CHAR(1)
</pre>

**Table Usage:** The table will store the attributes for the different accounts (retirement, individual, hsa etc).

**Slowly Changing Dimension**
The below attribure could change over time. The table will follow 'Type-1' update for the updated records and overwite the old with new record. 
1) `account_close_date` and `account_status`- will be update when the account is closed

## Fact Table Design
### Security Fact
**Database name:** `security_analysis`  

**Table Name:** `security_fct`      

<b>Columns:</b>
<pre>
symbol                  VARCAHR(8)  references security_dim(symbol)    
record_date             INT  references date_dim(date_key)  
market_value            DECIMAL(7,2)
expense_ratio           DECIMAL(5,2)
dividend_yield_pct      DECIMAL(5,2)
forward_dividend        DECIMAL(5,2)
pe_ratio                DECIMAL(5,2)
peg_ratio               DECIMAL(5,2)
52_week_low             DECIMAL(9,2)
52_week_low_date        INT  references date_dim(date_key)
52_week_high            DECIMAL (9,2)
52_week_high_date        INT  references date_dim(date_key)
beta_value              DECIMAL (3,2)
holding_turnover        DECIMAL(11,2)
in_nasdaq100            CHAR(1)
in_sp500                CHAR(1)
in_dow30                CHAR(1)
held_by_etf             CHAR(1)
top_etf_by_exposure     VARCHAR(8)
top_etf_name            VARCHAR(30)
top_etf_exposure_pct   DECIMAL(5,2)
earning_per_share       DECIMAL(5,2)
market_cap              DECIMAL(11,2)
net_assets              DECIMAL(15,2)
net_asset_value         DECIMAL(7,2)
equity_summary_score    DECIMAL(3,2)
morningstar_rating      CHAR(1)
morningstar_risk_rating CHAR(1)
sustainability_rating   CHAR(1)
</pre>

**Table Usage:** The table will store the security fact detail values at end of every month. It is a monthly snapshot fact table.

### Account Trade Fact
**Database name:** `security_analysis`  

**Table Name:** `account_trade_fct`      

<b>Columns:</b>
<pre>
account_id              VARCHAR(10) references account_dim(account_id) primary key
symbol                  VARCAHR(8)  references security_dim(symbol) primary key
trade_date              INT  references date_dim(date_key) primary key
unit_cost               DECIMAL(7,2) primary key
trade_quantity          DECIMAL(5,2)
trade_type              VARCHAR(5)
total_trade_cost        DECIMAL(7,2)
</pre>

**Table Usage:** The table will store buy and sell transaction made on all the accounts. It is a transactional fact table with grain at trade level.

### Account Cost Basis Fact
**Database name:** `security_analysis`  

**Table Name:** `account_cost_basis_fct`      

<b>Columns:</b>
<pre>
account_id              VARCHAR(10) references account_dim(account_id) primary key
symbol                  VARCAHR(8)  references security_dim(symbol) primary key
record_date             INT  references date_dim(date_key) primary key
current_quantity        DECIMAL(5,2)
total_market_value      DECIMAL(9,2)
avg_purchase_price      DECIMAL(7,2)
avg_selling_price       DECIMAL(7,2)
realized_gain_loss      DECIMAL(7,2)
unrealized_gain_loss    DECIMAL(7,2)
dividend_earned         DECIMAL(7,2)
</pre>

**Table Usage:** The table will store cost_basis for buy and sell transaction made on all the accounts. The grain is at the symbol inside each account level. It is a monthly snapshot fact table.

### Security Portfolio Fact
**Database name:** `security_analysis`  

**Table Name:** `portfolio_security_fct`      

<b>Columns:</b>
<pre>
symbol                  VARCAHR(8)  references security_dim(symbol) primary key
record_date             INT  references date_dim(date_key) primary key
current_quantity        DECIMAL(5,2)
total_market_value      DECIMAL(9,2)
avg_purchase_price      DECIMAL(7,2)
unrealized_gain_loss    DECIMAL(7,2)
</pre>

**Table Usage:** The table will store current security count and market_value in the portfolio at the end of every month. The grain is at the security level. It is a monthly snapshot fact table.
