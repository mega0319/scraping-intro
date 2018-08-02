# Intro To Scraping

## Dev Setup
0. Clone this repo: `git clone git@github.com:simplefractal/fractal-django-genie.git`;
1. Rename the cloned directory to your project name, create a new repo on GitHub with the same name, change the local git origin to point to the new repo and the push.
3. Create a virtualenv for Python 3.5: `mkvirtualenv scraping-intro -p /usr/bin/python3.5`;
4. Copy default postactivate: `cp contrib/proj_postactivate scraping-intro/bin/postactivate`;
5. Edit `scraping-intro/bin/postactivate` for you custom env config (e.g. update DATABASE_URL) and re-run the `workon` command;
6. Install the packages: `pip install -r dev-requirements.txt`;
7. Create postgres db: `createdb scraping_intro`;
8. Install redis ([click here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-redis) if you're using Ubuntu)
9. Migrations: `python project/manage.py migrate`


## Introduction

As the internet becomes a larger and more complete part of our lives the wealth of data that can be found online grows exponentially.
This leads to many opportunities where scraping the web can lead to more well informed stock market decisions, an immediate notification of
a price drop on certain items, or even exact identification of minor changes to a webpage. While our upcoming project may only need one of these use
cases learning scraping can certainly open new solutions to future problems you may encounter.

## Types of Scraping

Wikipedia lists the different types of scraping as:

  **Human copy-and-paste**
  Sometimes even the best web-scraping technology cannot replace a human’s manual examination and copy-and-paste, and sometimes this may be the only workable
  solution when the websites for scraping explicitly set up barriers to prevent machine automation.

  **Text pattern matching**
  A simple yet powerful approach to extract information from web pages can be based on the UNIX grep command or regular expression-matching facilities of programming languages (for instance Perl or Python).

  **HTTP programming**
  Static and dynamic web pages can be retrieved by posting HTTP requests to the remote web server using socket programming.

  **HTML parsing**
  Many websites have large collections of pages generated dynamically from an underlying structured source like a database.
  Data of the same category are typically encoded into similar pages by a common script or template.
  In data mining, a program that detects such templates in a particular information source, extracts its content and translates it into a relational form, is called a wrapper.
  Wrapper generation algorithms assume that input pages of a wrapper induction system conform to a common template and that they can be easily identified in terms of a URL common scheme.
  Moreover, some semi-structured data query languages, such as XQuery and the HTQL, can be used to parse HTML pages and to retrieve and transform page content.

  **DOM parsing**
  By embedding a full-fledged web browser, such as the Internet Explorer or the Mozilla browser control, programs can retrieve the dynamic content generated by client-side scripts.
  These browser controls also parse web pages into a DOM tree, based on which programs can retrieve parts of the pages.

  Additionally, many dynamic websites often expose their data in internal JavaScript variables.
  Therefore, using a JavaScript-enabled web browser it is often possible to extract required data simply by accessing variables stored, for example, on the Window object.

  **Vertical aggregation**
  There are several companies that have developed vertical specific harvesting platforms.
  These platforms create and monitor a multitude of “bots” for specific verticals with no "man in the loop" (no direct human involvement), and no work related to a specific target site.
  The preparation involves establishing the knowledge base for the entire vertical and then the platform creates the bots automatically.
  The platform's robustness is measured by the quality of the information it retrieves (usually number of fields)
  and its scalability (how quick it can scale up to hundreds or thousands of sites). This scalability is mostly used to target the Long Tail of sites that common aggregators find complicated or too labor-intensive to harvest content from.

  **Semantic annotation recognizing**
  The pages being scraped may embrace metadata or semantic markups and annotations, which can be used to locate specific data snippets.
  If the annotations are embedded in the pages, as Microformat does, this technique can be viewed as a special case of DOM parsing.
  In another case, the annotations, organized into a semantic layer, are stored and managed separately from the web pages, so the scrapers can retrieve data schema and instructions from this layer before scraping the pages.

  **Computer vision web-page analysis**
  There are efforts using machine learning and computer vision that attempt to identify and extract information from web pages by interpreting pages visually as a human being might.

  That's a pretty long list of types of scrapers to choose from, luckily we'll be focusing on three types of scraping for this tutorial.

## What We Will be Using

**API Requests**

The first, and arguably simplest, form of web scraping that we use involves directly querying the API that serves the web page the data it uses and parsing through that to get what we need.
These requests usually come in two main formats:

  1.) This type of request has the query params dictate how the response will be sent back.
      You can manually alter things like latitude, longitude, search radius, and max returned results.
      Changing these can reduce the number of requests you need to make or give you more accurate results to name a couple of outcomes.
      We can see an example of this type of request here - https://www.rei.com/rest/stores?retail=true&dist=7500&limit=1500&visible=true&lat=39.82832&long=-98.5795'
      Feel free to drop the above link directly into your address bar and try messing around with the query params. See what kind of different responses you get!
      The full scraper can be found in `project/src/engine/scrapers/rei_scraper.py`

  2.) The second type of request that falls into this category uses form data and headers in place of query params and requires a bit more work to implement.
      First, the network tab must be used to find a request made to the API you are trying to query.
      From there you can check grab the url, request headers, and form data.
      Lastly, you must use the `requests` library to structure your request in a way that will be accepted by the API.
      Sounds tedious, thankfully the Postman app has made this process a breeze. Check the screencast in the videos section to see an example of how this works.
      A full scraper can be found in `project/src/engine/scrapers/advanced_auto_parts_scraper.py`

**Parsing Content with BeautifulSoup**

The second type of scraping is done by loading either the entire HTML webpage and parsing it or requesting the pre-rendered dynamic template that then loads in the data using scripts.
Regardless of which way we are using this method we rely on the BeautifulSoup library to pull the data out of the response.

  1.) The first, and slightly trickier type of request to parse is one that uses scripts.
      The data is grabbed by using the `requests` library to load the response and then parse the information we need out of the script.
      An example of this can be seen in `project/src/engine/scrapers/hardees_scraper`
      https://maps.ckr.com/stores/search?brand=hardees&q=11104&brand_id=1

  2.) The second is where a static HTML page is loaded and we want to parse data from it.
      This functions similarly to the above example but is easier to implement due to the fact that we don't have to do as much string manipulation to get the data we want.
      Rather than looking for script tags we convert the response to a BeautifulSoup object which then allows us to query it for specific elements.
      An example of this can be seen in `project/src/engine/scrapers/dq_scraper`

  Notes: You've probably noticed that these two scrapers are not that different from each other in their implementation.
         This can be misleading because the processes for extracting the data we need are quite distinct.
         Check the respective scrapers for more in-depth comments on what is happening where and why.

**Selenium**

The award for trickiest and most work intensive method of scraping we use has to be awarded to Selenium.
This method uses an actual browser to load the webpage which we can then query for specific elements which we can then use to get
valuable information such as the text content or `href`.

Some reasons behind Selenium's learning curve:
  1.) Selenium requires a more in-depth set up than the previous two methods, depending on whether you're using Chrome or Firefox you will either need to have both the path
      to your Chrome bin file and Chromedriver(more information here: http://chromedriver.chromium.org/) or the Firefox app itself.
  2.) The use of a browser opens us up to many more possible failure types than the simple API request and response parsing method.
      Some frequent errors include not being able to locate the element you're looking for, a website taking too long to load and causing your request to timeout, and
      a modal on the page causing the page to become non-interactive until the modal is closed.
  3.) All of the above errors will require some logic coded into your scraper to handle and can cause a large increase in dev time.

An example of this can be found in `project/src/engine/scrapers/tj_maxx_scraper`

## When to Use What

In my experience it is always best to time box ten to twenty minutes before implementing a scraper to test out the website.
What this means is that you should manually go through the steps you wish to have your scraper emulate. Not only will this give you
a better understanding of what it is you need to do but you can also monitor the network tab of the dev tools to see if there are any usable API requests.

The above list of scrapers is in descending order in regards to ease of implementation so if you can find a way to scrape the information you need with
a simple API request you'll save yourself over an hour of dev time at the very least.
The second thing you'll want to look for is a static webpage because we can pass this as an argument to BeautifulSoup and requires much less work to implement in comparison to Selenium.
Lastly, and if all of the above fail, you'll be required to use Selenium as your scraper. Check the videos section for an examples on how this all works.

## Videos

**API Requests**

### Query Param API request

[Trawling for an API endpoint](https://drive.google.com/file/d/1GEvknaWg2nsI0yznuZDCpbZM_EYWWea1/view)

Here you'll see the standard first step of research for any new scraper. Is there a request we can find that returns the exact data?
In this case we got lucky and there was, even though it took some clicking and digging to find. You'll also see that I check most JSON formatted responses because
that is typically what will contain what you're looking for.

### Form Data API Request

Let's say you've found the URL you want but there are no query params or when you copy and paste said link into your address bar it returns a 404 or nothing at all.
This could mean that this a request that queries the API based on form data. Check out the screencast below for an easy way to handle these types of URLs.

[Sending Headers and Form Data with Postman Interceptor](https://drive.google.com/file/d/1Fic6iyeTCpqLpiRbBXzBQQeOjbra2YTm/view)
[Postman Interceptor Chrome Extension](https://chrome.google.com/webstore/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo?hl=en)

**Parsing Content with BeautifulSoup**

### Script Parsing

If the above method has yielded no viable results the second step is to look for the information you're looking for in either pure HTML format or loaded
by a script.
A good way to quickly check if what you're looking for is returned in those formats is through the CMD + F function that lets you search the content of
responses in Chrome's Dev Tools.

[Searching for Parsable Scripts](https://drive.google.com/file/d/18V-JBmu9QrCGT7oxumw8kbg4TaXUM-4i/view)

**Selenium**

Selenium will be your fallback when all the previous options are exhausted.
In the following video it will help to have the `tj_maxx_scraper` file open as there are comments in the code explaining what's happening in the screencast.

[Finding the Appropriate HTML Elements for Selenium](https://drive.google.com/file/d/1agiRbk16oQYUqAtYTBWa25AKkmI_a-wQ/view)

## Docs

Here are links to documentation for a few of the libraries mentioned above.

[Python Requests Library](http://docs.python-requests.org/en/master/)

[Python xmltodict Library](https://github.com/martinblech/xmltodict) - For when responses are in the XML format rather than JSON

[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

[Selenium](http://selenium-python.readthedocs.io/)

## Conclusion

This is by no means a comprehensive list on scraping but hopefully it is a sufficient introduction to make your first foray into
this wonderful world a little less fraught with peril. Feel free to Slack me with any questions, corrections, concerns, or just suggestions for improvement.
