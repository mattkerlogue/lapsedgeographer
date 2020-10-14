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

I’m sure I’m not the first person, and won’t be the last, to remark that 2020 is a very strange year. October 2020 marks five years since I last went on a canal boat holiday. An anniversary that at the outset of this year I had hoped I might have managed to avoid by taking to the water sometime over the summer. So, inspired by Matt Dray's [recent adventures](https://www.rostrum.blog/2020/09/21/londonmapbot/) in location-based Twitter-botting, I wondered whether it might be possible to make a Twitter bot that showcased features of the British canal network. So let me introduce you to [`narrowbotR`](https://twitter.com/narrowbotR) … see what I did there, an R based twitter bot that does things narrowboats do.

{{< addToC >}}

## How it works
Based on the principles of Matt's *`londonmapbot`*, `narrowbotR` uses Github Actions to do the following:
* Select a random location on the English and Welsh canal network
* Get a photo for the location
* Post a tweet

Matt’s bot is a lightweight bot that picks a random location within an effective bounding-box (by selecting random numbers between the east-west and north-south extents), grabs an image from the Mapbox API, and then constructs and posts a tweet via the {rtweet} package which includes a link to the location in OpenStreetMap. This process is encapsulated in a single script, which is then executed every 30 minutes by a Githhub Action YAML workflow.

`narrowbotR` extends the programming in a number of ways:
1. It builds and samples a dataset of  geospatial features on the canal network.
2. It access the Flickr API as well as the MapBox API.
3. It constructs a rich status message and posts this via a customised version of the `rtweet::post_tweet()` function to embed the location in the tweet’s metadata.
4. The GitHub Action operates only between 7am and 9pm because responsible boaters don't cruise after night.

Matt's bot runs from a single script, but in order to do all of the above narrowbotR is a collection of functions and a runner. The repo's structure is also follows:

```
narrowbotR
  │
  ├ .github/workflows
  │  ├ R_check.yaml.       # A manual workflow to check R on Github Actions
  │  ├ bot.yaml.           # The workflow controlling the scheduled bot
  │  └ bot_manual.yaml     # The bot workflow but with manual execution
  │
  ├ data
  │  ├ all_points.RDS      # A dataset of all the CRT point data
  │  ├ features.csv        # Metadata about types of CRT features
  │  └ feeds.csv           # URLs to the feeds for CRT geodata
  │
  ├ R
  │  ├ build_database.R    # A script to build data/all_points.RDS
  │  ├ flickr_functions.R  # Functions to interact with the Flickr API
  │  ├ pkg_install.R       # Functions to install packages on Github Actions
  │  └ post_geo.R          # A function to extent rtweet::post_tweet
  │
  ├ LICENCE                # An MIT licence for the code
  ├ narrowbot.R            # The bot workflow script
  └ README.md              # The repo README
  
```

`bot.yaml` runs the bot to a schedule and calls `narrowbot.R` to do the legwork, I'll discuss these towards the end of the blog, first I'll run through the main steps: building a database, accessing the Flickr API, and posting a geo-aware tweet.

## Building a database
As Ken Butler [points out](http://ritsokiguess.site/docs/2020/10/10/sampling-from-locations-in-a-city/), `londonmapbot` benefits from the fact that London occupies a relatively rectangular space that is relatively neatly aligned against the east-west and north-south axes of most coordinate systems. Matt helps this by saying that `londonmapbot`’s parameters are to find locations within the M25 which is rather more rectangular than the boundary of the Greater London region.

The UK’s inland waterways system doesn’t exist in such a neat set of geographical parameters, moreover it doesn’t exist as a single system. There are three major navigation authorities — the Canal and River Trust (for England and Wales, and itself a relatively recent merger formed of British Waterways and part of the Environment Agency), Scottish Canals (for Scotland), and Waterways Ireland (covering both Northern Ireland and the Republic of Ireland). But there also exist separate navigation authorities for specific watercourses/sets of waterways that aren’t covered by these bodies, the largest of these being the Broads Authority in Norfolk. To keep things simple, at the moment I’ve limited narrowbotR to working within the confines of the Canal and River Trust (CRT) network.

Thankfully, CRT publish a large amount of open geodata about their network, which made the process of building a database fairly easy. Different types of feature exist in their own GeoJSON feed, but it was fairly simple enough to create a simple csv listing these feeds, and then using `{purrr}` to run through these, download the data and combine it.

```r
feeds <- readr::read_csv("data/feeds.csv")

points <- feeds %>%
  dplyr::filter(geo_type == "point") %>%
  dplyr::mutate(data = purrr::map(feed_url, sf::st_read))
```

We also need to clean the file a little:
* the data includes points, polygons and lines, at this stage let's just concern ourselves with the point data;
* some features include an angle but this is stored in variables with different capitalisation, so let's unify that;
* let's extract the latitude and longitude positions of the point from the geometry;
* let's add some metadata about the features that we've created in a separate csv;
* finally, let's rename some of the fields to give them a slightly clearer name.

```r
features <- readr::read_csv("data/features.csv")

points_combined <- points %>%
  tidyr::unnest(data) %>%
  dplyr::mutate(angle = dplyr::case_when(
    !is.na(Angle) ~ as.double(Angle),
    !is.na(ANGLE) ~ as.double(ANGLE),
    TRUE ~ NA_real_
  )) %>%
  bind_cols(sf::st_coordinates(.$geometry) %>% 
              tibble::as_tibble() %>% 
              purrr::set_names(c("long", "lat"))) %>%
  dplyr::left_join(features, by = c("feature", "SAP_OBJECT_TYPE")) %>%
  dplyr::select(-Angle, -ANGLE, -OBJECTID) %>%
  janitor::clean_names() %>%
  dplyr::rename(uid = sap_func_loc,
         name = sap_description,
         obj_type = sap_object_type)

readr::write_rds(points_combined,"data/all_points.RDS", compress = "bz2")
```

At present this dataset is created by manually running this script. We could create this everytime narrowbotR but that is definite processing overkill as the data doesn't appear to be updated that regularly. In future I plan to write a script and Github Action to check on an occasional interval whether any of the CRT feeds have been updated and at that point issue a call to rebuild the dataset.

## Selecting Flickr photos
`londonmapbot` pulls an aerial photograph from the Mapbox API, but sometimes canal features are difficult to identify from above and/or can be things of interest to photographers. Flickr is an online photo sharing service, yes younglings we were able to share photos before Instagram, and has an API that allows you to access photos shared on the service. Photographers are able to geotag their photos, and using the Flickr API we're able to search for photos based on their location. There are some packages that have tried to provide access in R to the Flickr API but they aren't well maintained. Thankfully for the work `narrowbotR` does there's no need for a complex OAuth authentication proccess and the standard API access key is all that's needed. To get a Flickr API key you just need to [sign up to create an app](https://www.flickr.com/services/apps/create/).

To get the photos for a location we'll use the [`flickr.photos.search` API](https://www.flickr.com/services/api/flickr.photos.search.html), let's create the `flickr_get_photo_list()` function to handle this.

```r
flickr_get_photo_list <- function(key = NULL, lat, long) {
  
  if (is.null(key)) {
    key <- Sys.getenv("FLICKR_API_KEY")
    if (key == "") {
      stop("Flickr API key not set")
    }
  }
  
  if (missing(lat)) {
    stop("lat missing")
  }
  
  if (missing(long)) {
    stop("long missing")
  }
  
  # construct flickr api url
  # using lat/long will sort in geographical proximity
  # choose within 100m of the position (radius=0.1)
  url <- paste0(
    "https://www.flickr.com/services/rest/?method=flickr.photos.search",
    "&api_key=", key,
    "&license=1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%2C9%2C10",
    "&privacy_filter=1",
    "&safe_search=1",
    "&content_type=1",
    "&media=photos",
    "&lat=", lat,
    "&lon=", long,
    "&radius=0.1",
    "&per_page=100",
    "&page=1",
    "&format=json",
    "&nojsoncallback=1"
  )
  
  # get data
  r <- jsonlite::fromJSON(url)
  
  # extract the photo data
  p <- r$photos$photo
  
  ...
 
 }
```

First the function will check if an API `key` is provided and if not it'll look and see if its been set as an environment variable, if no key is available the function will throw an error. Similarly if no `lat` or `long` are provided let's throw an error. To avoid [this error](https://twitter.com/narrowbotR/status/1314551168283144195?s=20) during testing we could put in place some limits for the `lat` and `long` to check they're within the extent of the UK but let's not overcomplicate, and make it easy for folk to port the code.

Next let's construct our call to the `flickr.photos.search` API - this is in the format of a URL string with a series of arguments. We must supply the `api.key` argument, in addition to the `lat` and `lon` arguments, let's refine the call a bit further. The `licence` argument selects only those photos that are licenced for re-use. We also set the `privacy_filter` and `safe_search` to make sure we select photos suitable for broad public consumption. The `content_type` and `media` arguments ensure we just get photos, not screenshots or videos. In addition to the `lat` and `lon` arguments we can also set a `radius` (in km) from the point, in this case we're only going to ask for photos within 100m (0.1km) of the location; when `lat` and `lon` are specified the search will return photos in ascending order of proximity to the location (i.e. the first result will be the closest to the specified point). In the event of picking a very popular location, the `per_page` and `page` arguments ensure we only get the first 100 photos. Finally the `format` and `nojsoncallback` arguments are required for getting the data in JSON format.

Now we've constructed our URL we can get the data by calling on `jsonlite::fromJSON()`, which returns a list object within which there is a table of the photo data that we can extract.

Let's continue building our function by working with this table (the object `p`).

```r
flickr_get_photo_list <- function(key = NULL, lat, long) {
  
  ...
  
  # skip if less than 10 photos returned
  # suggests uninteresting/remote place
  if (length(p) < 10) {
    p <- NULL
  } else {
    
    # get info for photos, add sunlight hours
    # drop photos before sunrise or after sunset
    p <- p %>% 
      dplyr::mutate(
        info = purrr::map2(id, secret, 
                           ~flickr_get_photo_info(key = key, 
                                                  photo_id = .x, 
                                                  photo_secret = .y))
      ) %>%
      tidyr::unnest(info) %>%
      dplyr::mutate(
        suntimes = purrr::map(
          as.Date(date), 
          ~suncalc::getSunlightTimes(date = .x, lat = lat, lon = long, 
                                     keep = c("sunrise", "goldenHourEnd", 
                                              "goldenHour", "sunset"))),
        suntimes = purrr::map(suntimes, ~dplyr::select(.x, -date, -lat, -lon))
      ) %>% 
      tidyr::unnest(suntimes) %>%
      dplyr::mutate(after_sunset = date > sunset, 
                    before_sunrise = date < sunrise, 
                    goldenhour = dplyr::if_else(
                      (date >= sunrise & date <= goldenHourEnd) | 
                        (date <= sunset & date >= goldenHour), TRUE, FALSE)) %>% 
      dplyr::filter(!after_sunset) %>%
      dplyr::filter(!before_sunrise)
    
  }
  
  return(p)
  
}
```

If there are less than 10 photos we've probably selected somewhere fairly remote and/or uninteresting, one of my first tests was a hidden culvert for a sewer that was close to a Morrisons' car park which returned only three photos all of which were of the car park, so in this case let's just set `p` as `NULL` and be done with it. But if we get more than 10 photos let's get some more data about the photos, using another custom function (`flickr_get_photo_info`, of which more in a second) this will give us information including the date and time the photo was taken. Using the date as well as the latitude and longitude of the position we can use the `{suncalc}` package to identify sunrise and sunset times for the location and exclude photos taken at night, we can also calculate if the photo was taken during ["golden hour"](https://en.wikipedia.org/wiki/Golden_hour_(photography)) which is a prime time for photography.

The `flickr_get_photo_info()` function access the `flickr.photos.getInfo` API to access further data about specific photos using the `photo_id` and `photo_secret` data returned by our call to the search API.

```r
flickr_get_photo_info <- function(key = NULL, photo_id, photo_secret) {
  
  if (is.null(key)) {
    key <- Sys.getenv("FLICKR_API_KEY")
    if (key == "") {
      stop("Flickr API key not set")
    }
  }
  
  if (missing(photo_id)) {
    stop("photo_id missing")
  }
  
  if (missing(photo_secret)) {
    stop("photo_secret missing")
  }
  
  # create flickr api url
  url <- paste0(
    "https://www.flickr.com/services/rest/?method=flickr.photos.getInfo",
    "&api_key=", key,
    "&photo_id=", photo_id,
    "&secret=", photo_secret,
    "&format=json",
    "&nojsoncallback=1")
  
  # get data
  r <- jsonlite::fromJSON(url)
  
  # extract relevant info
  info <- tibble::tibble(
    username = r$photo$owner$username,
    realname = r$photo$owner$realname,
    licence = r$photo$license,
    description = r$photo$description$`_content`,
    date = lubridate::as_datetime(r$photo$dates$taken),
    can_download = r$photo$usage$candownload,
    can_share = r$photo$usage$canshare
  )
  
  return(info)
  
}
```

As with the `flickr_get_photos_list()` function, we'll check for an API key and that the other arguments aren't missing. Our API call this time is much smaller, needing just the `api_key`, `photo_id` and `secret` to get a dataset returned to us. We don't need all the data provided, so let's build a small tibble of the data we need.

