# tobi_stock
stock_review


주식 코드 가져오기(전종목 코드 종목코드 종목명)
주식 데이터  2017.06.15 19:15  

국내 증시에 상장되어 있는 모든 주식의 코드가 필요한 경우에는 KRX에 가시면 관련 정보를 구할 수 있습니다.

 http://marketdata.krx.co.kr/mdi#document=040601 

코스피/코스닥 구분하여 다운 받을 수 있습니다.

필요에 따라서 전체를 받으시거나, 구분하여 받으시기 바랍니다.

2017/6/15일자 기준으로 다운 받은 종목명과 종목코드을 첨부합니다.


출처: http://tradingidea.tistory.com/entry/주식-코드-가져오기 [트레이딩]


아래 코드, 
print(len(kospi))
print(len(kosdaq))
print(len(file_list_kospi))
print(len(file_list_kosdaq))

789, 1281, 0, 466으로 카운팅.
실제로는 784개까지 저장됐음
야후 파이낸스 아직 오류가 있는 듯 함.
일단 인터넷에 저장된 값으로 사용하고, 나중에 HTS에서 받아야 됨.

import pandas as pd
pd.core.common.is_list_like = pd.api.types.is_list_like
from pandas_datareader import data
import fix_yahoo_finance as yf
yf.pdr_override()
import pickle
import re
import glob

kospi=pd.read_csv('kospi_180902.csv')[['종목코드','기업명']]
kosdaq=pd.read_excel('kosdaq_180902.xls')[['종목코드','기업명']]

def reload_empty_kosdaq():
    #file_list = glob.glob('./kosdaq/*.csv')
    
    for stock in kosdaq.values:
        ticker = str(stock[0])
        if ticker not in file_list_kosdaq:

            try:
                #kor_name = stock[0]
                #ticker = str(stock[0])
                df = data.get_data_yahoo(ticker + '.KQ', '1996-05-06', thread=20)
                df.to_csv('./kosdaq/'+ticker + '.csv')
                        
            except ValueError:  #raised if `y` is empty.
                pass

def reload_empty_kospi():
    #file_list = glob.glob('./kospi/*.csv')
    for stock in kospi.values:
        ticker = stock[0]
        if ticker not in file_list_kospi:
            #kor_name = stock[1]
            #ticker = stock[0]
            
            try:
                df = data.get_data_yahoo(ticker + '.KS', '1996-05-06', thread=20)
                df.to_csv('./kospi/'+ ticker + '.csv')

            except ValueError:  #raised if `y` is empty.
                 pass

file_list_kosdaq = []
file_list_kospi = []
stop_count=0

while len(file_list_kosdaq) < len(kosdaq):
    reload_empty_kosdaq()
    file_list_kosdaq = glob.glob('./kosdaq/*.csv')
    stop_count+=1;
    if stop_count>3:
        break

stop_count=0
        
while len(file_list_kospi) < len(kospi):
    reload_empty_kospi()
    file_list_kospi = glob.glob('./kospi/*.csv')
    stop_count+=1;
    if stop_count>3:
        break


######
# 아래는 파일 가져와서 해당 일자에 오른 정도 분석 하기

from fredapi import Fred
fred = Fred(api_key='285dc6e882d8acdd5241b34c2f83f6ff')
data = pd.DataFrame(fred.get_series('INTDSRKRM193N'))
data2 = data.loc[(data.index > '2005-01-01')&(data.index<'2009-01-01')]
#data2

# INTDSRKRM193N -> Interest Rates, Discount Rate for Republic of Korea
# 리먼 직전 금리 인상 시기로 조정함
# 2006-02-01, 올렸다가 다시 내림. 이후 2006-06-01에 다시 올림

# 2006-01-01: 2.00 -> 2006-02-01: 2.50 
# 2006-05-01: 2.25 -> 2006-06-01: 2.50 
# 2008-09-01: 3.50 -> 2008-10-01: 2.50 

#file_list = glob.glob('*.csv')
file_list = glob.glob('./kospi_kosdaq/*.csv')
review =[]

for stock in file_list:
    try:
        df=pd.read_csv(stock)
        a = df.loc[(df['Date']>='2006-06-01')&(df['Date']<='2006-06-31'),'Adj Close'].dropna().mean()
        b = df.loc[(df['Date']>='2008-09-01')&(df['Date']<='2008-09-30'),'Adj Close'].dropna().mean()    
        c = round(b/a*100, 2)
        review.append(c)
    except:
        print(stock +'is empty file!!!')
        review.append(0)
        pass

my_df = pd.DataFrame({'name':file_list, 'rate':review})
my_df = my_df.sort_values(['rate'], ascending=[False])
my_df

### 종목 정보 확인 필요
kosdaq=pd.read_excel('kosdaq_180902.xls')
kospi=pd.read_csv('kospi_180902.csv')
#kosdaq_dict = kosdaq.set_index('종목코드').to_dict()
#kosdaq_dict
kosdaq.set_index('종목코드', inplace=True)

#df.groupby('ID').apply(lambda x: x.to_dict('list')).reset_index(drop=True).to_dict()

kosdaq.groupby('종목코드')['기업명','업종','자본금(원)'].apply(lambda x: x.to_dict('list')).reset_index(drop=True).to_dict()
