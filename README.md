# How much is that doggy in my (browser) window?

## Table of contents
* [TL;DR](#TL;DR)
* [Introduction](#Introduction)
  * [Background](#Background)
  * [Goals](#Goals)
* [Data collection and wrangling](#Data-collection-and-wrangling)
* [Data cleaning](#Data-cleaning)
  * [Seller & listing features](#Seller-&-listing-features)
  * [Pet features](#Pet-features)
  * [Cleaned data dictionary](#Cleaned-data-dictionary)
* [EDA](#EDA)
  * [Continuous variables](#Continuous-variables)
  * [Binary variables](#Binary-variables)
  * [Categorical variables](#Categorical-variables)
  * [Text variables](#Text-variables)
* [Modelling](#Modelling)
  * [Feature engineering](#Feature-engineering)
  * [Splitting and scaling the data](#Splitting-and-scaling-the-data)
  * [Building models](#Building-models)
  * [Initial models](#Initial-models)
  * [Parameter tuning](#Parameter-tuning)
  * [Comparing models](#Comparing-models)
* [Conclusions and next steps](#Conclusions-and-next-steps)
  

## TL;DR <a name="TL;DR"></a>

* This is my capstone project for General Assembly's 2022/23 Data Science Intensive Course. It is an end-to-end solo machine-learning project which I completed over a period of 5 weeks.
* Following the pandemic and UK's cost of living crisis, prices in the UK's pet sales market have been extremely turbulent. This project aims to use information from online pet sales listings to model and predict the prices of pets.
* My specific goal for the project was to build a regression model which could predict prices with a high degree of accuracy (defined as an R2 score of 0.80), which could be used by buyers and sellers to understand the current market rate for a wide variety of pet types.
* To collect the data, I scraped over 20,000 user-generated listings from the online pet sales platform Pets4Homes.co.uk using Selenium. I did this in two stages; 1) an inital scrape of the site's search pages to extract the individual listings' URLs and 2) a second round scraping each URL to collect information about each listing, seller and pet.
* A data wrangling challenge arose when I discovered that the Pets4Homes website automatically changes it's HTML class names at regular intervals.   I overcame this by using BeautifulSoup to extract information from the HTML based on the classes' unchanging attributes (e.g. the colour hex codes of buttons).
* The uncleaned dataset consisted of 20211 rows and 39 features. Cleaning the dataset was a combination of some commonplace jobs (e.g. updating data types, handling NaNs, dropping irrelevant or useless columns) and some more nuanced tasks (e.g. combining or separating columns with overlapping semantics, or handling columns which only applied to particular animal types). After cleaning, the data set conisited of 19,271 rows and 35 columns.
* I carried out the EDA by splitting the variables into their statistical datatypes. After I removing outliers from the continuous variables, I was able to discover that most had very diffrent ranges depending on the animal type and also had non-linear relationships with the target variable. I also found promising correaltions between the target and the binary and categorical variables. I also analyzed the text variables using tf-idf, but I came to the conclusion that they didn't add much information which was not already captured by the other variables.
* I modelled the data using SciKit Learn. I began comparing the performance 9 regession algorithms using standard parameters and chose the 4 best for parameter tuning. These were ridge regression, LASSO regression, random forest regression and gradient-boosting regression. After tuning, the gradient-boosting regressor had the best performance, attaining a test set R2 of 0.65. All of the tuned models tended to underestimate the target.
* I was unable to build a model which could meet my original R2 target of 0.80+. As such, further improvements are needed before these models could be deployed for use by pet sellers and buyers. As next steps, I would like to experiment with collecting a larger dataset over a longer timespan, building individual models for each animal type and using other NLP tools to extract more useful information from the text variables.

## Introduction <a name="Introduction"></a>
#### Background <a name="Background"></a>

During the COVID-19 pandemic, British people sought cures for their lockdown blues in a variety of places. Some people found solace in the simple things, like tending their garden (in Animal Crossing), pretending to enjoy the interminably drawn-out process of making sourdough bread, or adhering to a strict regime of near-daily quizzes held over Zoom with friends and family (anyone fancy another round of Geoguessr?). Others took even more extreme measures, like changing out of their pyjamas or showering more than once a week. Whilst the jury is still out on the efficacy of these activities, there was one way to combat the doldrums that we could all agree just worked - spending time with our pets (and by ‘all’, I of course mean [87% of respondents to a cross-sectional online survey of UK residents over the age of 18 conducted by the University of York between April and June 2020](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0239397). 

In fact, spending time with pets was such a popular solution to our collective cabin fever that the demand for pets increased massively during the pandemic. From the week of the 23rd of March 2020, when the first UK COVID-19 lockdown came into effect, to the week of the 10th of May 2020, when the conditional plan for lifting the lockdown was announced, Google searches from the UK for the term ‘puppies for sale’ almost tripled (see graph below). Pets sales website Pets4Homes reported that the average number of buyers competing for each pet on their platform had risen to 420 by May 2020 (see graph below). According to the Pet Food Manufacturers' Association, between the beginning of the pandemic and March 2021, a total of 3.2 million households in the UK acquired a pet, leading to national pet food shortages. This surge in demand was so great that Lee Gibson, then UK Commercial Director at Pets4Homes, described the pandemic as ‘what is likely to be the most significant demand increase for pets in modern history’.

[EMBED GOOGLE ANALYTICS “puppies for sale” CHART]
[ADD PETS4HOMES GRAPH]

As this historic growth in the demand for pets unfolded, supply dwindled. Pandemic restrictions disrupted essential aspects of the businesses of many British pet breeders. For instance, in-person viewings (which are recommended to ensure that the seller is genuine and to check that the pet is healthy and has been raised in an appropriate environment) became difficult or impossible due to social distancing rules and restrictions on travel. These disruptions were so significant that many breeders stopped advertising their litters altogether. 

Naturally, as demand rose and supply fell, there was only one possible outcome. The prices of pets in the UK skyrocketed. For instance, Pets4Homes reported that between March and September of 2019, the average asking price for a puppy on their platform was £888. For the same period in 2020, this had more than doubled - reaching an average of £1883. For certain trendy breeds, the price hikes were even more extreme- the prices of Cavapoo puppies almost tripled from ~£1000 to £2800+.

[ADD GRAPH FROM BBC ARTICLE] 

More recently, the UK’s ongoing cost of living crisis has made keeping or buying a pet financially prohibitive for many households. For instance, a 2021 survey by the Blue Cross and Edinburgh University found that 68% of respondents were concerned about the impact of the rising cost of living on their ability to care for their pets. This has sadly resulted in unprecedented numbers of people attempting to rehome their pets via animal shelters, or simply abandoning them outright. Between the 1st of January and the 31st of October, the Dogs Trust received 42,000 inquiries from dog owners about rehoming, 48% more than in the same period the year prior. Between January and July 2021, the RSPCA received 18,375  animal abandonment reports, for the same period in 2022 this figure had increased by ~25%, to 22,908 

#### Goals <a name="Goals"></a>

The extreme fluctuations seen in the UK’s demand for and supply of pets over the last few years have caused significant turbulence in pet prices. As such, it has become difficult for both sellers and buyers to know what is a fair price (or at least the market rate) for pets of different species, breeds, ages etc. Given this, I decided to model the current pet sales market. In particular, I wanted to see whether I would be able to build a model that could accurately predict the prices of individual pets based on their attributes. I decided to treat this as a regression problem (rather than binning pets in price bands to classify them) and to set a target of attaining an R2 of 0.80. Such a model would: 1) allow buyers to check whether an advert reflects current market prices, or older lockdown-inflated prices 2) allow buyers to determine which types of animals are within their budget, 3) allow sellers to figure out how best to competitively price their animals and 4) provide 3rd party platforms like Pets4Homes with a tool to inform their users about how listed prices compare to market rates (e.g. see Autotrader’s used car search as an example of this).

## Data collection and wrangling <a name="Data-collection-and-wrangling"></a>

To base my model on up-to-date data about pet prices, I decided I would build a data set by scraping information about pet adverts off of an online pet sales platform, specifically, Pets4Homes.co.uk. I selected this website as it is the largest pet sales website in the UK. It also asks sellers to list a large amount of information about the pets they are selling, which I would be able to scrape and feed into my model. I planned to scrape all listings for all animal types on Pets4Homes during the week of the 9th to the 16th of January, 2023. Due to the structure of the Pets4Homes website, it was necessary to do this in two stages. First, I would need to collect the individual listing URLs from the search pages. Second, I would need to collect data about each listing from those URLs.

Stage one involved making a series of searches which would return all listings for each of the following animal types (i.e. one search per type): dogs, cats, horses, birds, rodents, rabbits, reptiles, invertebrates and fish. Each search returned up to 500 pages of results, with each page containing around twenty listings, including a brief overview of each listing. Within the HTML for this brief overview, the URL for the full listing could be found. To automate this search and data collection, I used Selenium web driver. This returned the full HTML for each page of search results, which I could then transform into a Beautiful Soup object to extract the URLs. I ran this scrape until I had collected all listings on the website (except for animals listed as livestock or poultry - as I reasoned these were not pets). This came to over 20,000 unique URLs. Whilst using Selenium for such a simple scrape may seem like overkill, this was not without good reason. I had initially tried to use Python's Requests library for this process but found that Pets4Homes immediately blocked these requests, even when I tried methods such as using a VPN or manually changing my headers. 

The second stage of the scrape involved going to each of the URLs and extracting the relevant information about the listing. As before, I did this with a combination of Selenium and Beautiful Soup. However, I soon encountered a problem. I had written functions using Beautiful Soup to extract information about each pet listing from its HTML, using HTML class names to find where that information was represented. This seemed to work well at first, but then my scraper stopped returning any information. Unable to find any obvious errors in my code, I double-checked Pets4Home’s live HTML and discovered that the class names had changed. In fact, it appears that they were dynamically updating them multiple times a day. This had not become apparent during the first stage of my scrape, as it was relatively quick. By comparison, the second stage involved around 20x more web pages and was done in stages.

As I was working to a tight deadline and was initially unable to figure out a way to extract the information I needed without referencing the class names, I had to modify my plans. Rather than navigating to each URL and extracting only the information I needed, I simply downloaded the full HTML for every page. I reasoned that if I was unable to extract the information with the class names changing, then at least I could create a static copy of the HTML to work on. This would allow me to rerun my original function over the downloaded HTML and, each time it began failing to extract information from a page, manually check what the class names had updated to so I could update the function to search for those. However, given that the class names were updated multiple times a day and that it took several days to scrape every listing, I was very keen to avoid this. 

Fortunately, I was able to find a solution to this problem. Rather than using Beautiful Soup to search for class names in the HTML, I used it to search for the unchanging attributes related to each class. For this to work, it was necessary to find attributes which were unique to each class. In some cases this was simple. For instance, in the head of the HTML, only the class corresponding to the web page’s title had a text attribute containing the word ‘title’. Other classes required a more circuitous approach. For instance, sellers on Pets4Homes can be verified by one of four methods (phone, email, Google and Facebook). I was able to determine which methods a particular seller was verified by checking the hex colour codes of the icons corresponding to each method (with green indicating verification and grey indicating the opposite). Using these techniques, I was able to rewrite my original function to extract almost all of the information I needed. The exceptions to this were two classes with no unique attributes which indicated the number of times a listing had been viewed and the number of times it had been liked by viewers. 

With this problem solved, I ran my 20,000+ pages of downloaded HTML through the updated extraction function and soon enough had a complete dataset 

## Data cleaning <a name="Data-cleaning"></a>

In this section, I will review the data-cleaning steps that I took to prepare the data for EDA and predictive analysis. The uncleaned dataset consisted of 20211 rows and 39 features. The features were:

Target variable:
price - the price of 1 animal in the listing (some listings covered multiple animals, such as dog litters)
Seller/advert variables:
title - the title of the advert  
url - the advert URL, included for reference
seller_type - a categorical variable indicating whether the seller was a person or an organisation
seller_name - the seller’s name
phone_verified - a binary variable indicating whether the seller is verified by phone
email_verified - a binary variable indicating whether the seller is verified by email
facebook_verified - a binary variable indicating whether the seller is verified by Facebook     
google_verified - a binary variable indicating whether the seller is verified by Google     
n_images - a continuous variable indicating the number of images included in the advert           
advert_id - the advert ID, included for reference       
advert_location - the location of the seller (all sellers were UK based)
advert_type - a categorical variable indicating the type of advert. Pets4Homes also allows sellers to list accessories and services for sale. As I was only looking at pets, this value was the same for all rows and the variable was dropped.
advertiser_type - a categorical variable providing similar information to seller_type, but at a more granular level (e.g. it includes levels such as ‘licensed breeder’ and ‘charity’)
description - the seller-supplied description of the listing (a text box from the webpage)
pet_available - the date when the pet is available for collection        
General pet variables:
category - a categorical variable specifying how the animal is classed by Pets4Homes’ search function. This effectively corresponds to species (e.g. dogs, cats, horses), but also includes some more general bins (reptiles, birds, fish).     
breed - a categorical variable specifying the breed of the particular pet (e.g. ‘English springer spaniel’).
pet_age - the pet’s age formatted as variations on ‘n days, n months, n years’   
pet_colour - the colour of the pet              
pet_sex - the sex of the pet 
Cat and dog-specific variables:
health_checked - a binary variable indicating if the cat or dog has been health checked
microchipped - a binary variable indicating if the cat or dog has been microchipped
neutered - a binary variable indicating if the cat or dog has been neutered
vaccinated - a binary variable indicating if the cat or dog has been vaccinated          
worm_treated - a binary variable indicating if the cat or dog has been worm_treated  
pets_in_litter - two continuous variables formatted as variations on ‘n males / n females’
original_breeder - a binary variable indicating if the seller is the original breeder
Cat-specific variable:
registered - a categorical variable indicating whether a cat is registered with any of 3 cat owners associations      
Dog-specific variables:          
kc_registered - a binary variable indicating whether a dog is registered with the UK’s Kennel Club
Viewable_with_mother - a binary variable indicating whether a puppy is viewable with its mother
Horse specific variables     
birth_year - a continuous variable indicating a horse’s year of birth     
category_1 - a categorical variable indicating the horse’s primary use (e.g. dressage, show jumping, companionship etc.)
category_2 - a categorical variable indicating the horse’s secondary use with identical levels to category_1
gender - a categorical variable indicating both the horse’s sex and whether it was neutered, with 3 levels (stallion, gelding, mare)
Height - a continuous variable specifying the horse’s height, measured in hands                   
Origin - a categorical variable indicating the horse’s country of origin
level_jumping - a categorical variable indicating at what level the horse competed at show jumping
level_dressage - a categorical variable indicating at what level the horse competed at dressage

As you may be able to tell from the variables listed above, the initial dataset was quite messy and needed significant cleaning. Two of the most obvious problems were distinct columns representing similar information (e.g. the sex, neutered and gender columns, or the age and birth_year columns) and columns which only had values for certain animal types (e.g any of the dog, cat or horse specific columns) meaning the data set had a large number of NaNs. In addition to this, most columns contained information which was simply pulled straight from the raw HTML. As such, many columns contained unprocessed numerical information (e.g. ‘7 males / 5 females’) or contained correctly typed numerical information which was incorrectly typed as a string. There were a small number of exceptions where I had created numerical variables based on information in the HTML (e.g. the number of photos included in the advert). 

I decided to begin with the low-hanging fruit. The first cleaning step I took was to check for any duplicate rows in the data, of which I found 718. These were all dropped. Next, I updated the types of any columns which had correctly formatted numerical information, which was wrongly typed (for instance, converting the price column from string type to float type). Finally, I dropped columns which I felt would not be useful. Specifically, these were advert_type (as all values were identical) and pet_available (as the column was about 50% NaNs, I wasn’t able to scrape an upload date to compare it to and, ultimately, I felt it wasn’t relevant to my original question).

#### Seller & listing features <a name="Seller-&-listing-features"></a>

With the easy cleaning tasks complete, I started on the columns relating to the advert and/or seller. Slightly less than half of the seller names were unique. Of those who appeared more than once, about 2/3rds were individuals and the remaining 3rd were organisations. Whilst the organisation’s names were given in full, the names of individuals were formatted as their first name followed by their surname initial  (e.g. ‘John S.’). Due to this, it was unclear whether two listings by individuals with the same seller_name value were listings by the same seller or simply sellers with similar names. Given this, I decided I would drop the seller_name column and replace it with a column which would specify the number of listings associated with a seller, called seller_n_listings. Given that I could not be if any individual had more than one listing, I set the value of seller_n_listings to 1 for all sellers, whereas for all organisations I used the number of times that they appeared in the seller_name column.

The location column included 6653 unique values, which were not consistently formatted. Whilst some were quite specific, with multiple levels of address, others were as general as “London”. Whilst I had initially wanted to geocode the location values, this inconsistency meant that some of the geocoding would be much more precise and accurate for some rows than for others. Given this, I instead decided to strip the locations down to the most general level specified (as every instance contained this level of information) and treat location as a categorical variable. After processing the location column contained 1168 unique locations.

As mentioned above, the seller_type and advertiser_type columns contained very similar information. seller_type only had two levels, indicating whether the seller was an organisation or an individual. advertiser_type had more granular detail, for instance, distinguishing between charities and for-profit organisations. Whereas seller_type had no missing information, advertiser_type had some NaNs. As such, I used the former to fill the gaps in the latter and then dropped the former. This completed the cleaning related to the sellers and adverts. Next was the cleaning relating to the pets being sold.

#### Pet features <a name="Pet-features"></a>

92 rows had NaNs for the breed column. On closer inspection, I discovered that all of these rows had completely unverified sellers and that the advert titles were extremely generic and many of them were duplicated (e.g. ‘Goldfish for sale’). I opened some of these adverts in my browser and found that for many of them, the animals in the photos were not even the same as the listed category. Given this, I assumed that these adverts were either erroneously generated, or possibly fishing scams. As such, I dropped these rows.

The next step was to clean and combine the pet_age and birth_year columns. For pet_age, I needed to extract numeric information from text which was formatted in variations on ‘x years, x months, x, weeks, x days’, where each entry may have included some, but not all of the levels. Because the strings were inconsistently formatted, simply converting them using Python‘s datetime module was not an option. Instead, I split each string into a list of strings, anywhere a comma occurred. Next, I looped over this list and, for each element, checked the value of the numerical element that it contained and the amount of time it referred to (e.g. by checking if it contained the word ‘year’). With this information, I was able to determine the amount of time each list element corresponded to in days, and sum these values to get the pet’s age in days. Any rows from the ‘horses’ category did not have a value for pet_age but had one for birth_year instead. For these, I calculated the horse’s age in days (rounded to the year) by subtracting their birth year from 2023 and multiplying this value by 365. With these new values calculated, I added them to a new pets_age_in_days column and dropped pet_age and birth_year.

The pets_in_litter column contained variations on the text ‘x males / x females’. As such, I wanted to split this into two continuously valued columns, males_in_litter and females_in_litter. I did this in a very similar manner to that described above for pet_age, dropping pets_in_litter once the two new columns had been created. 

The columns 'pet_sex', 'gender' and 'neutered' had some semantic overlap which needed to be disentangled. 'pet_sex' had three levels (male, female and mixed). 'gender' was specific to horse listings and also has three levels (mare, gelding and stallion). 'neutered' only has two levels (yes, no) However, a gelding is a neutered male horse and all the horse rows all had NaNs for the 'neutered' column. I needed to sort these columns so that the information stored in the horse-specific 'gender' column could be moved to the general 'pet_sex' and 'neutered' columns. To do this, I updated all geldings to pet_sex = male, neutered = yes, stallions to pet_sex = male, neutered = no and mares to pet_sex = female and neutered = no. After this, I dropped the ‘gender’ column. Although I cannot be certain that all mares in the dataset were not neutered, I felt comfortable assuming this based on my background research. In the UK, neutering a mare is very rarely done. This is because it is an extremely risky operation and typically only ever done as a lifesaving procedure.

Having disentangled gender and pet_sex, there was still some semantic overlap between the pet_sex and fe/males_in_litter. Specifically, columns with non-NaN values for pet_sex all had NaNs for fe/males_in_litter. Where the litter was all male or all female, I updated pet_sex to match that. Where there are both male and female animals in the litter, I updated pet_sex can to ‘mixed’. I decided to keep both columns as fe/males_in_litter indicates a) that the pets are infants and b) the number of either gender, which is absent from pet_sex. However, pet_sex couldn’t be dropped either. as some of the pets were adult animals and so would have a fe/males_in_litter value of 0. Finally, as fish are unsexed on Pets4Homes, so added ‘Not applicable’ to pet_sex for all fish rows.

Both the cats and dogs categories had information relating to whether the pet was registered with an owners club/association. For dogs, this was the binary column ‘kc_registered’ (i.e. The Kennel Club). For cats, this was the categorical column ‘registered’, which could indicate that the cat was registered with any, some or all of 3 cat fancy associations, formatted as a string listing association names. I decided to flatten these two columns into a single column which indicated whether a cat or dog was registered with an owners club or association. 

Finally, there was a large number of columns which were specific to cats and/or dogs or horses. For each of these columns, any non-cat/dog/horse row contained NaNs. To address this, I added the value ‘Not applicable’ to these rows. For any cat/dog/horse rows with NaNs for these columns, I added the value ‘Unlisted’. Additionally, due to the very small number of non-NaN entries, I dropped the horse-specific origin and height columns and converted the dressage_level and jumping_level columns into binary columns simply indicating whether the horse had a dressage or jumping ranking.

#### Cleaned data dictionary <a name="Cleaned-data-dictionary"></a>

At this point, no NaNs remained. Title, description, url, advert_ID and the seller verification columns did not require any cleaning. The final cleaned data set was 19,271 rows and 35 columns, containing the following information:

| *Variable name*      | *Variable description*                                                   |
|----------------------|--------------------------------------------------------------------------|
| title                | The title of the listing                                                 |
| price                | The price of the listed pet                                              |
| url                  | The URL of the listing                                                   |
| phone_verified       | Whether the seller's profile has been verified by phone                  |
| email_verified       | Whether the seller's profile has been verified by email                  |
| facebook_verified    | Whether the seller's profile has been verified by Facebook               |
| google_verified      | Whether the seller's profile has been verified by Google                 |
| n_images             | The number of images included in the listing                             |
| category             | The type of animal                                                       |
| advert_id            | The listing's unique ID                                                  |
| advert_location      | The location of the seller                                               |
| advertiser_type      | The type of seller                                                       |
| breed                | The breed of the pet(s) being sold                                       |
| pet_age              | The age of the pet(s) being sold                                         |
| pet_colour           | The colour of the pet(s) being sold                                      |
| pet_sex              | The sex of the pet(s) being sold                                         |
| description          | The listing description provided by the seller                           |
| health_checked       | Whether the pets(s) have been health checked                             |
| microchipped         | Whether the pets(s) have been microchipped                               |
| neutered             | Whether the pet(s) have been neutered                                    |
| vaccinated           | Whether the pet(s) have been vaccinated                                  |
| worm_treated         | Whether the pet(s) have been worm treated                                |
| registered           | Whether the pet(s) are registered with a breeders or owners club/society |
| original_breeder     | Whether the seller is the original breeder of the pet                    |
| viewable_with_mother | Whether the pet is viewable with its mother                              |
| category_1           | The primary category of a horse                                          |
| category_2           | The secondary category of a horse                                        |
| height               | The height of a horse (measured in hands)                                |
| origin               | The origin of a horse                                                    |
| jumping_horse        | Whether a horse does show jumping                                        |
| dressage_horse       | Whether a horse does dressage                                            |
| seller_n_adverts     | The number of adverts that seller had at the time of data collection     |
| pets_age_in_days     | The age of the pet(s), measured in days                                  |
| males_in_litter      | The number of male pets in the litter                                    |
| females_in_litter    | The number of female pets in the litter                                  |


## EDA <a name="EDA"></a>

Due to the large number of variables, I decided to approach the exploratory data analysis by breaking down the variables into their statistical data types. As such, I began looking at the continuous variables, followed by the binary variables, the categorical variables and finally the text variables. I will approach this overview of the EDA in the same fashion.

### Continuous variables <a name="Continuous-variables"></a>

I began by using descriptive statistics and a z-scaled box plot (see below) to get a general overview of the continuous variables. These both showed that all the continuous variables had a strong right skew and some significant outliers. For n_images and seller_n_adverts, these outliers are plausible data points. For instance, one seller had 140 adverts - far more than most, but this seller was a business specialising in reptiles. The listing with the most images had 41, more than 4x the mean number of images (8.69), but I was able to confirm this was not an error by checking the listing. However, the max values for price, pet_age_in_days and fe/males_in_litter were clearly incorrect. The highest priced animal in the dataset was over £21m, the eldest animal was over 2000 years old and the largest litter contained 1300 puppies. These outliers needed to be removed.

Based on the above boxplot, I removed any values which were more than 20 stds from the mean of each variable. I then removed some more values based on background knowledge. For instance, according to Pets4Homes, dog "litters of over eight pups are considered to be large, and over ten is rare, although up to seventeen live births have been recorded in some cases!". As such, I decided to drop any rows where the sum of males_in_litter and females_in_litter was greater than 12. I also removed any rows where the animal’s pets_age_in_days value was suspiciously large for its category (e.g. a row with a dog with a pets_age_in_days equivalent to more than 18 years). I also discovered that there were several adverts with prices like £12,345 or £1,234 with unverified sellers, few photos, generic titles or extremely brief descriptions. As these adverts seemed to be fake, I removed any listings with prices matching these values. 

After removing these outliers, I recreated the z-scaled boxplot (see below). All the continuous variables still had values beyond the IQR * 1.5 range (a common threshold for outliers). However, since they also all had a strong right skew, the values were fairly continually distributed through their upper ranges and I had already performed domain-knowledge informed outlier removal, I decided to leave these values unchanged. With that handled, I was able to make the following observations about the continuous variables:
Price - This was the target variable. There was a very large range in price, from 20p up to £11,500. This was likely due to the very different animal categories in the dataset. As such, I have taken a further look at the distribution of prices amongst the individual animal categories below. As both the box plot and mean (£710.45) and mode (£550) show, price has a strong right skew.
n_images - All listings had at least 1 image. The listing with the most had 41. n_images has a slight right skew (mean 8.76, mode 6).
seller_n_adverts - A significant proportion of the adverts were listed by a seller with only one advert, although the majority are listed by sellers with two or more (mode 2). Although the max value is orders of magnitude larger than even the 75th percentile, I have decided to keep this range for the reasons discussed above.
pet_age_in_days - Similarly to price, this variable has a very strong right skew. Values range from newborn (i.e. 0 days) to 25 years (9125 days). I have included an individual breakdown by animal category below.
fe/males_in__litter - These columns showed a strong right skew, but this is due to the large number of 0s for animals where this value is either not applicable or unlisted. It may also be worth creating polynomial features for these as, according to Pets4Homes: "Overly large litters may lead to problems with delivery due to exhaustion, and the possibility of stillborn pups. Having a significant amount of puppies for the dam to feed after the birth can also pose problems, in terms of presenting difficulties with nursing all of the puppies, and ensuring that the dam stays hydrated and eats enough to support all of the puppies."


Histograms showing the distribution of price by animal category All of the histograms for pet prices subsetted by pet category still showed a strong right skew. However, the ranges between animal types varied greatly. For instance, the price range for invertebrates maxes out at around £250, whereas the price range for horses maxes out at around £7000. Clearly, category should be an important predictor for price.
The categories with the least extreme right skew were dogs, cats and horses. These were also the categories with the largest maximum prices. This indicated that the outlier removal I had already done had had little impact on the animal types with narrower price ranges. Given the extent of the skew for these categories, I decided to perform further outlier removal on them. Specifically, based on the z-scaled box plots below the histograms, I removed values more than 8 standard deviations from the mean of each category. This will remove some of the most extreme and suspicious outliers whilst keeping the main distributions intact.

Histograms showing the distribution of pet age by animal category. As with the previous set of histograms, the histograms showing pet age by animal category all exhibited considerable right skew over ranges that were particular to each category. This was unsurprising as pets are most commonly sold as infants. This skew was least pronounced for horses. However, the zscaled box plots below indicated that this skew is less extreme than for price and most categories had a more regularly/densely populated tail, rather than a few extreme outliers. The exceptions to this were invertebrates, rodents and possibly fish. Both invertebrates and rodents had a single value which was extremely far from the main distribution. I removed these two outliers but otherwise leave pet age as is.

Checking the linearity of the relationships between the continuous variables and the target. The four scatter plots below show the relationship between litter size (i.e. males_in_litter + females_in_litter), pets_age_in_days, n_images and n_adverts with the target variable. Each plot has an order 2 regression line fitted through it (n.b. I experimented with other orders, but found 2 the best fit for each). These plots suggest that all of the continuous variables have a nonlinear relationship with the target. Specifically:
smaller and larger litters were associated with a lower price, whereas mid-sized litters were associated with higher prices. This reflects the information from Pets4Homes about healthy litter sizes quoted above.
younger and older pets were associated with higher prices, whereas those in the mid-range were associated with lower prices. This is likely because people are typically interested in buying their pet when it is an infant and as pets age, they become cheaper. However, this would only be up to a point, as some long-lived animals (such as horses, large parrots, large tortoises etc) can be particularly expensive.
smaller and large numbers of images were associated with lower prices, whereas listings with 25 to 30 images were associated with higher prices. I would speculate that this is because more images, in and of itself, is typically a good thing, but very high numbers of images may be associated with other factors which are associated with lower prices. For instance, larger litters may have more images. Animals which are sometimes bred in large numbers (e.g. invertebrates, fish, birds, rodents, rabbits etc.) would require more images and are amongst the cheaper categories.
Smaller values for n_adverts were associated with a higher price. This makes sense, as sellers who list animals which are more time or effort intensive to breed are likely to list fewer animals, but charge greater amounts for them. Whilst the regression line does curve back up as the number of adverts climbs, this is likely due to the single seller with 140 adverts, who may charge more than other sellers who list multiple adverts.
All in all, these four plots indicate that it may be beneficial to create polynomial features for the continuous variables.

Heat map showing correlations amongst continuous variables The heatmap below shows that n_images and fe/males_in_litter were the best linear predictors of price amongst the continuous variables. seller_n_advets was slightly negatively correlated with price. This is likely because animals with longer infancies (dogs, cats or horses), which necessarily require more time and care, are amongst the more expensive categories. Whereas those which could feasibly be bred in larger numbers (leading to more adverts), such as rabbits, rodents and fish, are among the cheaper pets. Due to the non-linear relationships uncovered above, it is unsurprising that there are no strong correlations here.

### Binary variables <a name="Binary-variables"></a>

The first step with the binary variables was to separate the 'Not applicable' and 'Unlisted' values from the columns which would otherwise be purely binary. This allowed me to analyse them at the same time as the purely binary variables (i.e. those relating to verification). Where the column is applicable:
66% of pets are microchipped
6.5% of pets are neutered
75% of pets are vaccinated
91.6% of pets are worm treated
96.6% of pets are being sold by the original breeder
90.2% of pets are viewable with their mother
~100% of sellers are phone verified (20 are not)
~100% of sellers are email verified (41 are not)
15% of sellers are Facebook verified
15% of sellers are Google verified

Heat map showing correlations amongst binary variables and price The heatmap below allows us to see which binary variables have the strongest correlations with the target variable, price. These correlations can be seen on the bottom row of the heatmap. This row shows that an animal being microchipped, vaccinated, worm treated, being sold by the original breeder, being viewable with its mother and not being neutered are all positively correlated with price.
Since most of these features are specific to dogs and cats and these animals are among the more expensive categories, I wanted to check that these positive correlations were not simply due to them being associated with cats and dogs. As such, I created a 2nd heatmap below the first which displays the correlations among the sample variables, but for cats and dogs only. Whilst the 2nd heatmap shows positive correlations between the variables mentioned above and the target variable, the size of the correlations has reduced considerably. This is particularly true for worm_treated_yes and neutered_no (the size of which becomes negligible).

### Categorical variables <a name="Categorical-variables"></a>

There were 8 categorical variables (category, advert_location, advertiser_type, breed, pet_colour, pet_sex, category_1, category_2). For each of these, I created a bar chart indicating the value counts for each level of that particular variable. Each has a small section of markdown discussing the graph above it.
category - by far the most common category of pets was dogs, which made up almost half of the dataset. There were more than twice as many dog listings as the 2nd most common category, cats. Given how much the prices vary between the different categories and how few instances there are of both the horses and invertebrates categories (amongst the most and least expensive categories), it may be difficult to make accurate predictions across all categories.
advert_location - There were 1163 unique locations. Only 408 of these had more than 10 listings, and only 23 had more than 100 listings. By far the most common location was London, which, with over 1000 adverts, had more than double the 2nd most common location, Birmingham.

advertiser_type - There were 5 unique types of advertiser; individual, breeder, licensed breeder, organisation and company. Adverts listed by organisations and companies are extremely rare.

breed - There were 382 unique breeds in the dataset. Of these, 178 had more than 10 listings and 48 had more than 100. Mixed breed was the most common value. However, as this can apply to any animal category, I added the category type to the breed value of any mixed breed animals (e.g. a mixed breed animal in the category dogs becomes 'Mixed Breed Dogs'). I did this as I suspect that an animal being mixed breed may have a different impact on price depending upon the type of animal it is (e.g. a mixed breed dog or cat may be less valuable than a pedigree one, but I do not know if this is likely to be the case for, say, reptiles or birds). After cleaning these values, the most common breed type was Guinea Pig.

pet_colour - This variable needed some additional processing as some animals had multiple values (e.g. 'Brown, black, white'). I separated these using a count vectoriser (n.b. this means single adverts may count towards multiple colour columns). The bar chart shows that all colours except violet and pink appeared in a listing more than 100 times. The most common colour entry was mixed colour, followed by black. For the majority of adverts listed, colour was either not applicable or unlisted.

category_1/category_2 These two variables only apply to horses and specify the primary and secondary categories to which the horse belongs. By far the most common category_1 value was allrounder. The most common category_2 value was other. Given this, and the fact that horses make up a very small percentage of the overall dataset, I did not expect these features to be particularly useful for predicting price.


### Text variables <a name="Text-variables"></a>

Both the title and description columns contained raw text. As such, it was necessary to do some further processing on these before they could be analysed. I vectorised both using a term-frequency inverse document-frequency (tf–idf) vectoriser, which reflects how important a word is for each entry in either column. Specifically, a word’s tf–idf value increases in proportion to the number of times that word appears in a specific title or description entry (i.e. the ‘document’) and is offset by the number of entries in the column (i.e. the ‘corpus’) that contain the word. Having calculated this for each column, I then found the correlation between each word appearing in a row and the target variable, price. I plotted these correlations on bar charts (see below). The correlations are somewhat unsurprising given what we have already learnt about the variables above. For instance, the term which most strongly correlates with the price is ‘kc’ (as in kennel club) for both the title and description columns. 

## Modelling <a name="Modelling"></a>

Finally, having completed the EDA, I began modelling the data. In this section, I will outline the small number of steps I took to prepare the data for modelling, the selection of models I tested, which performed best and the final results of the project.

### Feature engineering <a name="Feature-engineering"></a>

All features discussed in the EDA section were included in the modelling stage, except the advert_ID and url (as these are reference features only, not predictors) and the two text variables, title and description (or the vectorised versions of them). I chose not to include the text variables as the EDA suggested that they would not add any information not already captured by the other features. Additionally, it would not be possible to use all of the vectorised words, as this would have left me with more features than instances. As such, I would have needed to find some threshold by which to exclude some of these. I considered using the correlations discussed above for this but ultimately would have been choosing an arbitrary threshold. I plan to explore this further in future work.

As discussed during the EDA, all of the continuous variables had non-linear relationships with the target variable. As such, I created polynomial features for these. I also dummified the categorical variables.
### Splitting and scaling the data <a name="Splitting-and-scaling-the-data"></a>

I split the data set into the target (price) and a set of predictors and then further split these into a train and test set (with the test set consisting of 20% of the total instances). Because the target variable has a very strong right skew, I used stratification when splitting the data. This allowed me to ensure that the distribution of the target in the training and test sets was similar. This can be seen in the histograms below.  

### Building models <a name="Building-models"></a>

I began by trying a large number of model types with some fairly standard values to see what kinds of baseline performance I could expect from each model type. After doing this, I selected the 4 most promising models to work on with parameter tuning. I then performed a deeper analysis of the best of the tuned models. Specifically, this involved looking at the coefficients/feature importances to determine which are most influential on the model's predictions and an analysis of the residuals. Finally, I built a stacking ensemble model from the best tuned models to see whether it could outperform the single best model.

#### Initial models <a name="Initial-models"></a>

I fit 9 different model types on the data and returned their training and test set scores. With this information, we can get a general impression of the performance of each model and whether it is over or underfitting the data.
I used 3 regularised linear regression models. These are; ridge, LASSO and elastic net regression.
Of the three, the ridge regressor performed best on the training set (R2: 0.74), but worst on the test set (R2: 0.6). This indicates that the model has overfit the training data.
This can also be said for the other two models, although the margin of differences between each model's training and test performance is smaller than for the ridge model. Otherwise, the performance of the LASSO and elastic net regressors are similar, with an R2 of ~0.72 for the training set and ~0.62 for the test set. This is perhaps unsurprising as the dataset contains a large number of variables, many of which may not be useful, so the more precise regularisation of LASSO and elastic net are likely to be useful.
Based on this, I decided to include LASSO and elastic net in the parameter tuning stage, but not ridge.
The KNN regressor performed poorly, with an R2 of 0.58 for the training set and 0.34 for the test set. This could potentially be improved through hyperparameter tuning, but given how far this model is behind the others in terms of its performance, I decided not to move forward with it.
The performance of the decision tree model was similar to that of the ridge regressor, only with more pronounced overfitting and a slightly worse test set R2 (0.59). As with ridge and KNN regression, I did not use this in the parameter tuning stage.
There were 4 ensemble models. These were; random forest, extra trees, adaboost and gradient boosting regression.
Of these, adaboost was by far the worst model, attaining a negative R2 on both the training and test sets. This indicates that the model performs worse than simply predicting the mean of the target value for every instance. This is likely because the initial parameters I have chosen are unsuitable. I decided to focus on improving the performance of the other ensemble models but would like to go back and test some other values in iterations of this project.
The random forest regressor had a similar test set R2 (0.61) to the LASSO and elastic net models, but a larger training set R2 (0.74). This indicates that the model's fair performance might be improved with some regularisation and parameter tuning to minimise overfitting.
The extra trees regressor was more similar to that of the ridge model (i.e. slightly higher train set R2 than the best similar models, slightly worse test set R2). On this basis, I didn’t move forward with it.
Finally, the gradient boosting regressor had the best test set performance of any of the models (R2: 0.65), but with a training set R2 of 0.92, it had massively overfit the data. As such, with some parameter tuning and regularisation, this could be a promising model.
In sum, I selected four models to take into the parameter tuning stage. These are LASSO, elastic net, random forest and gradient-boosting regression.

#### Parameter tuning <a name="Parameter-tuning"></a>

This section summarises the parameter tuning steps taken for the four best models.

LASSO regression
Given that I used the LassoCV class in the initial models section, some amount of parameter tuning will already have been done. As such, rather than using grid search with LASSO, I continued using the LassoCV class for hyperparameter tuning but searched over a wider range of parameters. 
By default, LassoCV explores 100 alpha levels, the range of which is determined automatically. I expanded this to explore 500 levels within the automatically determined range. Unfortunately, tuning alpha over a larger range did not improve the test set R2 for LASSO. Even with tuning, it failed to beat the initial gradient boosting model's test set R2.
I also attempted to improve the R2 with LASSO + random patches. I selected this as I did not have time to explore all bagging/pasting approaches and random patches was likely to create the most diverse ensemble (by maximising individual estimators' bias). Without parameter tuning, this performed worse than the tuned LASSO model. However, when I attempted tuning, I crashed my laptop. I would like to explore this further in the future, perhaps using a cloud-based machine to access more compute.
Elastic Net regression
As with the initial LassoCV model, the initial ElasticNetCV model would have done some basic parameter tuning over the values of alpha. So, similarly to above, rather than using grid search, I used the ElasticNetCV class with a larger n_alphas.
ElasticNetCV also allows parameter tuning over the values of l1_ratio (i.e. the ratio of ridge and lasso regression). In the initial model, this would have been the default value (0.5). The documentation for the class notes that “a good choice of list of values for l1_ratio is often to put more values close to 1 (i.e. Lasso) and less close to 0 (i.e. Ridge)”. I chose a parameter range based on this information.
Tuning the alpha value for the elastic net model resulted in a very small improvement on the test set performance (i.e. from 0.62 to 0.63). Interestingly, the best alpha level was quite a bit smaller than that for the LASSO regressor. The best l1_ratio value (l1_ratio) was also greater than the default value. This means that the model was using more l1 than l2 regularisation (given the relative performances of the initial ridge and LASSO models, this is to be expected).
I also tested whether this model could be further improved through the random patches method. I used a smaller number of estimators than before, due to long training times. Nonetheless, the random patches elastic net model performed worse than the original. This is likely because the parameters I chose introduced too much bias (this is also supported by the poorer training set R2). This could probably be improved by parameter tuning, but as noted above, the last time I attempted this for a bagging regressor, I crashed my laptop. As such, for the time being, I will treat the previous model as my final elastic net model. This final model still performs worse than the initial gradient boosting model.

Random forest regresssor
For the random forest regressor, I was attempting to find a set of parameters which could attain a higher test set R2 than the initial model's (0.61) and ideally greater than the initial gradient boost regressor model (0.64). To do this, I focused the grid search on 3 parameters used for regularisation; max_depth, min_samples_split and max_leaf_nodes. I began by looking over a reasonably wide range of values to understand the ideal scale for each parameter.
The best estimator found by the 1st round of parameter tuning reached a test set R2 of 0.63. This was a small improvement on the initial model, but still slightly behind the initial gradient boosting regressor. The best estimator’s parameters are max_depth=40, max_leaf_nodes=400, and min_samples_split=16. This meant that for each parameter, the highest value searched over was found to be the best. This indicated that it could be possible to attain slightly better performance by searching over a range of larger values. As such, I ran another round of parameter tuning with larger values and was able to marginally improve the model’s R2 to 0.64.

Gradient boosting
Gradient boosting was the best of the initial models (with the highest test set R2), but also the most overfit model. Given this, I decided to try to test the effects of halving and doubling both of the parameters specified in the original model. I also added n_estimators to the grid search to counterbalance the effect of varying the learning rate. Unfortunately, I had to stop the parameter early as it was taking an extremely long time to run. I am planning to re-run it in future work. Instead, I attempted some manual parameter tuning. However, I was only able to attain a very small improvement in the model's test set R2 (~0.007). Whilst this is not much of an improvement, it does mean that gradient-boosted regression remains the strongest of the models.

#### Comparing models <a name="Comparing-models"></a>

For each of the tuned models, I have generated four plots. These are; a bar chart showing either the 20 largest (absolute valued) coefficients or 20 most important features, a scatter plot showing the model's predictions against the true values for the target, a histogram showing the distribution of the model's residuals and a Q-Q plot, also showing the distribution of the model's residuals.
Between the four models, the scatterplots, histograms and Q-Q plots are highly similar. All four scatter plots indicate that the models have a slight tendency to underestimate the prices of the listings. Also, the models were unable to replicate sellers’ tendency to price their listings according to round numbers (e.g. £500, £1000 etc.).
The tendency to underestimate listing prices can also be seen in the histograms of all four models, as each has a longer left-hand tail than the right. The scale of these errors seems to be similar in each of the models.
Again, the Q-Q plots for all four models are highly similar. These confirm what can be seen in each model's histogram - the distribution of residuals is very tail-heavy, and the left tail is particularly skewed.
Given the similarity between the four models on these points, I suspect that the underestimation of prices is largely to do with the data itself. As discussed previously, both the target and many of the predictors have highly non-normal distributions. Despite my efforts in cleaning the data, several of the variables still have some mixed-signal because some binary columns are only relevant to certain animal types and thus that a 1 in that column indicates both that the record has that property and is of that animal type (and vice-versa). Given this, in future work, I would like to explore transforming the data to be more normally distributed and building separate models for the various animal categories.
Where the models differ somewhat is in the bar charts indicating strong/important features/coefficients. The two linear regression models both selected registered_1.0 as their largest coefficient, whereas the most important feature for the two ensemble models was neutered_no (which is not even in the LASSO model's top 20). Overall however, all models used predictors we would expect to indicate price quite strongly, i.e. variables which are associated with higher quality breeders for cats and dogs (e.g. vaccinated_yes) and the more expensive categories, such as horses, cats and dogs. For future work, I would like to take a deeper look at the coefficients for each animal breed, sorted by animal category, to determine which breeds best predict price.
Whilst these models could be used to predict the price of a pet listing with reasonable accuracy, their tendency, when wrong, to be wrong by a very large margin (£1000s) does make them quite limited. 

Stacking ensemble
I also experimented with creating a stacking ensemble with the four best models as the base estimators and a gradient-boosting regressor as the meta-learner. The model had a worse test set R2 than the gradient boosting regressor. Further analysis of the distribution of errors between models may be able to explain why this is the case.

## Conclusions and next steps <a name="Conclusions-and-next-steps"></a>
In this project, I set out to build a dataset representing the current state of the UK pet sales market. I did this by scraping user-generated listings from the popular pet sales platform, Pets4Homes. After cleaning the data, I had created a dataset with almost 20,000 instances and 35 features. Through EDA, I discovered that many of these features varied significantly between animal types. However, it did seem that many of these features could be decent predictors for pet prices. In the modelling stage, I experimented with several models and ultimately attained a peak test set R2 performance of 0.65 using a gradient-boosting regressor. All of the models tended to underestimate the target variable, which I suspect is due to its strong right skew. I was unable to build a model which could meet my original R2 target of 0.80+. As such, further improvements will need to be made before these models should be deployed for use by pet sellers and buyers. 

Going forward, I will be working to improve this project with the goal of building a model which can attain my original target R2 (0.80+). Here are some of the steps that I plan to take:

* I would like to re-run the project with a longer data collection period to build a larger dataset. E.g. scraping every listing over several months. I would be aiming to build a data set of 100,000-200,000 listings.
* Using this larger dataset, I would like to re-run the models discussed above, to see if this can help improve their performance. However, more importantly, more data would make it possible to build several category-specific models. I am very curious to see how the models above might perform when not trying to make predictions on animals as different as racehorses and goldfish.
* I would like to experiment with more NLP tools to see whether I can derive some useful predictors from the text variables. In particular, I would like to do sentiment analysis, as I noticed there was a large difference in prices between listings that appeared to be from professional breeders and regular pet owners who are (often reluctantly) selling their pets. I would anticipate that titles and descriptions for the former category would have much more positive sentiment than the latter.
* Bringing in external sources of data could be a way to improve the model’s performance further. For instance, looking at the number of times a breed has been mentioned on Instagram might be a good indicator of it’s desirability.
* Finally, I am interested in attempting to treat this as a classification problem by binning the target variable. Whilst this would be slightly less useful for the project’s stated aims, I suspect that I may be able to build more accurate models. At a minimum, this would allow me to gain a better understanding of which types of instances are most difficult to classify and whether there is any clear pattern among them.
