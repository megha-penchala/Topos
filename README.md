# Web Scrapping
### import the libraries used to query a website

import requests
import re
import csv

### specify the url
website_url= requests.get('https://en.wikipedia.org/wiki/List_of_United_States_cities_by_population').text

### import the Beautiful soup functions to parse the data returned from the website
from bs4 import BeautifulSoup

### Parse the html in the 'website_url' variable, and store it in Beautiful Soup format
soup = BeautifulSoup(website_url,'lxml')

### extracting content between < table > tag
my_table = soup.find('table', class_='wikitable sortable')
my_table

### Finding all the links of column "City" of top 100 cities in U.S.
links = []
 
for row in my_table.find_all('tr')[1:101]:
        for td in row.find_all('td')[1:2]:
            for link in td.find_all('a', attrs={'href': re.compile("^/wiki")}):
                links.append(link.get('href'))
                
print(links)

### extracting Rank, City, State, 2018 estimate, 2010 census from wikipedia 
### extracting time zone from individual city pages 

csv_file = open('us_topos.csv','w')
csv_writer = csv.writer(csv_file)
csv_writer.writerow(['Rank','City','State','2018 estimate','2010 census','Timezone'])

index = 0
for tr in my_table.findAll('tr')[1:101]:
    stack = []
    for td in tr.findAll('td')[0:5]:
        stack.append(td.text.strip())
        
        url="https://en.wikipedia.org"+links[index]
        city_url = requests.get(url).text
        soup = BeautifulSoup(city_url,'lxml')
        timezone = soup.find('a', href=True, text='Time zone').next_element.next_element.text.strip()
    stack.append(timezone)        
    index = index + 1
    csv_writer.writerow(stack)
csv_file.close()
