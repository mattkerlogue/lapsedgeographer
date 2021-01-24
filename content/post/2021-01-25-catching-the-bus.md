---
title: "Catching the bus"
date: 2021-01-24
slug: catching-the-bus
cover: "img/post/2021-01-25-spice-bus.jpg"
draft: true
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
  - transport
  - publictransport
  - dft
  - naptan
---

The next phase of planning our walk along the coastline is working out how to get there? Thanks to the Department for Transport's NaPTAN service we can access a lot (a hell of a lot) of data about public transport in Great Britain. The data on train stations is pretty good quality... because train stations aren't in the habit of changing that often[^1]. Oh but you want to get the bus... ahhhaa, well let me treat you to a rather wonderful story about using NaPTAN to name bus stop locations. I lied, it's not a wonderful story, it's traumatic, and because I've been through that trauma now I have to share it with you.

{{< addTOC >}}

## Trains, planes and automobiles

The Department for Transport (DfT) publishes open data about Great Britain's public transport network through the National Public Transport Access Nodes (NaPTAN) and National Public Transport Gazetteer (NPTG) databases[^2]. While it's good these are published, I'd like to say before going further that the structure and supporting information for the data is a lot more confusing than it ought to be. That's both in terms of the fundamental structure and documentation, but also as we'll soon come to learn the data included within it. For example, take this instruction about how to find information for a specific local authority:

> To find the code for each local authority, see the NaPTAN Last Submissions page.[^3]

There's no canonical list of local authorities, you instead have to go to a page that lists the data submissions from local authorities and weed your way through that to get the codes.

[^1]: Something about ghost stations.

[^2]: The NaPTAN/NPTG schema information is on [GOV.UK](https://www.gov.uk/government/publications/national-public-transport-access-node-schema) though you'll sometimes end up at the DfT's [former website](http://naptan.dft.gov.uk/naptan/overview.htm). The documentation helpfully includes [diagrams of the database structure](http://naptan.dft.gov.uk/naptan/csvformat.htm), unhelpfully they're at a very low resolution so you can't actually read them. The NaPTAN/NPTG data is downloaded from the DfT's [NaPTAN/NPTG app](https://naptan.app.dft.gov.uk/).

[^3]: Advice on the [download options page](https://naptan.app.dft.gov.uk/datarequest/help) of the NaPTAN/NPTG app.