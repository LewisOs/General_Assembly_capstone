# How much is that doggy in my (browser) window?

This is the capstone project from my Data Science Immersive course at General Assembly, which I completed on 03/02/23. Whilst the project itself is complete, this README file is a **work in progress**. Please bare with me whilst I update it.

## TL;DR

8-10 bullet points summarising the project.

## Intro

During the COVID-19 pandemic, British people sought cures for their lockdown blues in a variety of places. Some people found solace in the simple things, like tending their garden (in Animal Crossing), pretending to enjoy the interminably drawn-out process of making sourdough bread, or adhering to a strict regime of near-daily quizzes held over Zoom with friends and family (anyone fancy another round of Geoguessr?). Others took even more extreme measures, like changing out of their pyjamas or showering more than once a fortnight. Whilst the jury is still out on the efficacy of these activities, there was one way to combat the dolrums that we could all agree just worked - spending time with our pets (and by ‘all’, I of course mean 87% of respondents to a cross-sectional online survey of UK residents over the age of 18 conducted by the University of York between April and June 2020). 

In fact, spending time with pets was such a popular solution to our collective cabin-fever that the demand for pets increased massively during the pandemic. From the week of the 23rd of March 2020, when the first UK COVID-19 lockdown came into effect, to the week of the 10th of May 2020, when the conditional plan for lifting the lockdown was announced, Google searches from the UK for the term ‘puppies for sale’ almost tripled (see graph below). Pets sales website Pets4Homes reported that the average number of buyers competing for each pet on their platform had risen to 420 by May 2020 (see graph below). According to the Pet Food Manufacturers' Association, between the beginning of the pandemic and March 2021, a total of 3.2 million households in the UK acquired a pet, leading to national pet food shortages. This surge in demand was so great that Lee Gibson, then UK Commercial Director at Pets4Homes, described the pandemic as ‘what is likely to be the most significant demand increase for pets in modern history’.

[EMBED GOOGLE ANALYTICS “puppies for sale” CHART]
[ADD PETS4HOMES GRAPH]

As this historic growth in the demand for pets unfolded, supply dwindled. Pandemic restrictions disrupted essential aspects of the businesses of many British pet breeders. For instance, in-person viewings (which are recommended to ensure that the seller is genuine and to check that the pet is healthy and has been raised in an appropriate environment) became  difficult or impossible due to social distancing rules and restrictions on travel. These disruptions were so significant that many breeders stopped advertising their litters altogether. 

Naturally, as demand rose and supply fell, there was only one possible outcome. The prices of pets in the UK skyrocketed. For instance, Pets4Homes reported that between March and September of 2019, the average asking price for a puppy on their platform was £888. For the same period in 2020, this had more than doubled - reaching an average of £1883. For certain trendy breeds, the price hikes were even more extreme- the prices of Cavapoo puppies almost tripled from ~£1000 to £2800+.

[ADD GRAPH FROM BBC ARTICLE] 

More recently, the UK’s ongoing cost of living crisis has made keeping or buying a pet financially prohibitive for many households. For instance, a 2021 survey by the Blue Cross and the Edinburgh University found that 68% of respondents were concerned about the impact of the rising cost of living on their ability to care for their pets. This has sadly resulted in unprecedented numbers of people attempting to rehome their pets via animal shelters, or simply abandoning them outright. Between the 1st of January and 31st of October, the Dogs Trust received 42,000 inquiries from dog owners about rehoming, 48% more than in the same period the year prior. Between January and July 2021, the RSPCA received 18,375  animal abandonment reports, for the same period in 2022 this figure had increased by ~25%, to 22,908 

The extreme fluctuations seen in the UK’s demand for and supply of pets over the last few years has caused significant turbulence in pet prices. As such, it has become difficult for both sellers and buyers to know what is a fair price (or at least the market rate) for pets of different species, breeds, ages etc. Given this, I decided to model the current pet sales market. In particular, I wanted to see whether I would be able to build a model that could accurately predict the prices of individual pets based on their attributes. I decided to treat this as a regression problem (rather than binning pets in price bands to classify them) and to set a target of attaining an R2 of 0.80. Such a model would: 1) allow to buyers to check whether an advert is using current market prices, or older lockdown-inflated prices 2) allow buyers to determine which types of animals are within their budget, 3) allow sellers to figure out how best to competitively price their animals and 4) provide 3rd party platforms like Pets4Homes with a tool to inform their users about how listed prices compare to market rates (see Autotrader’s used car search as an example of this).

## Data collection and wrangling

In order to base my model on up-to-date data about pet prices, I decided I would build a data set by scraping information about pet adverts off of an online pet sales platform. Specifically; Pets4Homes.co.uk. I selected this website as it is the largest pet sales websites in the UK. It also asks sellers to list a large amount of information about the pets they are selling, which I would be able to scrape and feed into my model. My plan was to scrape all listings for all animal types on Pets4Homes during the week of the 9th to the 16th of January, 2023. Due to the structure of the Pets4Homes website, it was necessary to do this in two stages. First, I would need to collect the individual listing URLs from the search pages. Second, I would need to collect the data about each listing from those URLs.

Stage one involved making a series of searches which would return all listings for each of the following animal types (i.e. one search per type): dogs, cats, horses, birds, rodents, rabbits, reptiles, invertebrates and fish. Each search returned up to 500 pages of results, with each page containing around twenty listings, including a brief overview of each listing. Within the HTML for this brief overview, the URL for the full listing could be found. To automate this search and data collection, I used Selenium web driver. This returned the full HTML for each page of search results, which I could then transform into a Beautiful Soup object in order to extract the URLs. I ran this scrape until I had collected all listings on the website (with the exception of animals listed as livestock or poultry - as I reasoned these were not pets). This came to over 20,000 unique URLs. Whilst using Selenium for such a simple scrape may seem like overkill, this was not without good reason. I had initially tried to use Python's Requests library for this process, but found that Pets4Homes immediately blocked these requests, even when I tried methods such as using a VPN or manually changing my headers. 

The second stage of the scrape involved going to each of the URLs and extracting the relevant information about the listing. As before, I did this with a combination of Selenium and Beautiful Soup. However, I soon encountered a problem. I had written functions using Beautiful Soup to extract information about each pet listing from it’s HTML, using HTML class names to find where that information was represented. This seemed to work well at first, but then my scraper stopped returning any information. Unable to find any obvious errors in my code, I doubled checked Pets4Home’s live HTML and discovered that the class names had changed. In fact, it appear that they were dynamically updating them multiple times a day. This had not become apparent during the first stage of my scrape, as it was relatively quick. By comparison, the second stage involved around 20x more web pages and was done in stages.

As I was working to a tight deadline and was initially unable to figure out a way to extract the information I needed without referencing the class names, I had to modify my plans. Rather than nativgating to each URL and extracting only the information I needed, I simply downloaded the full HTML for every page. My reasoning was that if I was unable to extract the information with the class names changing, then at least I could create a static copy of the HTML to work on. This would allow me to rerun my original function over the downloaded HTML and, each time it began failing to extract information from a page, manually check what the class names had updated to so I could update the function to search for those. However, given that the class names were updating multiple times a day and that it took several days to scrape every listing, I was very keen to avoid this. 

Fortunately, I was able to find a solution to this problem. Rather than using Beautiful Soup to search for class names in the HTML, I used it to search for the unchanging attributes related to each class. For this to work, it was necessary to find attributes which were unique to each class. In some cases this was simple. For instance, in the head of the HTML, only the class corresponding to the webpage’s title had a text attribute containing the word ‘title’. Other classes required a more circuitous approach. For instance, sellers on Pets4Homes can be verified by one of four methods (phone, email, Google and Facebook). I was able to determine which methods a particular seller was verified by checking the hex colour codes of the icons corresponding to each method (with green indicating verification and grey indicating the opposite). Using these techniques, I was able to rewrite my original function to extract almost all of the information I needed. The exceptions to this were two classes with no unique attributes which indicated the number of times a listing had been viewed and the number of times it had been liked by viewers. 

With this problem solved, I ran my 20,000+ pages of downloaded HTML through the updated extraction function and soon enough had a complete dataset 

## Data Cleaning

The uncleaned dataset consisted of 20211 rows and 39 features. The features were:

Target variable:
price - the price of 1 animal in the listing (some listings covered multiple animals, such as dog litters)
Seller/advert variables:
title - the title of the advert  
url - the advert URL, included for reference
seller_type - a categorical variable indicating whether the seller was a person or an organsiation
seller_name - the seller’s name
phone_verified - a binary variable indicating whether the seller is verified by phone
email_verified - a binary variable indicating whether the seller is verified by email
facebook_verified - a binary variable indicating whether the seller is verified by facebook     
google_verified - a binary variable indicating whether the seller is verified by google     
n_images - a continuous variable indicating the number of images included in the advert           
advert_id - the advert ID, included for reference       
advert_location - the location of the seller (all sellers were UK based)
advert_type - a categorical variable indicating the type of advert. Pets4Homes also allows sellers to list accessories and services for sale. As I was only looking at pets, this value was the same for all rows and the variable was dropped.
advertiser_type - a categorical variable providing similar information to seller_type, but at a more granular level (e.g. it includes levels such as ‘liscenced breeder’ and ‘charity’)
description - the seller supplied description of the listing (a text box from the webpage)
pet_available        
General pet variables:
category - a categorical variable specifying how the animal is classed by Pets4Homes’ search function. This is effectively species (e.g. dogs, cats, horses), but also includes some more general bins (reptiles, birds, fish).     
breed - a categorical variable specifying the breed of the particular pet (e.g. ‘English springer spaniel’).
pet_age - the pet’s age formatted as variations on ‘n days, n months, n years’   
pet_colour - the colour of the pet              
pet_sex - the sex of the pet 
Cat and dog specific variables:
health_checked - a binary variable indicating if the cat or dog has been health checked
microchipped - a binary variable indicating if the cat or dog has been microchipped
neutered - a binary variable indicating if the cat or dog has been neutered
vaccinated - a binary variable indicating if the cat or dog has been vaccinated          
worm_treated - a binary variable indicating if the cat or dog has been worm_treated  
pets_in_litter - two continuous variables formatted as variations on ‘n males / n females’
original_breeder - a binary variable indicating if the seller is the original breeder
Cat specific variable:
registered - a categorical variable indicating whether a cat is registered with any of 3 cat owners associations      
Dog specific variables:          
kc_registered - a binary variable indicating whether a dog is registered with the UK’s Kennel Club
Viewable_with_mother - a binary variable indicating whether a puppy is viewable with it’s mother
Horse specific variables     
birth_year - a continuous variable indicating a horse’s year of birth     
category_1 - a catergorical variable indicating what the horse’s primary use (e.g. dressage, show jumping, companionship etc.)
category_2 - a catergorical variable indicating what the horse’s secondary use with identical levels to category_1
gender - a categorical variable indicating both the horse’s sex and whether it was neutered, with 3 levels (stallion, gelding, mare)
Height - a continuous variable specifying the horse’s height, measured in hands                   
Origin - a categorical variable indicating the horse’s country of origin
level_jumping - a categorical variable indicating at what level the horse competed at show jumping
level_dressage - a categorical variable indicating at what level the horse competed at dressage


As you may be able to tell from the variables listed above, the dataset was quite messy and needed significant cleaning. Two of the most initially obvious problems were distinct columns representing similar information (e.g. the sex, neutered and gender columns, or the age and birth_year columns) and columns which only had values for certain animal types (e.g any of the dog, cat or horse specific columns) meaning the data set had a large number of NaNs. In addition to this, most columns contained data which was simply pulled straight from the HTML. As such, most columns were incorrectly formatted as strings. There were a small number of exceptions where I had created numerical variables based on information in the HTML (e.g. the number of photos included in the advert). In this section, I will review the steps that I took to address these issues and to prepare the data for EDA and predictive analysis.

The first cleaning step I took was to check for any duplicate rows in the data, of which I found 718. 

Converted price to a float
Dropped advert type
Almost half the seller names were unique. ⅓ of those that appeared more than once were organisations. Dropped seller name and replaced it with seller_n_adverts.
6653 unique locations, down to town level. Stripped these back to the most general location (e.g. London, Birmingham etc.). Ended with 1168 locations.
Seller_type and advertiser_type contained similar information, with the latter having more granular detail, but some NaNs. I used the former to fill the gaps in the latter and then dropped the former.
9 

## EDA


## Modelling


## Results


## Next steps



