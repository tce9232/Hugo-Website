---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Madison Landlord Lookup Tool"
summary: "Examining the rental landscape of Madison, Wisconsin"
authors: ['Ted Carlson']
tags: ['Web Scraping', 'MapBox']
categories: []
date: 2020-11-15T13:41:02-06:00

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "The Hub Apartment Complex on State Street in Madison"
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

Everyone who has changed leases in Madison Wisconsin, or even just been in town during the nights of July 31st or August 14th knows the horrors of Madison moving day. For those that haven't experienced this ritual, it forces thousands of predominantly students and younger tenents to move out a full day, or even up to a couple weeks before their new lease kicks in. Almost all Madison leases either start on August 1st or August 15th, but you're typically required to move out the previous day. This causes a huge battle for U-Hauls and parking spaces so residents can pack up all of their belongings and move to the place they're going to stay the night before their new lease begins. Some especially unlucky tenents will have to pay double rent for the first half of August, as their leases overlap, or even worse, not have any lease for that time period. For people that have lived in Madison as long as I have, this ritual might seem almost normal. But for those that are new students or new to the Madison renting schedule, this phenomenon makes no sense at all.



However, from a property management perspective, this ritual makes perfect sense. It balances out your leases so you only have to pay for maintenance and cleaning during one day, and since month-to-month leases aren't common, you don't have to worry about constantly finding new tenents. There's a very predictable yearly leasing pattern that allows you to properly market and sell your properties at the highest price possible. 

So what gives? Why are landlords allowed to force tenants to accommodate to their schedule, and why doesn't our state and local government push back? [Entire books](https://www.evictedbook.com/) can be written about how unfair the Wisconsin housing protections are. And Madison is still primarily a college town, which can lead to more predatory leases. But I wanted to take a closer look at the networks of Madison properties and their owners to see if I could visualize the extent to which the properties downtown and near campus are monopolized by just a handful of property management companies. Creating this visualization/tool could also lead to better tenant organization, as tenants would easily be able to see which addresses were owned by the same landlord.

![pic_name](/img/hippiechristmas.jpg)

### Data Scraping

The City of Madison Assessor's Office website allows you to search and obtain data for any commercial or residential property in Madison. By doing a bit of manual work and a bit of web scraping, I was able to obtain a link to every property in Madison that was owned by an entity that starts with a standard English alpha-numeric character. Another python script visited each page and scraped information like the address, owner name, property value, property type (Apartment, Condo, etc..) and property class (commercial, residential, etc..). Here's an example of the text parsing require to get a property's address:

```python
address = soup.find('span', class_='nav-heading')
    p = re.compile('Address[\s]+([\w ,&-+%#]+)')
```

### Data Cleaning

In order to properly clean the data, I needed to fill missing values with zeros and removing the dollar sign from the property values. For the purposes of this project, I combined both Condos and Apartments as "Apartments". This is because the owners of these condos according to the Assessor's website are often the owners of the building.

The most important aspect of the cleaning was finding a proper mapping for owner names. This is challenging because you'll often have the same owner with different names depending on the building. Some of these are typos, but most of them are a result of displaying additional information in the owner name, like the address or name of the property manager (who usually works for the property management company). This was the most labor intensive aspect of the project, and I couldn't find a better way to accomplish this except looking for key characters that would indicate something like this happening (# was often used before an address or property manager name) or searching for the biggest rental companies I knew in Madison and seeing if they had a consistent naming pattern.

Once this owner name cleaning was done, I was able to calculate the number of units from each apartment by parsing the building information using the following python code:

```python
p = re.compile('([\d]+) unit')
df['Units'] = df['property_type'].str.findall(p)
df['Units'] = pd.to_numeric(df['Units'].apply(lambda x: 1 if not x else x if isinstance(x, float) else x[0]))
```
I was then able to sum the total number of buildings, units, and property value for each owner name, which helped create aggregated statistics for each property owner across Madison.

At this point, I had to make a decision about which properties should be included in this dataset. I didn't want to include private personal properties, but I still wanted to make sure I was including houses that got split into floors and were leased out, as they constitute a majority of the units around campus. For these reasons, I included any apartments or single family homes that were either owned by somebody with multiple properties or had a value over $1 million. This isn't a perfect metric, as there are obviously private homes worth more than $1 million in Madison, but it was the only way I found to ensure the inclusion of large apartment complexes that were, for whatever reason, classified as single family homes.

Once I finalized the properties that would be included in the visualization, I converted each of their text addresses into long-lats using GeoPy in the following code:

```python
# 1 - conveneint function to delay between geocoding calls
geocode = RateLimiter(locator.geocode, min_delay_seconds=1)
landlordDF['exact_address'] = landlordDF['address'].apply(lambda x: str(x) + " Madison, Wisconsin USA")
# 2- - create location column
landlordDF['location'] = landlordDF['exact_address'].apply(geocode)
# 3 - create longitude, laatitude and altitude from location column (returns tuple)
landlordDF['point'] = landlordDF['location'].apply(lambda loc: tuple(loc.point) if loc else None)
# 4 - split point column into latitude, longitude and altitude columns
landlordDF[['latitude', 'longitude', 'altitude']] = pd.DataFrame(landlordDF['point'].tolist(), index=landlordDF.index)
```

This step is tricky, as you can easily max out on calls to GeoPy if you don't limit your call attempts. But after almost a full day of letting this code run, it allowed me to convert the data into a GeoJSON format, which is easily readable by Mapbox.

### Data Analysis

After gathering and aggregating this data, my first goal was to see which landlords owned the collective highest valued properties in Madison. Looking at the top ten landlords by property value, it becomes clear that owning a large apartment complex or two will add to a higher value than a large portfolio of homes or small apartment buildings. 

{{< load-plotly >}}
{{< plotly json="/plotly/Properties-Owner-Bar.json" height="600px" width="800px" >}}

Every owner except Forward Management, who owns a bunch of properties on the far East and West sides, owns 11 or fewer buildings. The landlord with the highest valued portfolio is Core Campus Madison LLC, who owns The Hub and The James, two gigantic luxury apartment complexes near downtown and campus. Since we noticed this trend of landlords with fewer properties owning the most property value, I wanted to see how this relationship looked across all landlords in my database. 

{{< plotly json="/plotly/Value-Properties-Scatter.json" height="600px" width="800px"  >}}

The plot seems to tell us that landlords with fewer properties tend to own more valuable ones, but landlords with more properties own relatively little in value. This seems to be because developers of large luxury apartments tend to just own a few properties, but manage hundreds of units. On the flip side, there are some landlords like Madtown Properties Inc and Madison Opera Inc that own large buildings of condos that are individually listed at very low prices. Madtown Properties Inc owns the condos near Hilldale Mall, which are valued at only $15,000 each, and Madison Opera Inc owns the condos on 23 N Brooms St, which are valued at $8,000 each. 

Perhaps a better visualization would be to look at a landlord's total values against their total number of rental units. I measured these by scraping any number that came after the word "Unit" in the building description, and if there was none, then just defaulting to 1. 

{{< plotly json="/plotly/Value-Units-Scatter.json" height="600px" width="800px"  >}}

This plot shows a much stronger positive correlation between units and property value. Core Campus ends up close to the top of the chart, over the top of the trend because of their luxury apartment value, and owners like Nob Hill Investments, with affordable apartments on the far south side end up on the lower end of the linear trend.

Visualizing this data helps us understand how the Madison rental property value is distributed among its landlords. But Madison is a city with many (relatively) unique neighborhoods, and in order to better understand these property management company's relationships with the community, we need to see where they own property in relation to campus, downtown and the suburbs. 

### Plotting with Mapbox
You can find the most recent version of the Mapbox visualization at the following link: [Madison Landlord Lookup](https://madison-landlords.herokuapp.com/)

The goal for my final visualization was to give the user some basic statistics for Madison properties both on the individual property and property owner levels. The constantly floating tooltip in the upper left corner gives the user these stats about whichever property was the last one they hovered over.

Since there are so many properties, I also found it important to let the user only focus on those that are of interest to them. Whenever you're hovering over a property, it's highlighted pink, as well as any other properties that are owned by the same landlord. There's also a toggle built in that lets you click on any individual property and remove any points on the map that aren't owned by the same landlord. This can all be reset by clicking the "reset map" button or any blank area.

Although I believe this map can be informative as is, I welcome any feedback or feature improvements to either the data consolidation, cleaning or visualization processes.

