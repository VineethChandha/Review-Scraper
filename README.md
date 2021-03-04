# Review-Scraper(WEB-Scraping_of_Reviews)
## 1)Introduction
Web scraping is a technique using which the webpages from the internet are fetched and parsed to understand and extract specific information similar to a human being. Web scrapping consists of two parts:
* Web Crawling: Accessing the webpages over the internet and pulling data from them.
* HTML Parsing: Parsing the HTML content of the webpages obtained through web crawling and then extracting specific information from it.

Hence, web scrappers are applications/bots, which automatically send requests to websites and then extract the desired information from the website output.

Let’s take an example: 
how do we buy a phone online?
1.	We first look for a phone with good reviews
2.	We see on which website it’s available at the lowest price
3.	We check whether it’s  delivered in our area or not
4.	If everything looks good, then we buy the phone.

What if there is a computer program that can do all of these for us? That’s what web scrappers necessarily do. They try to understand the webpage content as a human would do.

* For example, if we open filpkart.com and search for ‘iPhone’, the search result will be as follows:


![image](https://user-images.githubusercontent.com/69765021/109957734-5e970580-7d0b-11eb-8ffc-38db4b908333.png)

* Then if we click on a product link, it will take us to to the following page:


![image](https://user-images.githubusercontent.com/69765021/109957908-9736df00-7d0b-11eb-90f2-78afff74f5ad.png)

* If we scroll down on this page, we’ll get to see the comments posted by the customers:


![image](https://user-images.githubusercontent.com/69765021/109957998-b9306180-7d0b-11eb-8a31-299f91fb50e6.png)

Our end goal is to build a web scraper that collects the reviews of a product from the internet.

## 2)Prerequisites:

*	Python installed.
*	A Python IDE (Integrated Development Environment): like [PyCharm](https://www.jetbrains.com/pycharm/download/#section=windows), Spyder, or any other IDE of choice.
*	Flask Installed. (A simple command: pip install flask)
*	[MongoDB](https://www.mongodb.com/try/download/community) installed.
*	Basic understanding of Python and HTML.

## 3)Application Architecture:

![image](https://user-images.githubusercontent.com/69765021/109958718-89ce2480-7d0c-11eb-9bdf-7013c43f5031.png)

## 4)Python Implementation
Note: I have used PyCharm as an IDE for this documentation
1.	Let’s create a folder called ‘reviewScrapper’ on our local machines.
2.	Inside that folder, let’s create two more folders called ‘static’ and ‘templates’ to hold the code for the UI of our application. Inside ‘static’, let’s create a folder called ‘css’ for keeping the stylesheets for our UI.
3.	Let’s create a file called ‘flask_app.py’ inside the ‘reviewScrapper’ folder. 
4.	Inside the folder ‘css’, create the files: ‘main.css’ and ‘style.css’. The files are attached here for reference. 
  
5.	Inside the folder ‘templates’, create three HTML files called: ‘base.html’,’index.html’, and ‘results.html’. The files are attached here for reference. 
   
* base.html: It acts as the common building block for the other two HTML pages.
*	index.html:  Home page of our application.
*	results.html: Page to show the reviews for the searched keyword.
6.	Now, let’s understand the flow:
a)	When the application starts, the user sees the page called ‘index.html’.
b)	The user enters the search keyword into the search box and presses the submit button.
c)	The application now searches for reviews and shows the result on the ‘results.html’ page.
7.	Understanding flask_app.py.
 
a)	Import the necessary libraries:.

* from flask import Flask, render_template, request,jsonify
* from flask_cors import CORS,cross_origin
* import requests
* from bs4 import BeautifulSoup as bs
* from urllib.request import urlopen as uReq
* import pymongo
 
b)	Initialize the flask app
* app = Flask(__name__)  # initialising the flask app with the name 'app'

c)	Creating the routes to redirect the control inside the application itself. Based on the route path, the control gets transferred inside the application.
* @app.route('/',methods=['POST','GET']) # route with allowed methods as POST and GET

d)	Now let’s understand the ‘index()’ function.

i.	If the HTTP request method is POST(which is defined in index.html at form submit action), then first check if the records for the searched keyword is already present in the database or not. If present, show that to the user.
* dbConn = pymongo.MongoClient("mongodb://localhost:27017/")  # opening a connection to Mongo
* db = dbConn['crawlerDB'] # connecting to the database called crawlerDB
* reviews = db[searchString].find({}) # searching the collection with the name same as the keyword
* if reviews.count() > 0: # if there is a collection with searched keyword and it has records in it
* return render_template('results.html',reviews=reviews) # show the results to user

ii.	If the searched keyword doesn’t have a database entry, then the application tries to fetch the details from the internet, as shown below:
* flipkart_url = "https://www.flipkart.com/search?q=" + searchString # preparing the URL to search the product on Flipkart
* uClient = uReq(flipkart_url) # requesting the webpage from the internet
* flipkartPage = uClient.read() # reading the webpage
* uClient.close() # closing the connection to the web server
* flipkart_html = bs(flipkartPage, "html.parser") # parsing the webpage as HTML


iii.	Once we have the entire HTML page, we try to get the product URL and then jump to the product page. It is similar to redirecting to the following page:
The equivalent Python code is:
* productLink = "https://www.flipkart.com" + box.div.div.div.a['href'] # extracting the actual product link
* prodRes = requests.get(productLink) # getting the product page from server
* prod_html = bs(prodRes.text, "html.parser") # parsing the product page as HTML

![image](https://user-images.githubusercontent.com/69765021/109957908-9736df00-7d0b-11eb-90f2-78afff74f5ad.png)


iv.	On the product page, we need to find which HTML section contains the customer comments. Let’s do inspect element(ctrl+shift+i) on the page first to open the element-wise view of the HTML page. There we find the tag which corresponds to the customer comments as shown below:

![image](https://user-images.githubusercontent.com/69765021/109960533-e7fc0700-7d0e-11eb-94e8-d25c74b429dc.png)

v.	Once we have the list of all the comments, we now shall extract the customer name, the rating, comment heading, and the customer comment from the tag. If you notice, the parsing is done using the try-except blocks. It is done to handle the exception cases. If there is an exception in parsing the tag, we’ll insert a default string in that place.

vi.	Once we have the details, we’ll insert them into MongoDB. After that, we’ll return the ‘results.html’ page as the response to the user containing all the reviews.  The python code for that is:
* mydict = {"Product": searchString, "Name": name, "Rating": rating, "CommentHead": commentHead,"Comment": custComment} # saving that detail to a dictionary
* x = table.insert_one(mydict) #insertig the dictionary containing the rview comments to the collection
* reviews.append(mydict) #  appending the comments to the review list
* return render_template('results.html', reviews=reviews) # showing the review to the user

e)	After this, we’ll just run our python app on our local system, and it’ll start scraping for reviews as shown below:	

IDE:

![image](https://user-images.githubusercontent.com/69765021/109961894-8fc60480-7d10-11eb-8524-1bedd96853e3.png)

Home Page(we get this once we click on the link present in run):

![image](https://user-images.githubusercontent.com/69765021/109962063-c6038400-7d10-11eb-8a8e-65ac9563daa2.png)

Search Result:

![image](https://user-images.githubusercontent.com/69765021/109962264-fcd99a00-7d10-11eb-8f30-488d403e22ee.png)


## Thank you :pray:
## Happy coding :satisfied:



