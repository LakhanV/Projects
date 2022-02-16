# Wool search app
import bs4

import pandas
from urllib.request import urlopen as ureq
import requests
from bs4 import BeautifulSoup as soup

#finding the last page of the webstie
html = requests.get("https://www.wollplatz.de/wolle?page=1")
page_no = soup(html.text)
lp = page_no.find('ul', class_='paging-ul')
pages = [li.text for li in lp.find_all('li')]
last_page =int(pages[-2])
last_page+=last_page
filename="products.txt"
f=open(filename,"w")
headers="product_name\tprice\tcompositions\tneedle_size \n\n"
f.write(headers)

#loop to navigate through all the pages
for tag in range(1,last_page):
    my_url='https://www.wollplatz.de/wolle?page='+str(tag)
    uclient = ureq(my_url)
    page_html = uclient.read()
    uclient.close()
    page_soup = soup(page_html,"html.parser")
    containers = page_soup.findAll("div",{"class":"productlistholder productlist25"})
    
    for container in containers:
#filtering the required Products from all the Products
        product_name= container.div["data-productname"]
   
        for name in('Drops Baby Merino Mix','DMC Natura XL' , 'Drops Safran' ,'Hahn Alpacca Speciale' , 'Stylecraft Special double knit'):
            if name in product_name:
                price_container = container.findAll("span",{"class":"product-price-amount"})
                price=price_container[0].text
#using the refrence link to access the materials and needile size information
                for link in container.findAll('a'):
                        ref_url=link.get('href')
            
                uclient=ureq(ref_url)
                pro_page_html=uclient.read()
                uclient.close()
                pro_page_soup=soup(pro_page_html,"html.parser")
                spec_container = pro_page_soup.findAll("div",{"class":"innerspecsholder"})
                needle = spec_container[0].find(lambda t: t.text.strip()=='Nadelstärke')
                needle_size = needle.find_next('td')
                material = spec_container[0].find(lambda t: t.text.strip()=='Zusammenstellung')
                composition = material.find_next('td')
#writing the data in the text file
                print(product_name+"\t"+price+"\t"+composition.text+"\t"+needle_size.text+"\n")
                f.write(product_name+",\t"+price+",\t"+composition.text+",\t"+needle_size.text +"\n\n")
            
f.close()
