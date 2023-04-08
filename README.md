
Twitter-scraping What is Twitter Scraping?

Scraping is a technique to get information from Social Network sites. Scraping Twitter can yield many insights into sentiments, opinions and social media trends. Analysing tweets, shares, likes, URLs and interests is a powerful way to derive insight into public conversations. It is legal to scrape Twitter or any other SNS(Social Networking Sites) to extract publicly available information, but you should be aware that the data extracted might contain personal data.

How to Scrape the Twitter Data?

Scraping can be done with the help of many opensource libraries like

Tweepy
Twint
Snscrape
Getoldtweets3
Step 1

Importing the libraries.

As I have already mentioned above the list of libraries/modules needed for the project. First we have to import all those libraries. Before that check if the libraries are already installed or not by using the below piece of code.

!pip install ["Name of the library"]
If the libraries are already installed then we have to import those into our script by mentioning the below codes.

    import re
    import csv
    from getpass import getpass
    from time import sleep
    from selenium.webdriver.common.keys import Keys
    from selenium.common.exceptions import NoSuchElementException
    from msedge.selenium_tools import Edge, EdgeOptions
Step 2

    def get_tweet_data(card):
"""Extract data from tweet card"""
username = card.find_element_by_xpath('.//span').text
try:
    handle = card.find_element_by_xpath('.//span[contains(text(), "@")]').text
except NoSuchElementException:
    return

try:
    postdate = card.find_element_by_xpath('.//time').get_attribute('datetime')
except NoSuchElementException:
    return

comment = card.find_element_by_xpath('.//div[2]/div[2]/div[1]').text
responding = card.find_element_by_xpath('.//div[2]/div[2]/div[2]').text
text = comment + responding
reply_cnt = card.find_element_by_xpath('.//div[@data-testid="reply"]').text
retweet_cnt = card.find_element_by_xpath('.//div[@data-testid="retweet"]').text
like_cnt = card.find_element_by_xpath('.//div[@data-testid="like"]').text

# get a string of all emojis contained in the tweet
"""Emojis are stored as images... so I convert the filename, which is stored as unicode, into 
the emoji character."""
emoji_tags = card.find_elements_by_xpath('.//img[contains(@src, "emoji")]')
emoji_list = []
for tag in emoji_tags:
    filename = tag.get_attribute('src')
    try:
        emoji = chr(int(re.search(r'svg\/([a-z0-9]+)\.svg', filename).group(1), base=16))
    except AttributeError:
        continue
    if emoji:
        emoji_list.append(emoji)
emojis = ' '.join(emoji_list)

tweet = (username, handle, postdate, text, emojis, reply_cnt, retweet_cnt, like_cnt)
return tweet
Step 3

create instance of web driver

    options = EdgeOptions()
    options.use_chromium = True
    driver = Edge(options=options)
Step 4

application variables

 user = input('username: ')
 my_password = getpass('Password: ')
 search_term = input('search term: ')
Step 5

navigate the login screen

   driver.get('https://www.twitter.com/login')
   driver.maximize_window()
   sleep(5)
   username = driver.find_element_by_xpath('//input[@name="text"]')
   username.send_keys(user)
   username.send_keys(Keys.RETURN)
   sleep(3)
Step 6

 password = driver.find_element_by_xpath('//input[@name="password"]')
 password.send_keys(my_password)
 password.send_keys(Keys.RETURN)
 sleep(3)
Step 7

find search input and search for term

 search_input = driver.find_element_by_xpath('//input[@aria-label="Search query"]')
 search_input.send_keys(search_term)
 search_input.send_keys(Keys.RETURN)
 sleep(1)
Step 8

navigate to historical 'latest' tab

 driver.find_element_by_link_text('Latest').click()
Step 9

get all tweets on the page

data = []
tweet_ids = set()
 last_position = driver.execute_script("return window.pageYOffset;")
   scrolling = True

 while scrolling:
   page_cards = driver.find_elements_by_xpath('//div[@data-testid="tweet"]')
  for card in page_cards[-15:]:
    tweet = get_tweet_data(card)
    if tweet:
        tweet_id = ''.join(tweet)
        if tweet_id not in tweet_ids:
            tweet_ids.add(tweet_id)
            data.append(tweet)
        
scroll_attempt = 0
while True:
    # check scroll position
    driver.execute_script('window.scrollTo(0, document.body.scrollHeight);')
    sleep(2)
    curr_position = driver.execute_script("return window.pageYOffset;")
    if last_position == curr_position:
        scroll_attempt += 1
        
        # end of scroll region
        if scroll_attempt >= 3:
            scrolling = False
            break
        else:
            sleep(2) # attempt another scroll
    else:
        last_position = curr_position
        break
Step 10

 with open('Elon Musk_tweets.csv', 'w', newline='', encoding='utf-8') as f:
header = ['UserName', 'Handle', 'Timestamp', 'Text', 'Emojis', 'Comments', 'Likes', 'Retweets']
writer = csv.writer(f)
writer.writerow(header)
writer.writerows(data)
Step 11

close the web driver

    driver.close()
    
 Workflow:
 linkedin demo video link:https://www.linkedin.com/feed/update/urn:li:activity:7050370635264380928?updateEntityUrn=urn%3Ali%3Afs_updateV2%3A%28urn%3Ali%3Aactivity%3A7050370635264380928%2CFEED_DETAIL%2CEMPTY%2CDEFAULT%2Cfalse%29&lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base%3Bu3NOow5AT2uLjvuoY7rF1g%3D%3D
