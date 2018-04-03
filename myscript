from bs4 import BeautifulSoup
import requests
from urllib.request import urlopen
import re
import csv
from urllib.parse import urljoin
from urllib.error import HTTPError
import json
from urllib.error import URLError
from datetime import datetime
import sys, getopt

def convert_date(date):
    string_date =date['DisseminationDate']

    datetimeobject = datetime.strptime(string_date, '%d-%b-%y')
    newformat = datetimeobject.strftime('%Y-%m-%d')

    date['DisseminationDate']=newformat


def convert_json_date(date):
    datetimeobject = datetime.strptime(date, '%d-%b-%y')
    newformat = datetimeobject.strftime('%Y-%m-%d')

    return newformat

def prepare_json_pcp(standard,records ):
    dic = {}
    for col in standard:
        if col == '':
            continue
        dic[col] = ""
    for record, col, i in zip(records, standard, range(len(records))):
        if i == 0:
            continue
        elif i == 1:
            dic['name'] = record
        elif i == 2:
            dic['sector'] = record
        elif i == 3:
            dic['rating']=record
        elif i == 4:
            dic['date'] = record
        elif i == 5:
            dic['lt_rating'] = record
        elif i == 6:
            dic['st_rating'] = record
        elif i == 7:
            dic['action'] = record
        elif i == 8:
            dic['outlook'] = record
        elif i == 9:
            dic['press_link'] = record
        elif i == 10:
            dic['report_link'] = record
        elif i == 11:
            dic['history_link'] = record
        else:
            dic[col] = record


    text=dic['date']
    dic['date']=convert_json_date(text)

    return dic

def prepare_json(standard,records ):
    dic={}
    for col in standard:
        if col=='':
            continue
        dic[col]=""
    for record,col,i in zip(records,standard,range(len(records))):
        if i==0:
            dic['name']=record
        elif i== 1:
            dic['date'] = record
        elif i==2:
            continue
        elif i==5:
            dic['outlook'] = record
        elif i==6:
            dic['action']=record
        elif i==8:
            dic['history_link']=record
        elif i==9:
            dic['report_link'] = record
        elif i == 10:
            dic['sector'] = record
        else:
            dic[col]=record

    return dic


def dum_to_json(all_data):
    standard=['date','name','sector','lt_rating','st_rating','action','outlook','press_link','report_link','history_link','rating','']
    data={}
    data_list=[]
    pcp_list=[]
    for data,i in zip(all_data,range(len(all_data))):
        if i==0:
            for record in data:
                data_list.append(prepare_json(standard,record))

        elif i==1:
            for record in data:
                pcp_list.append(prepare_json_pcp(standard, record))

            continue
    with open('data.json', 'w') as outfile:
        filter_dic_json(pcp_list,'Entity')

        for dic in data_list:
            dic.popitem()
        for dic in pcp_list:
            dic.popitem()
        json.dump(data_list+pcp_list, outfile)
        #json.dump(pcp_list, outfile)

def write_to_csv(all_data):
    with open('output.csv', 'w') as csvfile:
        fieldnames = ['date','name','sector','lt_rating','st_rating','action','outlook','press_link','report_link','history_link']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

        writer.writeheader()
        for j ,data in zip(range(len(all_data)),all_data):
            if (j==0):
                for i in range(len(data)):
                    temp=data[i]
                    writer.writerow({'date': temp['Date'], 'name': temp['Name'], 'sector': temp['sector'],
                                     'lt_rating': temp['LongTerm'], 'st_rating': temp['ShortTerm'],
                                     'action': temp['Action'], 'outlook': temp['Outlook'],
                                     'press_link': temp['PressRelease'], 'report_link': temp['RatingReport'],
                                     'history_link': temp['History']})
            else:
                for i in range(len(data)):
                    temp = data[i]

                    convert_date(temp)

                    writer.writerow(
                        {'date': temp['DisseminationDate'], 'name': temp['Entity'], 'sector': temp['Industry'],
                         'lt_rating': temp['LTRating'], 'st_rating': temp['STRating'], 'action': temp['Action'],
                         'outlook': temp['Outlook'], 'press_link': temp['PressRelease'], 'report_link': temp['Report'],
                         'history_link': temp['History']})

def filter_dic(data_dic,cond):
    data_dic[:] = [d for d in data_dic if d.get('RatingType') == cond]
    return data_dic
def filter_dic_json(data_dic,cond):
    data_dic[:] = [d for d in data_dic if d.get('rating') == cond]
    return data_dic

def scrape_pacra():
    try:
        r = requests.get('http://www.pacra.com.pk/reports.php')

    except HTTPError as e:

        print(e)

    except URLError:

        print("Server down or incorrect domain")

    else:
        data=[]
        text=[]
        soup = BeautifulSoup(r.content, 'lxml')
        All_table = soup.find_all("table")
        div=soup.find("div", {"id": "mainDiv"})
        for elem in div :
            if elem.name=='div':
                trs =elem.find_all('tr')
                for tr in trs :
                    for td in tr :
                        if td.text=='view' or td.text=='History':
                            try:
                                link= td.a.get('href')
                                link = urljoin('http://www.pacra.com.pk/', link)
                                text.append(link)
                            except:
                                text.append(td.text)
                        else:
                            text.append(td.text)

                    data.append(list(text))
                    text.clear()
    return data

def initialize_columns(columns):
    try:

        r  = requests.get('http://jcrvis.com.pk/ratingSect.aspx')

    except HTTPError as e:

        print(e)

    except URLError:

        print("Server down or incorrect domain")
    else:

        soup = BeautifulSoup(r.content, 'lxml')
        fields = soup.find("tr", {"class": 'fields'})
        rows = fields.find_all('th')
        for row in rows:
            columns.append(row.text)
        tbody = soup.find_all("tbody")
        #links =tbody.find_all('tr',{'class':'files'})
        for tr,i in zip(tbody,range(0,1)):

            links=tr.find_all('li')
            for link in links:
                columns.append(link.text)
        for i in range(len(columns)):
            columns[i] = re.sub('[^A-Za-z0-9]+', '', columns[i])

        columns.append('sector')

        return columns

def todic(record,columns):
    dict={}
    for cell,col in zip(record,columns):
        dict[col]=cell


    return dict

def parse_jcrvis():

    try:

        r  = requests.get('http://jcrvis.com.pk/ratingSect.aspx')

    except HTTPError as e:

        print(e)

    except URLError:

        print("Server down or incorrect domain")
    else:

        soup = BeautifulSoup(r.content, 'lxml')


        tags = soup.findAll("div", {"class": "ratings-data"})

        All_table=soup.find_all("table")
        checker = None



        dic={}
        data=[]
        data_p=[]
        text=[]
        href=[]
        reference=[]
        for item in All_table:
            for elem in item :


                if elem.name =='thead':
                    try:
                        if elem['id']=='Corporates' and elem['class'][0]=='sector-type':
                            checker=True

                    except:
                        continue
                if elem.name == 'thead' and checker == True:
                    try:
                        if elem['id'] != 'Corporates' and elem['class'][0] == 'sector-type':
                            checker = False

                    except:
                        continue
                if elem.name == 'thead' and checker == True:
                    try:
                        if elem['class'][0] == 'sector-header':
                            sec=elem.find_all('td')
                            for head in sec:
                                sector=head.text
                    except:
                        continue
                if elem.name=='tbody'and checker== True:

                    try:
                        rows = elem.find_all('tr')
                        for tr in rows:

                            try:
                                if (tr['class'][0] == 'files'):
                                    links = tr.find_all('li')
                                    counter=0
                                    for link in links:
                                        counter=counter+1
                                        link=link.a.get('href')
                                        link= urljoin( 'http://jcrvis.com.pk/',link)
                                        text.append(link)

                                    if counter<3:
                                        text.append('null')
                                    text.append(sector)
                                    data.append(list(text))
                                    text.clear()
                            except:
                                continue

                            try:
                                if (tr['class'][0] == 'data'):
                                    tds=tr.find_all('td')
                                    for td in tds:
                                        text.append(td.text)


                            except:
                                continue
                    except:
                        print("no tbody")



    return  data

def cleandata(data):
    for record in data:
        for col in record :
            #col=re.sub('[^A-Za-z0-9]+', '', col)
            col=col.rstrip()


    for i in range(len(data)):
        for j in range (len (data[i])):
            data[i][j]=data[i][j].replace("\r\n",'')
            data[i][j] = data[i][j].rstrip()
            data[i][j] = data[i][j].strip()
    return data

def main (arg):



    raw_data = []
    list_dic=[]
    data_dic=[]
    columns = []
    new_coloumn=[]
    All_data = []

    columns = initialize_columns(columns)
    data=parse_jcrvis()
    data=cleandata(data)# scrape first  website

    for record in data:
        list_dic.append(todic(record,columns))#converting  to dictionary for faster retrieval

    data_p=scrape_pacra()# scrape second website

    for cell in data_p[0]:
        new_coloumn.append(cell)
    data_p=data_p[1:]
    raw_data.append(data)
    raw_data.append(data_p)
    for record in data_p :
        data_dic.append(todic(record,new_coloumn))

    data_dic=filter_dic(data_dic,'Entity') # filter data on the basis of entity parameter can be changed
    All_data.append(list_dic)
    All_data.append(data_dic)
    if arg[0].endswith('.csv'):
        write_to_csv(All_data)
    elif arg[0].endswith('.json'):
        dum_to_json(raw_data)

main(sys.argv[1:])








