from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains
from time import sleep
import pandas as pd
from openpyxl import Workbook
import csv

# create object for chrome options
chrome_options = Options()
base_url = 'https://dubailand.gov.ae/en/eservices/brokers-overview/approved-brokers'

chrome_options.add_argument('disable-notifications')
chrome_options.add_argument('--disable-infobars')
chrome_options.add_argument('start-maximized')
chrome_options.add_argument(
    'user-data-dir=C:\\Users\\username\\AppData\\Local\\Google\\Chrome\\User Data\\Default')
# To disable the message, "Chrome is being controlled by automated test software"
chrome_options.add_argument("disable-infobars")
# Pass the argument 1 to allow and 2 to block
chrome_options.add_experimental_option("prefs", {
    "profile.default_content_setting_values.notifications": 2
})
# invoke the webdriver
browser = webdriver.Chrome(executable_path=r'C:\Program Files (x86)\chromedriver.exe',
                           options=chrome_options)
browser.get(base_url)
delay = 5  # secods

csv_file = open('sheet.csv', 'w', newline='')
csv_writer = csv.writer(csv_file)
csv_writer.writerow(["first_name", "family_name", "phone_type", "phone number",  "email",
                     "email_type", "occupation", "occupation_type", "broker_number", "Image Url"])
for i in range(10000):
    try:
        WebDriverWait(browser, delay)
        print("Page is ready")
        sleep(8)
        html = browser.execute_script(
            "return document.getElementsByTagName('html')[0].innerHTML")

        soup = BeautifulSoup(html, "html.parser")
        items = soup.find_all('div', class_="col-md-12 card-detail")
        for item in items:
            # Exctracting The First and The Last Name:
            name = item.find('div', class_="custom-card-title").text
            name = name.split(' ')
            first_name = name[0]

            del name[0]
            family_name = ''
            for elements in name:
                family_name += str(elements) + " "

            # Exctracting The Phone and The Mobile Name:
            phone = item.find('li', class_="call").text
            print(phone)

            # Exctracting the email:
            email = item.find('li', class_="email").text
            print(email)

            image_url = item.find('img', class_="img-fluid")['src']
            print(image_url)

            # Exctracting the Occupation:
            infos = item.find('ul', class_="list-style font-size14")
            info = infos.find_all('li')
            occupation = info[1].text

            # Exctracting the Broker Number:
            infos = item.find('ul', class_="list-style font-size14")
            info = infos.find_all('li')
            broker_number = info[0].text
            print(broker_number)

            occupation_type = "Real Estate Agency"
            phone_type = "Mobile"
            email_type = "Business"

            csv_writer.writerow([first_name, family_name, phone_type, phone,  email,
                                 email_type, occupation, occupation_type, broker_number, image_url])

        load_more = browser.find_element_by_id("load-more")
        actions = ActionChains(browser)
        actions.click(load_more)
        actions.perform()

    except TimeoutException:
        print("Loading took too much time!-Try again")
print("File is Saved")


# close the automated browser
browser.close()
