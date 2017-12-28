# -*- coding: utf-8 -*-
# @Time    : 17-12-26 上午8:59
# @Author  : Lin
# @Email   : wujialin5@huawei.com
# @File    : eda1.py
# @Software: PyCharm

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.cluster import KMeans

def poi():
    poi = pd.read_csv('../data/poi.csv', header=None)
    headers = ['geo_id', '加油站', '超市', '住宅区', '地铁站', '公交站', '咖啡厅', '中餐厅', 'ATM', '写字楼', '酒店']
    poi[range(0, 21, 2)].to_csv('poi_header.csv', index=False, header=headers)

def analy_poi():
    data = pd.read_csv('train_july.csv')
    sns.boxplot(data['start_geo_id'], data['count'])
    plt.show()

def isweekday(date):
    if date.weekday()>4:
        return 1
    return 0

def disc_hour(date):
    hour = date.hour
    if hour<8:
        return 0
    elif hour<15:
        return 1
    else:
        return 2

def wea_null_pro():
    wea = pd.read_csv('../data/weather.csv', parse_dates=['date'])
    wea.drop(['text', 'feels_like', 'humidity', 'wind_direction_degree', 'wind_speed'], axis=1, inplace=True)
    for name, type in wea.dtypes.iteritems():
        if type == object:
            for i, cate in enumerate(wea[name].unique()):
                 wea.loc[wea[name]==cate, name+'_code'] = i+1
            wea.drop([name], axis=1, inplace=True)
            wea.rename(columns={name+'_code': name}, inplace=True)

    date_range = pd.date_range('2017-7-1', '2017-8-8', freq='0.5H')[:-1]
    date_range = pd.DataFrame({'date': date_range})
    date_range['day'] = date_range['date'].apply(isweekday)
    date_range['hour'] = date_range['date'].apply(lambda x: x.hour)
    date_range['hour'] = date_range['date'].apply(disc_hour)
    date_range = pd.merge(date_range, wea, on=['date'], how='left')
    date_range.fillna(method='bfill', inplace=True)
    date_range = pd.get_dummies(date_range, columns=['code', 'wind_direction', 'day', 'hour'])
    date_range.to_csv('weather.csv', index=False)



if __name__ == '__main__':
    analy_poi()
    # wea_null_pro()


