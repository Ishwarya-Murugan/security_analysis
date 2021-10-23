# <center> Dimensional Model for Securities </center> 
The repository is part of the Security Analysis project.

The repository has the code and design to store the security information for Stocks, ETFs and Mutual Funds, I'm currently tracking manually for security analysis.

## Data Model Design Overview

### Business Process
Create a data model to store the information on Securities traded in US stock market for analysis. The securities could be of different type like Stocks, ETFs or Mutual Funds etc.

### Grain
The dimensional and fact tables will have grain of one record for each security.

### Dimensional Tables
The below will be the conforming dimensions which would be used to drill-down and roll-up the details on all the fact tables in the Security Analysis project.

Security_dim would be a slowly-changing dimension table with type- 1 update.
  - date_dim 
  - security_dim
  
### Fact Tables
The fact table will get loaded at 05:00 PM EST of the last day of the month irrespective of the holiday or weekend.
  - security_fact

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
2) `record_date` in the `security_fact` table

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

## Fact Table Design
### Security Fact
**Database name:** `security_analysis`  

**Table Name:** `security_fact`      

<b>Columns:</b>
<pre>
symbol                  VARCAHR(8)  references security_dim(symbol)    
record_date             INT  references date_dim(date_key)  
expense_ratio           DECIMAL(5,2)
dividend_yield_pct      DECIMAL(5,2)
forward_dividend        DECIMAL(5,2)
pe_ratio                DECIMAL(5,2)
peg_ratio               DECIMAL(5,2)
52_week_low             DECIMAL(9,2)
52_week_high            DECIMAL (9,2)
beta                    DECIMAL (3,2)
holding_turnover        DECIMAL(11,2)
in_nasdaq100            CHAR(1)
in_sp500                CHAR(1)
in_dow30                CHAR(1)
held_by_etf             CHAR(1)
top_etf_by_exposure     VARCHAR(8)
top_etf_name            VARCHAR(30)
top_etf_expossure_pct   DECIMAL(5,2)
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
