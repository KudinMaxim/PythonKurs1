import requests
import pandas as pd
from bs4 import BeautifulSoup
pd.set_option('display.max_columns', None)

#Получаем страницу нужного нам сайта
r=requests.get('https://mfd.ru/marketdata/?id=5&group=16&mode=3&sortHeader=name&sortOrder=1&selectedDate=01.11.2019')
html=BeautifulSoup(r.content)
table=html.find('table', {'id': 'marketDataList'})
rows=[]
#Перебираем теги, получаем таблицу и выбираем из неё все значения по котировкам акций
trs=table.find_all('tr')
for tr in trs:
    tr=[td.get_text(strip=True) for td in tr.find_all('td')]
    if len(tr)>0:
        rows.append(tr)
#Вгружаем в DataFrame все списки и обозначаем колонки
data=pd.DataFrame(rows, columns=['Тикер', 'Дата', 'Сделки', 'С/рост', 'С/%', 'Закрытие', 'Открытие', 'Мин', 'Макс', 'АВГ', 'Шт', 'Руб', 'Всего'])
#Фильтруем и сортируем
data=data[data['Сделки']!='N/A']
data['С/%']=data['С/%'].str.replace('−', ('-')).str.replace('%', '').astype(float)
data=data.set_index('С/%')
data=data.sort_index(ascending=False)
#Находим по какому тикеру был максимальный рост числа сделок (в процентах) за 1 ноября 2019 года
print(data['Тикер'].head(1))
