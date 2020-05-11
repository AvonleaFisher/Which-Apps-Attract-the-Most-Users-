
# Which Apps Attract the Most Users?

This project analyzes data on Android and iOS mobile apps to determine which types of apps are most popular. The project focuses on apps that are free to use and are targeted to an English-speaking audience. We begin by opening the Google Play Store and Apple Store datasets.


```python
#open files

from csv import reader

#Google Play data
opened_file = open('googleplaystore.csv')
read_file = reader(opened_file)
android = list(read_file)
android_header = android[0]
android = android[1:]

#App Store data
opened_file = open('AppleStore.csv')
read_file = reader(opened_file)
ios = list(read_file)
ios_header = ios[0]
ios = ios[1:]
```

## Exploring the data
The explore_data function prints the first few rows of both data sets and lists the number of rows and columns in each.


```python
#Explore data
def explore_data(dataset, start, end, rows_and_columns = False):
    dataset_slice = dataset[start:end]
    for row in dataset_slice:
        print(row)
        print('\n')
    if rows_and_columns:
        print('Number of rows:', len(dataset))
        print('Number of columns:', len(dataset[0]))

explore_data(android, 0, 3, True)
explore_data(ios, 0, 3, True)
```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    Number of rows: 10841
    Number of columns: 13
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of rows: 7197
    Number of columns: 16


## Cleaning the Data

### Remove rows with missing data
In a discussion on the Android data, row 10472 was flagged as containing incorrect data. The print output for this row shows that it has a missing value. 


```python
print(android[10472])
```

    ['Life Made WI-Fi Touchscreen Photo Frame', '1.9', '19', '3.0M', '1,000+', 'Free', '0', 'Everyone', '', 'February 11, 2018', '1.0.19', '4.0 and up']



```python
#delete row with missing value
del android[10472]
```

### Remove duplicate entries
For the analysis, we only want to deal with unique app entries. The Google Play data set has 1,181 duplicate app entries that should be removed.


```python
#Find number of duplicate entries
dup_apps = []
unique_apps = []

for app in android:
    name = app[0]
    if name in unique_apps:
        dup_apps.append(name)
    else:
        unique_apps.append(name)

print('Number of duplictate apps:', len(dup_apps))
print('\n')
print('Examples of duplicate apps:', dup_apps[:15])
```

    Number of duplictate apps: 1181
    
    
    Examples of duplicate apps: ['Quick PDF Scanner + OCR FREE', 'Box', 'Google My Business', 'ZOOM Cloud Meetings', 'join.me - Simple Meetings', 'Box', 'Zenefits', 'Google Ads', 'Google My Business', 'Slack', 'FreshBooks Classic', 'Insightly CRM', 'QuickBooks Accounting: Invoicing & Expenses', 'HipChat - Chat Built for Teams', 'Xero Accounting Software']


For each app entry, the number of user reviews is recorded. This number varies among duplicate entries. For example, the number of reviews (4th element) for Slack is higher in the third printed entry below:


```python
for app in android:
    name = app[0]
    if name == 'Slack':
        print(app)
```

    ['Slack', 'BUSINESS', '4.4', '51507', 'Varies with device', '5,000,000+', 'Free', '0', 'Everyone', 'Business', 'August 2, 2018', 'Varies with device', 'Varies with device']
    ['Slack', 'BUSINESS', '4.4', '51507', 'Varies with device', '5,000,000+', 'Free', '0', 'Everyone', 'Business', 'August 2, 2018', 'Varies with device', 'Varies with device']
    ['Slack', 'BUSINESS', '4.4', '51510', 'Varies with device', '5,000,000+', 'Free', '0', 'Everyone', 'Business', 'August 2, 2018', 'Varies with device', 'Varies with device']


Since apps accrue more reviews over time, the most recent entry in each set of duplicate apps will be the one with the highest number of reviews. Therefore, we will use this criterion to eliminate duplicate apps.


```python
reviews_max = {}
for app in android:
    name = app[0]
    n_reviews = float(app[3])
    if name in reviews_max and reviews_max[name] < n_reviews:
        reviews_max[name] = n_reviews
    elif name not in reviews_max:
        reviews_max[name] = n_reviews        
```

Above, we create a dictionary with the app names as keys and their max number of reviews as values. The print output for Slack confirms that the highest number of reviews was stored as the value:


```python
print(reviews_max['Slack'])
```

    51510.0


The number of rows in the dictionary should be equal to the number of rows in the original android data set minus the number of duplicate entries (1181). The following boolean statement confirms that these two values are equal. 


```python
len(android) - 1181 == len(reviews_max)
```




    True



To remove non-English apps from the dataset, we build a function to detect whether characters in the app's name belong to the English language. Using the ord() built-in function, we can check the number that's associated with a value behind the scenes. If the function shows that the string contains more than 3 values greater than 127, the app
likely does not belong to the English language.


```python
def english(string):
    non_english = 0
    
    for character in string:
        if ord(character) > 127:
            non_english += 1
    
    if non_english > 3:
        return False
    else:
        return True
```

Let's test the function on a variety of app names:


```python
print(english('Instagram'))
print(english('爱奇艺PPS -《欢乐颂2》电视剧热播'))
print(english('Docs To Go™ Free Office Suite'))
print(english('Instachat 😜'))
```

    True
    False
    True
    True


The function can now be used to filter out non-English apps from the data. To do this, we can create new empty lists and append them with the filtered data. 


```python
android_english = []

for app in android:
    if english(app[0]) == True:
        android_english.append(app)

ios_english = []

for app in ios:
    if english(app[0]) == True:
        ios_english.append(app)
```

By exploring the data, we find that the filtered android data has fewer rows than the original android data.


```python
print('Android unfiltered:')
explore_data(android, 0, 0, True)
print('\n')
print('Android filtered:')
explore_data(android_english, 0, 0, True)
print('\n')
print('ios unfiltered:')
explore_data(ios, 0, 0, True)
print('\n')
print('ios filtered:')
explore_data(ios_english, 0, 0, True)
```

    Android unfiltered:
    Number of rows: 10840
    Number of columns: 13
    
    
    Android filtered:
    Number of rows: 10795
    Number of columns: 13
    
    
    ios unfiltered:
    Number of rows: 7197
    Number of columns: 16
    
    
    ios filtered:
    Number of rows: 7197
    Number of columns: 16


### Removing non-free apps
Finally, we will filter the data further to only include free apps. By printing the headers of both datasets, we find the indices with the price data. By then printing the first row of the English app datasets, we find the values that correspond to free apps. 


```python
print('Android price in app[7]:')
print(android_header) 
print('\n')
print('ios price in app[4]:')
print(ios_header)
print('\n')
print('Free ios apps denoted by 0.0')
print(ios_english[0])
print('\n')
print('Free android apps denoted by 0')
print(android_english[0])
```

    Android price in app[7]:
    ['App', 'Category', 'Rating', 'Reviews', 'Size', 'Installs', 'Type', 'Price', 'Content Rating', 'Genres', 'Last Updated', 'Current Ver', 'Android Ver']
    
    
    ios price in app[4]:
    ['id', 'track_name', 'size_bytes', 'currency', 'price', 'rating_count_tot', 'rating_count_ver', 'user_rating', 'user_rating_ver', 'ver', 'cont_rating', 'prime_genre', 'sup_devices.num', 'ipadSc_urls.num', 'lang.num', 'vpp_lic']
    
    
    Free ios apps denoted by 0.0
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    Free android apps denoted by 0
    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']


Next, we create two new empty lists for our final data sets, and then append them with apps that are free.


```python
android_eng_free = []
ios_eng_free = []

for app in android_english:
    if app[7] == '0':
        android_eng_free.append(app)

for app in ios_english:
    if app[4] == '0.0':
        ios_eng_free.append(app)
```

Exploring the final datasets shows that we are left with 9999 apps for the Android data and 4056 apps for the ios data.


```python
print('Android:')
explore_data(android_eng_free, 0, 0, True)
print('ios:')
explore_data(ios_eng_free, 0, 0, True)
```

    Android:
    Number of rows: 9999
    Number of columns: 13
    ios:
    Number of rows: 4056
    Number of columns: 16


## Analyzing the data

Our goal in analyzing this final data is to find an app profile that is successful in both Google Play and the App store. If a minimal version of an app based on this profile is successful in Google Play, it will be developed further. If the app is profitable after six months, it will also be added to the App Store. 
### Most Common Apps
We'll begin our analysis by determining the most common app genres in both markets. From the headers printed in the last step, we can infer that 'prime_genre'(iOS), 'genres' and 'category' (Android) are the columns that correspond to the app genres. Let's print those columns to confirm.


```python
for app in android_eng_free[1:5]:
    print(app[1])
print('\n')
for app in android_eng_free[1:5]:
    print(app[-4])
print('\n')
for app in ios_eng_free[1:5]:
    print(app[11])
```

    ART_AND_DESIGN
    ART_AND_DESIGN
    ART_AND_DESIGN
    ART_AND_DESIGN
    
    
    Art & Design;Pretend Play
    Art & Design
    Art & Design
    Art & Design;Creativity
    
    
    Photo & Video
    Games
    Games
    Music



```python
def freq_table(dataset, index):
    frequency_table = {}
    count = 0
    for row in dataset:
        count += 1
        data_point = row[index]
        if data_point in frequency_table:
            frequency_table[data_point] += 1
        else:
            frequency_table[data_point] = 1
    
    percentages = {}
    for key in frequency_table:
        percentage = (frequency_table[key]/count) * 100
        percentages[key] = percentage
    return percentages

def display_table(dataset, index):
    table = freq_table(dataset, index)
    table_display = []
    for key in table:
        key_val_as_tuple = (table[key], key)
        table_display.append(key_val_as_tuple)

    table_sorted = sorted(table_display, reverse = True)
    for entry in table_sorted:
        print(entry[1], ':', entry[0])
```


```python
print('iOS:')
display_table(ios_eng_free, 11)
```

    iOS:
    Games : 55.64595660749507
    Entertainment : 8.234714003944774
    Photo & Video : 4.117357001972387
    Social Networking : 3.5256410256410255
    Education : 3.2544378698224854
    Shopping : 2.983234714003945
    Utilities : 2.687376725838264
    Lifestyle : 2.3175542406311638
    Finance : 2.0710059171597637
    Sports : 1.947731755424063
    Health & Fitness : 1.8737672583826428
    Music : 1.6518737672583828
    Book : 1.6272189349112427
    Productivity : 1.5285996055226825
    News : 1.4299802761341223
    Travel : 1.3806706114398422
    Food & Drink : 1.0601577909270217
    Weather : 0.7642998027613412
    Reference : 0.4930966469428008
    Navigation : 0.4930966469428008
    Business : 0.4930966469428008
    Catalogs : 0.22189349112426035
    Medical : 0.19723865877712032


The frequency table percentages show that games and entertainment are the most common app genres in the App Store data set. Social networking and photo & video apps are also common. While entertainment is a distinct genre, all of these common app genres could be said to have entertainment as one of their primary purposes. 


```python
print('Android Genres:')
display_table(android_eng_free, -4)
```

    Android Genres:
    Tools : 7.630763076307631
    Entertainment : 6.0006000600060005
    Education : 5.1305130513051305
    Business : 4.45044504450445
    Productivity : 3.95039503950395
    Sports : 3.74037403740374
    Communication : 3.5903590359035906
    Lifestyle : 3.5803580358035805
    Medical : 3.5403540354035403
    Finance : 3.49034903490349
    Action : 3.4103410341034106
    Health & Fitness : 3.2503250325032504
    Photography : 3.1203120312031203
    Personalization : 3.08030803080308
    Social : 2.9202920292029204
    News & Magazines : 2.7702770277027704
    Shopping : 2.5702570257025705
    Travel & Local : 2.45024502450245
    Dating : 2.2702270227022705
    Arcade : 2.0002000200020005
    Books & Reference : 1.9901990199019903
    Simulation : 1.8801880188018802
    Casual : 1.84018401840184
    Video Players & Editors : 1.6801680168016802
    Maps & Navigation : 1.3001300130013
    Food & Drink : 1.25012501250125
    Puzzle : 1.21012101210121
    Racing : 0.9500950095009502
    Strategy : 0.9300930093009301
    House & Home : 0.88008800880088
    Role Playing : 0.87008700870087
    Libraries & Demo : 0.8400840084008401
    Auto & Vehicles : 0.8200820082008201
    Weather : 0.7400740074007401
    Events : 0.6300630063006301
    Adventure : 0.6200620062006201
    Comics : 0.58005800580058
    Art & Design : 0.54005400540054
    Beauty : 0.53005300530053
    Parenting : 0.44004400440044
    Education;Education : 0.4300430043004301
    Card : 0.41004100410041006
    Educational;Education : 0.38003800380038005
    Casino : 0.38003800380038005
    Trivia : 0.37003700370037007
    Board : 0.35003500350035005
    Educational : 0.33003300330033003
    Word : 0.29002900290029
    Entertainment;Music & Video : 0.27002700270027
    Casual;Pretend Play : 0.25002500250025006
    Music : 0.21002100210021002
    Casual;Action & Adventure : 0.2000200020002
    Racing;Action & Adventure : 0.19001900190019003
    Puzzle;Brain Games : 0.17001700170017
    Educational;Pretend Play : 0.14001400140014
    Action;Action & Adventure : 0.14001400140014
    Casual;Brain Games : 0.13001300130013002
    Arcade;Action & Adventure : 0.12001200120012002
    Simulation;Action & Adventure : 0.11001100110011
    Adventure;Action & Adventure : 0.11001100110011
    Entertainment;Brain Games : 0.08000800080008001
    Education;Pretend Play : 0.08000800080008001
    Board;Brain Games : 0.08000800080008001
    Parenting;Education : 0.07000700070007
    Casual;Creativity : 0.07000700070007
    Art & Design;Creativity : 0.07000700070007
    Role Playing;Action & Adventure : 0.06000600060006001
    Parenting;Music & Video : 0.06000600060006001
    Educational;Brain Games : 0.06000600060006001
    Role Playing;Pretend Play : 0.05000500050005
    Puzzle;Action & Adventure : 0.05000500050005
    Education;Music & Video : 0.05000500050005
    Education;Creativity : 0.05000500050005
    Educational;Action & Adventure : 0.040004000400040006
    Education;Brain Games : 0.040004000400040006
    Education;Action & Adventure : 0.040004000400040006
    Video Players & Editors;Music & Video : 0.030003000300030006
    Simulation;Pretend Play : 0.030003000300030006
    Entertainment;Creativity : 0.030003000300030006
    Entertainment;Action & Adventure : 0.030003000300030006
    Educational;Creativity : 0.030003000300030006
    Video Players & Editors;Creativity : 0.020002000200020003
    Sports;Action & Adventure : 0.020002000200020003
    Puzzle;Creativity : 0.020002000200020003
    Music;Music & Video : 0.020002000200020003
    Entertainment;Pretend Play : 0.020002000200020003
    Casual;Music & Video : 0.020002000200020003
    Casual;Education : 0.020002000200020003
    Board;Action & Adventure : 0.020002000200020003
    Art & Design;Pretend Play : 0.020002000200020003
    Art & Design;Action & Adventure : 0.020002000200020003
    Adventure;Education : 0.020002000200020003
    Trivia;Education : 0.010001000100010001
    Travel & Local;Action & Adventure : 0.010001000100010001
    Tools;Education : 0.010001000100010001
    Strategy;Education : 0.010001000100010001
    Strategy;Creativity : 0.010001000100010001
    Strategy;Action & Adventure : 0.010001000100010001
    Simulation;Education : 0.010001000100010001
    Role Playing;Brain Games : 0.010001000100010001
    Racing;Pretend Play : 0.010001000100010001
    Puzzle;Education : 0.010001000100010001
    Parenting;Brain Games : 0.010001000100010001
    Music & Audio;Music & Video : 0.010001000100010001
    Lifestyle;Pretend Play : 0.010001000100010001
    Lifestyle;Education : 0.010001000100010001
    Health & Fitness;Education : 0.010001000100010001
    Health & Fitness;Action & Adventure : 0.010001000100010001
    Entertainment;Education : 0.010001000100010001
    Communication;Creativity : 0.010001000100010001
    Comics;Creativity : 0.010001000100010001
    Card;Brain Games : 0.010001000100010001
    Card;Action & Adventure : 0.010001000100010001
    Books & Reference;Education : 0.010001000100010001
    Arcade;Pretend Play : 0.010001000100010001



```python
print('Android Category:')
display_table(android_eng_free, 1)
```

    Android Category:
    FAMILY : 17.67176717671767
    GAME : 10.591059105910592
    TOOLS : 7.640764076407641
    BUSINESS : 4.45044504450445
    PRODUCTIVITY : 3.95039503950395
    SPORTS : 3.6003600360036003
    LIFESTYLE : 3.5903590359035906
    COMMUNICATION : 3.5903590359035906
    MEDICAL : 3.5403540354035403
    FINANCE : 3.49034903490349
    HEALTH_AND_FITNESS : 3.2503250325032504
    PHOTOGRAPHY : 3.1203120312031203
    PERSONALIZATION : 3.08030803080308
    SOCIAL : 2.9202920292029204
    NEWS_AND_MAGAZINES : 2.7702770277027704
    SHOPPING : 2.5702570257025705
    TRAVEL_AND_LOCAL : 2.4602460246024602
    DATING : 2.2702270227022705
    BOOKS_AND_REFERENCE : 1.9901990199019903
    VIDEO_PLAYERS : 1.7001700170017002
    EDUCATION : 1.5101510151015103
    ENTERTAINMENT : 1.4701470147014701
    MAPS_AND_NAVIGATION : 1.3001300130013
    FOOD_AND_DRINK : 1.25012501250125
    HOUSE_AND_HOME : 0.88008800880088
    LIBRARIES_AND_DEMO : 0.8400840084008401
    AUTO_AND_VEHICLES : 0.8200820082008201
    WEATHER : 0.7400740074007401
    EVENTS : 0.6300630063006301
    ART_AND_DESIGN : 0.6100610061006101
    COMICS : 0.59005900590059
    PARENTING : 0.58005800580058
    BEAUTY : 0.53005300530053


Tools, entertainment, and education are the most common genres in the Android data set. The most common app categories are family, game, and tools. Given that tools appeared most frequently for both variables, we can infer that apps used for practical purposes are more common on Google Play than on the App Store.

These frequency tables only tell us which app types are the most common. They don't contain information about which genres have the most users. In order to make an app profile recommendation, we will also need to analyze data on the number of users across different categories in both datasets.

### Apps with the most users
The Android data set contains data for the number of installs for each app in column 5. The App Store data set does not include this information, but in column 5, it does include the number of user ratings given for each app. We can use this data as a proxy for the number of users.
### iOS

We begin by generating a frequency table for the unique genres in the app store data set. We then loop over these genres, and create variables for the number of user ratings (total) and number of apps(len_genre) in each genre. Within this loop, we loop over the App Store data to update the variables. Finally, we divide the total number  of user ratings by the number of apps to get the average number of ratings for each genre. 


```python
unique_ios = freq_table(ios_eng_free, -5)

for genre in unique_ios:
    total = 0
    len_genre = 0
    for app in ios_eng_free:
        genre_app = app[-5]
        if genre_app == genre:
            user_ratings = float(app[5])
            total += user_ratings
            len_genre += 1
    avg_ratings = total/len_genre
    print(genre, ':', int(avg_ratings))
```

    Shopping : 18746
    Business : 6367
    Food & Drink : 20179
    Navigation : 25972
    News : 15892
    Entertainment : 10822
    Education : 6266
    Lifestyle : 8978
    Sports : 20128
    Weather : 47220
    Music : 56482
    Reference : 67447
    Social Networking : 53078
    Health & Fitness : 19952
    Games : 18924
    Medical : 459
    Book : 8498
    Travel : 20216
    Finance : 13522
    Catalogs : 1779
    Utilities : 14010
    Productivity : 19053
    Photo & Video : 27249


Reference, social networking, and music apps have the greatest average number of ratings. Let's examine these categories by printing the ratings counts and app names for each.

#### Reference


```python
total = 0
for app in ios_eng_free:
    if app[-5] == 'Reference':
        total += (int(app[5]))
print(total)
```

    1348958



```python
for app in ios_eng_free:
    if app[-5] == 'Reference':
        print(app[1], ' ', app[5])
```

    Bible   985920
    Dictionary.com Dictionary & Thesaurus   200047
    Dictionary.com Dictionary & Thesaurus for iPad   54175
    Google Translate   26786
    Muslim Pro: Ramadan 2017 Prayer Times, Azan, Quran   18418
    New Furniture Mods - Pocket Wiki & Game Tools for Minecraft PC Edition   17588
    Merriam-Webster Dictionary   16849
    Night Sky   12122
    City Maps for Minecraft PE - The Best Maps for Minecraft Pocket Edition (MCPE)   8535
    LUCKY BLOCK MOD ™ for Minecraft PC Edition - The Best Pocket Wiki & Mods Installer Tools   4693
    GUNS MODS for Minecraft PC Edition - Mods Tools   1497
    Guides for Pokémon GO - Pokemon GO News and Cheats   826
    WWDC   762
    Horror Maps for Minecraft PE - Download The Scariest Maps for Minecraft Pocket Edition (MCPE) Free   718
    VPN Express   14
    Real Bike Traffic Rider Virtual Reality Glasses   8
    教えて!goo   0
    彩库宝典-【官方版】   0
    Jishokun-Japanese English Dictionary & Translator   0
    無料で音楽や写真・カメラの裏技アプリ for iPhone7   0


#### Social Networking


```python
total = 0
for app in ios_eng_free:
    if app[-5] == 'Social Networking':
        total += (int(app[5]))
print(total)
```

    7590182



```python
for app in ios_eng_free:
    if app[-5] == 'Social Networking':
        print(app[1], ':', app[5])
```

    Facebook : 2974676
    Pinterest : 1061624
    Skype for iPhone : 373519
    Messenger : 351466
    Tumblr : 334293
    WhatsApp Messenger : 287589
    Kik : 260965
    ooVoo – Free Video Call, Text and Voice : 177501
    TextNow - Unlimited Text + Calls : 164963
    Viber Messenger – Text & Call : 164249
    Followers - Social Analytics For Instagram : 112778
    MeetMe - Chat and Meet New People : 97072
    We Heart It - Fashion, wallpapers, quotes, tattoos : 90414
    InsTrack for Instagram - Analytics Plus More : 85535
    Tango - Free Video Call, Voice and Chat : 75412
    LinkedIn : 71856
    Match™ - #1 Dating App. : 60659
    Skype for iPad : 60163
    POF - Best Dating App for Conversations : 52642
    Timehop : 49510
    Find My Family, Friends & iPhone - Life360 Locator : 43877
    Whisper - Share, Express, Meet : 39819
    Hangouts : 36404
    LINE PLAY - Your Avatar World : 34677
    WeChat : 34584
    Badoo - Meet New People, Chat, Socialize. : 34428
    Followers + for Instagram - Follower Analytics : 28633
    GroupMe : 28260
    Marco Polo Video Walkie Talkie : 27662
    Miitomo : 23965
    SimSimi : 23530
    Grindr - Gay and same sex guys chat, meet and date : 23201
    Wishbone - Compare Anything : 20649
    imo video calls and chat : 18841
    After School - Funny Anonymous School News : 18482
    Quick Reposter - Repost, Regram and Reshare Photos : 17694
    Weibo HD : 16772
    Repost for Instagram : 15185
    Live.me – Live Video Chat & Make Friends Nearby : 14724
    Nextdoor : 14402
    Followers Analytics for Instagram - InstaReport : 13914
    YouNow: Live Stream Video Chat : 12079
    FollowMeter for Instagram - Followers Tracking : 11976
    LINE : 11437
    eHarmony™ Dating App - Meet Singles : 11124
    Discord - Chat for Gamers : 9152
    QQ : 9109
    Telegram Messenger : 7573
    Weibo : 7265
    Periscope - Live Video Streaming Around the World : 6062
    Chat for Whatsapp - iPad Version : 5060
    QQ HD : 5058
    Followers Analysis Tool For Instagram App Free : 4253
    live.ly - live video streaming : 4145
    Houseparty - Group Video Chat : 3991
    SOMA Messenger : 3232
    Monkey : 3060
    百度贴吧-全球最大兴趣交友社区 : 2860
    Down To Lunch : 2535
    Flinch - Video Chat Staring Contest : 2134
    Highrise - Your Avatar Community : 2011
    LOVOO - Dating Chat : 1985
    PlayStation®Messages : 1918
    MOMO陌陌-开启视频社交,用直播分享生活 : 1862
    BOO! - Video chat camera with filters & stickers : 1805
    Qzone : 1649
    Chatous - Chat with new people : 1609
    Kiwi - Q&A : 1538
    GhostCodes - a discovery app for Snapchat : 1313
    Jodel : 1193
    FireChat : 1037
    Google Duo - simple video calling : 1033
    Fiesta by Tango - Chat & Meet New People : 885
    Google Allo — smart messaging : 862
    Peach — share vividly : 727
    Hey! VINA - Where Women Meet New Friends : 719
    Battlefield™ Companion : 689
    All Devices for WhatsApp - Messenger for iPad : 682
    YY- 小全民手机直播交友软件 : 624
    百度贴吧HD : 542
    Chat for Pokemon Go - GoChat : 500
    IAmNaughty – Dating App to Meet New People Online : 463
    Qzone HD : 458
    Zenly - Locate your friends in realtime : 427
    League of Legends Friends : 420
    豆瓣 : 407
    Candid - Speak Your Mind Freely : 398
    知乎 : 397
    Selfeo : 366
    Fake-A-Location Free ™ : 354
    Popcorn Buzz - Free Group Calls : 281
    Fam — Group video calling for iMessage : 279
    QQ International : 274
    Ameba : 269
    SoundCloud Pulse: for creators : 240
    Tantan : 235
    Cougar Dating & Life Style App for Mature Women : 213
    Rawr Messenger - Dab your chat : 180
    WhenToPost: Best Time to Post Photos for Instagram : 158
    Inke—Broadcast an amazing life : 147
    same - 就是聊得来 : 78
    Mustknow - anonymous video Q&A : 53
    CTFxCmoji : 39
    タップル誕生 - 出会いアプリで趣味から恋活・婚活 : 37
    Lobi : 36
    Chain: Collaborate On MyVideo Story/Group Video : 35
    花椒直播-高清美颜直播互动平台 : 19
    ひみつの出会い探しはバクアイ-無料の出会いチャットアプリで友達作り : 13
    Jメール 出会える人気の匿名SNS出会い系アプリ : 11
    botman - Real time video chat : 7
    秀色秀场-网红直播 聊天交友 : 6
    非诚勿扰-中国最大免费婚恋交友平台 : 3
    美播 - 玩不尽的真人游戏 : 1
    ピグパーティ - きせかえ・アバターで楽しむトークアプリ : 1
    派派激情版-寂陌陌声人交友神器 : 0
    视吧-全民手机直播 : 0
    マッチングならSecret- 友達・恋人探しSNS : 0
    BestieBox : 0
    おチャベリ-ビデオ通話でマッチングアプリ : 0
    暇なら話そう！誰でも話せて友達も作れる「KoeTomo」 : 0
    出会い無料のチャット掲示板 LINK : 0
    出会い応援チャットアプリLIKE YOU : 0
    BiBi娱乐社区 - 好玩的游戏都在这里！ : 0
    HONNE -本音が言える匿名つぶやき&チャットアプリ : 0
    NOW直播—腾讯旗下全民直播平台 : 0
    【完全無料】ソク出会える！ひみつフレ探し掲示板 : 0
    陌爱神器-不闲聊！陌生人快速约见面平台 : 0
    【完全無料】おとな専用！出会い掲示板 : 0
    洋葱圈—正经人的不正经聊天工具 : 0
    出会い系無料で遊べるsnsアプリ内緒チャットトーク : 0
    出会い秘密チャット - 相席 出会い系チャット : 0
    MATCH ON LINE chat : 0
    大人の出会いチャット  - 出会い系 アプリ : 0
    niconico ch : 0
    ライブチャット、ビデオチャット通話が楽しめる！ ライブでゴーゴー : 0
    ゲーム攻略「SGGP」掲示板、SNSな友達出会い : 0
    LINE BLOG : 0
    FixYou-無料でチャットトークができるSNSマッチングアプリ : 0
    bit-tube - Live Stream Video Chat : 0
    出会い系アプリ i-Mail（アイメール） : 0
    同城爱约会-单身男女约会神器 : 0
    マッチング チャット - マッチング&恋人探しチャット : 0
    友達 恋人探し であい ちゃっと sns -ギャルとも : 0


#### Music


```python
total = 0
for app in ios_eng_free:
    if app[-5] == 'Music':
        total += (int(app[5]))
print(total)
```

    3784296



```python
for app in ios_eng_free:
    if app[-5] == 'Music':
        print(app[1], ':', app[5])
```

    Pandora - Music & Radio : 1126879
    Spotify Music : 878563
    Shazam - Discover music, artists, videos & lyrics : 402925
    iHeartRadio – Free Music & Radio Stations : 293228
    SoundCloud - Music & Audio : 135744
    Magic Piano by Smule : 131695
    Smule Sing! : 119316
    TuneIn Radio - MLB NBA Audiobooks Podcasts Music : 110420
    Amazon Music : 106235
    SoundHound Song Search & Music Player : 82602
    Sonos Controller : 48905
    Bandsintown Concerts : 30845
    Karaoke - Sing Karaoke, Unlimited Songs! : 28606
    My Mixtapez Music : 26286
    Sing Karaoke Songs Unlimited with StarMaker : 26227
    Ringtones for iPhone & Ringtone Maker : 25403
    Musi - Unlimited Music For YouTube : 25193
    AutoRap by Smule : 18202
    Spinrilla - Mixtapes For Free : 15053
    Napster - Top Music & Radio : 14268
    edjing Mix:DJ turntable to remix and scratch music : 13580
    Free Music - MP3 Streamer & Playlist Manager Pro : 13443
    Free Piano app by Yokee : 13016
    Google Play Music : 10118
    Certified Mixtapes - Hip Hop Albums & Mixtapes : 9975
    TIDAL : 7398
    YouTube Music : 7109
    Nicki Minaj: The Empire : 5196
    Sounds app - Music And Friends : 5126
    SongFlip - Free Music Streamer : 5004
    Simple Radio - Live AM & FM Radio Stations : 4787
    Deezer - Listen to your Favorite Music & Playlists : 4677
    Ringtones for iPhone with Ringtone Maker : 4013
    Bose SoundTouch : 3687
    Amazon Alexa : 3018
    DatPiff : 2815
    Trebel Music - Unlimited Music Downloader : 2570
    Free Music Play - Mp3 Streamer & Player : 2496
    Acapella from PicPlayPost : 2487
    Coach Guitar - Lessons & Easy Tabs For Beginners : 2416
    Musicloud - MP3 and FLAC Music Player for Cloud Platforms. : 2211
    Piano - Play Keyboard Music Games with Magic Tiles : 1636
    Boom: Best Equalizer & Magical Surround Sound : 1375
    Music Freedom - Unlimited Free MP3 Music Streaming : 1246
    AmpMe - A Portable Social Party Music Speaker : 1047
    Medly - Music Maker : 933
    Bose Connect : 915
    Music Memos : 909
    QQ音乐-来这里“发现・音乐” : 745
    UE BOOM : 612
    LiveMixtapes : 555
    NOISE : 355
    MP3 Music Player & Streamer for Clouds : 329
    Musical Video Maker - Create Music clips lip sync : 320
    Cloud Music Player - Downloader & Playlist Manager : 319
    Remixlive - Remix loops with pads : 288
    QQ音乐HD : 224
    Blocs Wave - Make & Record Music : 158
    PlayGround • Music At Your Fingertips : 150
    Music and Chill : 135
    The Singing Machine Mobile Karaoke App : 130
    radio.de - Der Radioplayer : 64
    Free Music -  Player & Streamer  for Dropbox, OneDrive & Google Drive : 46
    NRJ Radio : 38
    Smart Music: Streaming Videos and Radio : 17
    BOSS Tuner : 13
    PetitLyrics : 0


We can see that for each of the most common App Store genres, only a handful of highly popular apps account for large percentages of the user ratings. By printing the percentages for the app with the most ratings in each category, we see that all of them single-handedly account for more than a quarter of the ratings.   


```python
#Bible
print(int(985920/1348958 * 100))

#Facebook
print(int(2974676/7590182 * 100))

#Pandora
print(int(1126879/3784296 * 100))

```

    73
    39
    29


This tells us than an app profile with a similar concept to any of these highly popular apps likely would not be competitive. It may be more worthwhile to design an app that combines some of their functions while adding new ones. For example, an app that's focused on sharing and discussing books, music, and other media with friends would offer an attractive social element, while also providing entertainment and educational value. Users' experience could vary greatly depending on the type of media they and their friends choose to share, which means that the app could attract a large and diverse audience.

### Google Play


```python
unique_android = freq_table(android_eng_free, 1)

for category in unique_android:
    total = 0
    len_category = 0
    for app in android_eng_free:
        category_app = app[1]
        if category_app == category:            
            installs_count = app[5]
            installs_count = installs_count.replace(',', '')
            installs_count = installs_count.replace('+', '')
            total += float(installs_count)
            len_category += 1
    avg_installs = total / len_category
    print(category, ':', int(avg_installs))
```

    COMICS : 950443
    EVENTS : 253542
    FINANCE : 2511355
    HOUSE_AND_HOME : 1917187
    PERSONALIZATION : 7533233
    FAMILY : 5784094
    WEATHER : 5747142
    GAME : 33111302
    BEAUTY : 513151
    LIFESTYLE : 1479956
    MEDICAL : 147563
    SOCIAL : 48184458
    SPORTS : 4860918
    ART_AND_DESIGN : 2038050
    EDUCATION : 5760596
    LIBRARIES_AND_DEMO : 749950
    DATING : 1164270
    NEWS_AND_MAGAZINES : 27058831
    PRODUCTIVITY : 35885137
    BOOKS_AND_REFERENCE : 9655197
    AUTO_AND_VEHICLES : 647317
    ENTERTAINMENT : 19516734
    FOOD_AND_DRINK : 2190710
    PHOTOGRAPHY : 32321374
    SHOPPING : 12637504
    HEALTH_AND_FITNESS : 4869225
    VIDEO_PLAYERS : 36599010
    COMMUNICATION : 90935671
    PARENTING : 542603
    MAPS_AND_NAVIGATION : 5569698
    BUSINESS : 2250454
    TRAVEL_AND_LOCAL : 27921561
    TOOLS : 14988276


Communication apps have the highest number of installs: 90,935,671. Social and productivity apps are also among the app categories with the highest number of installs. We can create a list of the most installed apps in each of these categories to get a sense of which apps have the most users.


```python
most_installed = []
for app in android_eng_free:
    if (app[1] == 'PRODUCTIVITY' or app[1] == 'COMMUNICATION' or app[1] == 'SOCIAL') and app[5] == '1,000,000,000+':
        if app[0] not in most_installed:
            most_installed.append(app[0])
print(most_installed)
```

    ['Messenger – Text and Video Chat for Free', 'WhatsApp Messenger', 'Google Chrome: Fast & Secure', 'Gmail', 'Hangouts', 'Skype - free IM & video calls', 'Facebook', 'Instagram', 'Google+', 'Google Drive']


The apps with the most installs include social networking giants like Facebook and Instagram and a handful of Google apps. As is the case with the iOS data, it would be difficult to design a competitive app that simply replicates the functions of any one of the apps in the above list. Knowing which apps have the most users can nonetheless inform our recommendation for a potentially competitive app profile. 

## Conclusion: App Profile Recommendation 
Both iOS users and Android users like to use apps with a social/communicative component, and productivity apps like Google Drive are also highly popular among Android users. To make our iOS app idea even more competitive among Android users, we may want to include a feature that allows users to organize shared content by type of media (e.g. books, music, podcasts), genre, and other categories users may find helpful. In each of the media type domains, there can be an editable "to read" or "to listen" list, wherein users can bookmark media they want to engage with based on other users' recommendations. These feature may cater well to productivity-oriented users who like to stay organized and create/check off goals. For many users, being encouraged to discuss and reflect on media they've interacted with with their friends may enhance the social and educational value they gain from the app.
