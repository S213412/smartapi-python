# SMARTAPI-PYTHON

SMARTAPI-PYTHON is a Python library for interacting with Angel's Trading platform  ,that is a set of REST-like HTTP APIs that expose many capabilities required to build stock market investment and trading platforms. It lets you execute orders in real time..


## Installation

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install smartapi-python.

```bash
pip install -r requirements_dev.txt       # for downloading the other required packages
```

## Usage

```python
# package import statement
from SmartApi import SmartConnect #or from SmartApi.smartConnect import SmartConnect
import pyotp
from logzero import logger

api_key = '3L1oD6hx '
username = Your client code'
pwd = 'Your pin'
smartApi = SmartConnect(api_key)
try:
    token = "Your QR value"
    totp = pyotp.TOTP(token).now()
except Exception as e:
    logger.error("Invalid Token: The provided token is not valid.")
    raise e

correlation_id = "abcde"
data = smartApi.generateSession(username, pwd, totp)

if data['status'] == False:
    logger.error(data)
    
else:
    # login api call
    # logger.info(f"You Credentials: {data}")
    authToken = data['data']['jwtToken']
    refreshToken = data['data']['refreshToken']
    # fetch the feedtoken
    feedToken = smartApi.getfeedToken()
    # fetch User Profile
    res = smartApi.getProfile(refreshToken)
    smartApi.generateToken(refreshToken)
    res=res['data']['exchanges']

    #place order
    try:
      import pandas as pd

# Function to check entry condition
def entry_condition(df):
    return df['Close'][0] > df['Close'][-1] * 1.005 and df['Close'][0] > 0

# Function to check exit condition
def exit_condition(entry_price, current_price, stop_loss_percent=0.5, target_profit_percent=5.0):
    stop_loss = entry_price * (1 - stop_loss_percent / 100)
    target_profit = entry_price * (1 + target_profit_percent / 100)
    return current_price <= stop_loss or current_price >= target_profit

# Sample data (replace this with your actual historical price data)
data = {'Close': [100, 102, 101, 105, 103, 108, 107]}

df = pd.DataFrame(data)

# Loop through the data to simulate trading
for i in range(1, len(df)):
    current_time = pd.to_datetime(df.index[i])
    
    # Check entry condition
    if current_time.hour >= 9 and current_time.minute >= 25:
        if entry_condition(df.iloc[i-1:i+1]):
            entry_price = df['Close'][i]
            print(f"Buy 100 shares at {entry_price} on {current_time}")

    # Check exit condition
    if 'entry_price' in locals() and exit_condition(entry_price, df['Close'][i]):
        exit_price = df['Close'][i]
        print(f"Sell 100 shares at {exit_price} on {current_time}")
        del entry_price  # Reset entry_price after selling

    #gtt rule creation
    try:
        gttCreateParams={
                "tradingsymbol" : "SBIN-EQ",
                "symboltoken" : "3045",
                "exchange" : "NSE", 
                "producttype" : "MARGIN",
                "transactiontype" : "BUY",
                "price" : 100000,
                "qty" : 10,
                "disclosedqty": 10,
                "triggerprice" : 200000,
                "timeperiod" : 365
            }
        rule_id=smartApi.gttCreateRule(gttCreateParams)
        logger.info(f"The GTT rule id is: {rule_id}")
    except Exception as e:
        logger.exception(f"GTT Rule creation failed: {e}")
        
    #gtt rule list
    try:
        status=["FORALL"] #should be a list
        page=1
        count=10
        lists=smartApi.gttLists(status,page,count)
    except Exception as e:
        logger.exception(f"GTT Rule List failed: {e}")

    #Historic api
    try:
        historicParam={
        "exchange": "NSE",
        "symboltoken": "3045",
        "interval": "ONE_MINUTE",
        "fromdate": "2021-02-08 09:00", 
        "todate": "2021-02-08 09:16"
        }
        smartApi.getCandleData(historicParam)
    except Exception as e:
        logger.exception(f"Historic Api failed: {e}")
    #logout
    try:
        logout=smartApi.terminateSession('Your Client Id')
        logger.info("Logout Successfull")
    except Exception as e:
        logger.exception(f"Logout failed: {e}")

    ```

    ## Getting started with SmartAPI Websocket's

    ```python
    ####### Websocket V2 sample code #######

    from SmartApi.smartWebSocketV2 import SmartWebSocketV2
    from logzero import logger

    AUTH_TOKEN = "Your Auth_Token"
    API_KEY = "Your Api_Key"
    CLIENT_CODE = "Your Client Code"
    FEED_TOKEN = "Your Feed_Token"
    correlation_id = "abc123"
    action = 1
    mode = 1
    token_list = [
        {
            "exchangeType": 1,
            "tokens": ["26009"]
        }
    ]
    #retry_strategy=0 for simple retry mechanism
    sws = SmartWebSocketV2(AUTH_TOKEN, API_KEY, CLIENT_CODE, FEED_TOKEN,max_retry_attempt=2, retry_strategy=0, retry_delay=10, retry_duration=30)

    #retry_strategy=1 for exponential retry mechanism
    # sws = SmartWebSocketV2(AUTH_TOKEN, API_KEY, CLIENT_CODE, FEED_TOKEN,max_retry_attempt=3, retry_strategy=1, retry_delay=10,retry_multiplier=2, retry_duration=30)

    def on_data(wsapp, message):
        logger.info("Ticks: {}".format(message))
        # close_connection()

    def on_control_message(wsapp, message):
        logger.info(f"Control Message: {message}")

    def on_open(wsapp):
        logger.info("on open")
        some_error_condition = False
        if some_error_condition:
            error_message = "Simulated error"
            if hasattr(wsapp, 'on_error'):
                wsapp.on_error("Custom Error Type", error_message)
        else:
            sws.subscribe(correlation_id, mode, token_list)
            # sws.unsubscribe(correlation_id, mode, token_list1)

    def on_error(wsapp, error):
        logger.error(error)

    def on_close(wsapp):
        logger.info("Close")

    def close_connection():
        sws.close_connection()


    # Assign the callbacks.
    sws.on_open = on_open
    sws.on_data = on_data
    sws.on_error = on_error
    sws.on_close = on_close
    sws.on_control_message = on_control_message

    sws.connect()
    ####### Websocket V2 sample code ENDS Here #######

    ########################### SmartWebSocket OrderUpdate Sample Code Start Here ###########################
    from SmartApi.smartWebSocketOrderUpdate import SmartWebSocketOrderUpdate
    client = SmartWebSocketOrderUpdate(AUTH_TOKEN, API_KEY, CLIENT_CODE, FEED_TOKEN)
    client.connect()
    ########################### SmartWebSocket OrderUpdate Sample Code End Here ###########################
```
