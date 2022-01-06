# Import libraries
import pyrebase
import requests
from bs4 import BeautifulSoup
from datetime import datetime

# Set the configuration for your app
# Todo : Replace with your project's config object
config = {
    "apiKey": "AIzaSyC9xLrOv9Bs80v4j-8EkaBfOP4UwUUb5jQ",
    "authDomain": "power-meter-5a83d.firebaseapp.com",
    "databaseURL": "https://power-meter-5a83d-default-rtdb.asia-southeast1.firebasedatabase.app",
    "projectId": "power-meter-5a83d",
    "storageBucket": "power-meter-5a83d.appspot.com",
    "messagingSenderId": "122976377150",
    "appId": "1:122976377150:web:17cb046bce5f1f5964f46f",
    "measurementId": "G-0X4FV8QKBF"
}
firebase = pyrebase.initialize_app(config)

# Get a reference to the database service
db = firebase.database()


# Define scraper function
def scraper():
    query = "covid"
    query_separator = "%20".join(query.split())

    start = "https://news.google.com/search?q="
    end = "&hl=en-IN&gl=IN&ceid=IN%3Aen"
    URL = start + query_separator + end

    r = requests.get(URL)
    soup = BeautifulSoup(r.content, 'html.parser')
    headlines = soup.find_all('a', class_='DY5T1d RZIKme')
    all_headlines = {}
    count = 0
    for headline in headlines:
        if count < 9:
            key = "0"+str(count)
        else:
            key = str(count)
        all_headlines[key] = headline.text
        count += 1

    now = datetime.now()
    current_date = now.strftime("%d")
    current_month = now.strftime("%B")
    current_year = now.strftime("%Y")

    db.child("CovidHeadlines").child(current_year).child(
        current_month).child(current_date).set(all_headlines)


# Define scheduler function
def scheduler():
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    # Trigger scraper function everyday on 12 AM
    if current_time == "00:00:00":   
        scraper()

scheduler()
