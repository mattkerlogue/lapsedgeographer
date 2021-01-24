---
title: "Planning our Twitter walk"
date: 2021-01-10
slug: tweet-walk-planning
cover: "img/post/2021-01-10-dungeness_pebbles.jpg"
caption: "The pebble beach at Dungeness"
tags:
  - r
  - geography
  - spatial
  - sp
  - coastalwalkr
  - wip
  - twitter
  - mapbot
  - walking
---

Let's just acknowledge upfront that this is a [very](https://www.theguardian.com/us-news/2021/jan/08/donald-trump-twitter-ban-suspended) [strange](https://twitter.com/TwitterSafety/status/1347684877634838528) [week](https://www.politico.com/news/2021/01/08/trump-reacts-to-twitter-ban-456785) to be writing about Twitter posting strategies[^1].

This is the second blog about my coastalwalkr project, you can read the first [here]({{< relref "2021-01-03-lets-go-for-a-walk.md">}}). The last post introduced a conceptual evolutionary jump for the mapbotverse — walking — and we've seen how in R we might be able to programmatically take a walk along the coast of Great Britain. For humans the walk in and of itself can be the sole purpose of the activity, but a mapbot probably needs something more to keep itself occupied and entertained? Most folk when heading out on a walk will plan their route in advance, and the coast can definitely be somewhere where a route plan is a [good idea](https://metro.co.uk/2014/03/17/four-people-and-dog-rescued-from-british-beach-after-getting-stuck-in-quicksand-4622816/)[^2]. So we should be responsible mapbot walkers[^3], and come up with a plan for our walk[^4]. This post explores what a strategy for tweeting along our mapbot's walk might include based on potential data/sources that we might use.

#### Natural mapbot *bay*-sics
Naturally there are some basic features of mapbots that we can work with for starters: standard street maps and aerial photography. The other mapbots rely on the aerial photography from Mapbox, and link to OpenStreetMap. We can also use Mapbox to obtain street mapping imagery to insert into a tweet. But if you're a proper walker you've probably got some Ordnance Survey maps, and luckily for the mapbots there is also now a wide range of [Ordnance Survey data](https://www.ordnancesurvey.co.uk/business-government/products?Product%20type=0%2F154%2F173%2F175) that is being published under open licences. So let's clock that as something to work with in a tweet or two.

#### *Train*ing the mapbot
Like the other mapbots this bot will start at a random location. Unless we're lucky it's likely that this mapbot won't be close to home[^5]. Thankfully, the Department for Transport, in collaboration with local authorities and the devolved administrations, publishes a large quantity of open data about public transport in the UK through something called the National Public Transport Access Nodes (NaPTAN) and National Public Transport Gazeteer (NPTG). Through the [NaPTAN/NPTG download service](https://naptan.app.dft.gov.uk/frmLogin) you can download data about all public transport stations and stops in Great Britain. Beware though, as I'll discuss in a later post, this data isn't as clean as we might hope it would/could be - prepare yourself for a fun/not-fun post about bus stop naming conventions (or the lack thereof).

#### Photography
In addition to aerial photography, the [narrobotR]({{< relref "2020-10-14-virtual-gongoozling.md">}}) demonstrated how we can access the Flickr API to get geo-tagged photos from a specific location. So that's an easy win.

#### Fancy a dip?
As we're by the coast why not go swimming?[^6]. But before we dip our feet in the water, should we check if the water is ok? The UK's various environmental bodies publish data about bathing quality - The Environment Agency [Bathing Water Quality API](https://environment.data.gov.uk/bwq/) (for England) and Natural Resources Wales [Bathing Water Quality Data API](https://environment.data.gov.uk/wales/bathing-waters/) both provide data for a several coastal sites each country using a similar API format. Sourcing data for Scotland was more tricky, raw data from the Scottish Environment Protection Agency appears to be [missing](https://www.environment.gov.scot/data/data-analysis/bathing-waters/), but some data is published on the web: (i) a table of location names and their rating [here](https://www2.sepa.org.uk/bathingwaters/Classifications.aspx); (ii) HTML profiles for individual locations [here](https://www2.sepa.org.uk/bathingwaters/Locations.aspx); and, (iii) more detailed PDF profiles [here](https://www2.sepa.org.uk/bathingwaters/Profiles.aspx). So we could scrape the Scottish data and create a dataset.

How about sea temperatures? There is historical [seawater temperature data](http://data.cefas.co.uk/#/View/19490) published by Cefas covering 27 sites across England. For current conditions, there are several sites, such as the snappily named [seatemperature.org](https://www.seatemperature.org/europe/united-kingdom/), which use data published by the US National Oceanic and Atmospheric Administration. NOAA publishes [a lot of data](https://www.ncdc.noaa.gov/data-access/model-data/model-datasets), which is great, but trying to just extract current sea-temperatures might be a bit excessive. The UK Met Office publishes [some](https://www.metoffice.gov.uk/about-us/legal/open-data-policy) open data through their [DataPoint](https://www.metoffice.gov.uk/services/data/datapoint) product but this doesn't appear to include sea temperature, but let's maybe come back to DataPoint another time.

#### What if we get in to trouble?
As I've already mentioned the coast can be dangerous, let's hope that we don't get into trouble if we go for a dip. But if we do, is there a lifeboat service nearby? The Royal National Lifeboat Institution (RNLI) is a charity that operates lifeboat and lifeguard services around the coasts of Great Britain, Ireland, the Isle of Man and Channel Islands. And, quite wonderfully they publish a huge amount of [open data](https://data-rnli.opendata.arcgis.com) about lifeboat stations, the lifeboat fleet and launches, and beaches with lifeguards. This is definitely something to explore and investigate.

#### A local walk for local people
So far we've concerned ourselves with finding a location, getting there and some of the local weather/climatic conditions. But what about the social geography of the place we're visiting? We can use the [NOMIS API](https://www.nomisweb.co.uk/) to access to a range of local statistics, for example population estimates, household composition, employment rate, regional income levels. For consistency across the different nations and the most up-to-date data[^7] the local authority district level is the lowest available geography. Using a local authority would also allow ability to link to authority website/twitter handle in the tweet.

And, if we're tweeting the local authority, perhaps we can also tweet local politicians?[^8] The UK Parliament's [open data service](https://explore.data.parliament.uk/?endpoint=members) provides data about MPs, including their twitter handles. Similarly the Scottish Parliament's [open data service](https://data.parliament.scot/#/datasets) also provides a variety of data about MSPs.

It is unclear if the Senedd's [open data services](https://senedd.wales/help/open-data/) includes details of MSs. It may be easier to collate a separate dataset by scraping the HTML pages/other sources.

#### A look across the water
All of the prior steps draw down data from other services, but this mapbot's native language is R, so maybe we should do something R-y[^9]. How about using `{sf}`[^10] to find nearest location not on Great Britain. Though for the west coast of Scotland this will largely be an island in the [Hebrides](https://en.wikipedia.org/wiki/Hebrides), so let's consider excluding points within the United Kingdom and finding the nearest non-UK location. Once we've identified the point we can use a [reverse geocoding service](https://en.wikipedia.org/wiki/Reverse_geocoding) to translate the longitude and latitude we've pinpointed to a human-readable place name.

#### *We've come a long way, but we're not too sure where we've been*
*We've had success* in identifying some information to include with our tweets along our walk. *We've had good times* exploring the data. *But remember this, been on this path of [coast] for so long, feel I've walked a thousand miles*, so we should maybe calculate whether it has been that far. *Sometimes strolled hand in hand with love*, love of R. *Everybody's been there*.

*With danger on my mind I would stand on the line of hope and I knew I could make it*, because we can find the nearest RNLI lifeboat to rescue us. *Once I knew the boundaries*, of our `{sf}` dataset, *I looked into the clouds and saw my face in the moonlight*, and the weather from the Met Office data.

*Just then I realised what a fool I could be, just 'cause I look so high you don't have to see me*... but we've not thought about the high-tide or tide times because I couldn't find open data for that. *Finding a paradise wasn't easy but still*, I think can find somewhere nearby across the sea. *There's a road going down the other side of this hill*, let's hope there's a bus stop or train station over the hill.

*Never forget where you've come here from*, let's use our final tweet to draw a trace of our walk. *Never pretend that it's all real, someday soon this will all be someone else's dream, this will be someone else's dream*, when I eventually write a GitHub Actions workflow so that it can run in the electric dreams of GitHub's virtual environment.

*Safe from the arms of disappointment for so long*, because GH Actions will notify me of any time the workflow fails. *Feel each day we've come too far*, perhaps when we select the points for our walk we can limit the distance. *Yet each day seems to make much more, sure is good is to be here*, so we'll make sure to tell the local council and MP.

*I understand the meaning of "I can't explain this feeling" now, and it feels so unreal*, perhaps because we've gone swimming in water that is too cold or picked up a nasty bug because the water wasn't very clean. *At night I see the hand that reminds me of the stand that I made, the fact of reality*, that this is quite a bit of work and so maybe the mapbot should only do one walk each day.

...

*We're not invincible, we're not invincible, no*

*We're only mapbots, we're only mapbots*

*Hey we're not invincible, we're not invincible*

*So again I tell you*

*Never forget where you've come here from*, by making sure you tweet the walk you've taken.[^11]


[^1]: Note to future-self, digital archaeologists/anthropologists from the year 3000[^1a], folks from other planets: (a) surely there are better things to read than this blog, if it still exists; (b) the President of the United States of America got banned from Twitter for inciting a rebellion against the US Congress.

[^1a]: Did we fix climate change, or do you live underwater? {{< youtube Tu7HoGZaspo >}}

[^2]: I was disappointed that Googling "funny lost walker stories" did not return anything worthy of comedy, "strange lost walker stories" largely gets you a lot of stories of mysterious and unexplained deaths. Do stay safe when out walking, make sure you prepare appropriately and try to avoid hazards, especially around coastal hazards (mudflats, marshes, cliffs, etc).

[^3]: {{<figure src="/img/post/2021-01-10-pirate_doggo_smol.png" alt="a small cute dog dressed as a pirate saying: don't walk the planl, walk me instead">}}

[^4]: Arguably also required because this is will be an automated non-sentient bot and therefore needs a clear workflow in order to run. But that's just a minor technical detail.

[^5]: Though being an island, nobody living in Great Britain is ever that far from the sea. The furthest you can get from the coast is *"just east of Church Flatts Farm, about a mile south-east of [Coton-in-the-Elms, Derbyshire](https://en.wikipedia.org/wiki/Coton_in_the_Elms)."* according to the [Ordnance Survey](https://www.ordnancesurvey.co.uk/blog/2012/01/where-is-the-centre-of-great-britain/), which is roughly 113 kilometres (or 70 miles) from the sea.

[^6]: While sea temperatures around Great Britain have been [rising](https://moat.cefas.co.uk/ocean-processes-and-climate/sea-temperature/), it's still fairly [cold](https://www.thebeachguide.co.uk/sea-temperature).

[^7]: Most sub-local authority data is based on the decennial census - the last one was in 2011, so a bit out of date... in fact, so out of date that a [new census](https://census.gov.uk) is almost upon us[^7a] in March this year.

[^7a]: Unless you live in Scotland, the Scottish Census has been [delayed to 2022](https://www.scotlandscensus.gov.uk/node/753) due to the COVID-19 pandemic.

[^8]: Hmm, politicians and Twitter, haven't we already covered that off in this post?

[^9]: **R-y** *(adj.)*: doing something in the R programming language. Pronounced "R.E.", just like you pronounced it when talking about your Religious Education lessons... which, I suppose is apt, I certainly have some degree of faith in R.

[^10]: How have we got this far into the blog without anything explicitly coding related.

[^11]: I'm sorry. So very sorry.[^11a] {{< youtube yoO_1FFr56k >}}

[^11a]: But you can rest safe in the kowledge that I'll be *Back for Good* with another post about the development of this mapbot.
