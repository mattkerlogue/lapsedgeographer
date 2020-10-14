---
title: "Virtual Gongoozling"
author: Matt
date: "2020-10-14"
slug: virtual-gongoozling
draft: true
tags:
  - r
  - flickr
  - twitter
  - github-actions
---

> *gongoozler* [n]  a person who enjoys watching boats and activities on canals  

I’m sure I’m not the first person, and won’t be the last, to remark that 2020 is a very strange year. October 2020 marks five years since I last went on a canal boat holiday. An anniversary that at the outset of this year I had hoped I might have managed to avoid by taking to the water sometime over the summer. So, inspired by Matt Dray's [recent adventures](https://www.rostrum.blog/2020/09/21/londonmapbot/) in location-based Twitter-botting, I wondered whether it might be possible to make a Twitter bot that showcased features of the British canal network. TLDR: let me introduce you to [narrowbotR](https://twitter.com/narrowbotR) … see what I did there, an R based twitter bot that does things narrowboats do.

{{< addToC >}}

## How it works
Based on the principles of Matt's *londonmapbot*, narrowbotR uses Github Actions to do the following:
* Select a random location on the English and Welsh canal network
* Get a photo for the location
* Post a tweet

Matt’s bot is a lightweight bot that picks a random location within an effective bounding-box (by selecting random numbers between the east-west and north-south extents), grabs an image from the Mapbox API, and then constructs and posts a tweet via the {rtweet} package which includes a link to the location in OpenStreetMap. This process is encapsulated in a single script, which is then executed every 30 minutes by a Githhub Action YAML workflow.

narrowbotR extends the programming in a number of ways:
1. It builds and samples a dataset of  geospatial features on the canal network.
2. It access the Flickr API as well as the MapBox API.
3. It constructs a rich status message and posts this via a customised version of the `rtweet::post_tweet()` function to embed the location in the tweet’s metadata.
4. Its GitHub Action operates only between 7am and 9pm because responsible boaters don't cruise after night.

## Building a database
As Ken Butler [points out](http://ritsokiguess.site/docs/2020/10/10/sampling-from-locations-in-a-city/), londonmapbot benefits from the fact that London occupies a relatively rectangular space that is relatively neatly aligned against the east-west and north-south axes of most coordinate systems. Matt helps this by saying that londonmapbot’s parameters are to find locations within the M25 which is rather more rectangular than the boundary of the Greater London region.

The UK’s inland waterways system doesn’t exist in such a neat set of geographical parameters, moreover it doesn’t exist as a single system. There are three major navigation authorities — the Canal and River Trust (for England and Wales, and itself a relatively recent merger formed of British Waterways and part of the Environment Agency), Scottish Canals (for Scotland), and Waterways Ireland (covering both Northern Ireland and the Republic of Ireland). But there also exist separate navigation authorities for specific watercourses/sets of waterways that aren’t covered by these bodies, the largest of these being the Broads Authority in Norfolk. To keep things simple, at the moment I’ve limited narrowbotR to working within the confines of the Canal and River Trust (CRT) network.

Thankfully, CRT publish a large amount of open geodata about their network, which made the process of building a database fairly easy. Different types of feature exist in their own GeoJSON feed, but it was fairly simple enough to create a simple csv listing these feeds, and then using `{purrr}` to run through these, download the data and combine it.

**INSERT CHUNK**
