# Stock_Data_attribute_average
# Compute the average closing price of stock of company (data access from yahoo finance data)

import re
import requests
import json
import pandas as pd
import time
from datetime import date
import matplotlib.pyplot as plt
from scipy.cluster.vq import*

def stock_data(stock_code):                   # defining a function to retrieve stock_data
    json_data = list()                        # an empty list 'json_data'
    url = 'https://finance.yahoo.com/quote/%s/history?p=%s' % (stock_code, stock_code)           # modifying the url with address/parameter=stock_code
    fhandle = requests.get(url)               # Getting the file handle of url
    str_data = re.findall('"HistoricalPriceStore":{"prices":(.*?),"isPending"', fhandle.text)      # returning the string (search_pattern) in string fhandle.text
    if str_data:                    # True if some value, False if empty
        json_data = json.loads(str_data[0])                # converting string data (string encasulating lists/dicts) into json data (a collection of list/dict)
        json_data = json_data[::-1]                        # reversing the order/indexing of json_data
    return [item for item in json_data if not 'type' in item]          # returning the json data (a list/dict) containing no string "type"

json_data = stock_data('KO')               #calling the function of stock_data with company code='KO'

str_date = list()                         # an empty list for string date
for i in range(len(json_data)):            # iterate through the length of json_data
    time_format = date.fromtimestamp (json_data[i]['date'])             # Extracting the date from json_data in time format
    str_format = date.strftime (time_format, '%Y-%m-%d')                # change time format date into string format
    str_date.append(str_format)                                         # make the lsit of string format date
json_data_df_original = pd.DataFrame(json_data, index = str_date)       # converting the json data into data frame with indes as string dates
json_data_df = json_data_df_original.drop (['date'], axis =1)           # delete the date column from the original data frame

list_month = list()                                   # an empty list for months
for i in range(len(json_data_df)):                         # iterate through the length of json_data_df
    temp = time.strptime (json_data_df.index[i], "%Y-%m-%d")          # convert the date form string format into time format
    list_month.append(temp.tm_mon)                                    # append the month in list_month

temp_df = json_data_df.copy()                      # make the copy of json_data_df
temp_df['month'] = list_month                      # add column "month" containgn list_month in temp_df
avg_close_price = temp_df.groupby('month').close.mean()            # avg of column 'close' of temp_df (groupby w.r.t months)
plt.plot(avg_close_price.index,avg_close_price.values)            # ploting the average close price with its index
plt.title("Stock Data of company 'KO'")
plt.xlabel('Months')
plt.ylabel('Average Closing Price of Stock')
plt.show()
plt.savefig('1.jpg')                                 # save the figure as jpeg file
