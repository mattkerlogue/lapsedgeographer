---
title: "Beep... boop... boot, booting the narrowbotr off Twitter"
author: Matt
date: "2023-04-15"
cover: "img/post/2023-04-15-towpath-closed-elliotbrown.jpg"
caption: "Towpath closed by Elliot Brown (ell-r-brown) on Flickr (CC BY-SA-2.0)"
slug: beep-boop-boot
tags:
  - r
  - flickr
  - twitter
  - mastodon
  - narrowbotr
---

Back in November after the various developments with Twitter, I
[worked out]({{< relref "2022-11-14-mastodon-bot-swtich.md" >}})
how to get my [Twitter bot](http://twitter.com/narrowbotR) to run on
[Mastodon](http://botsin.space/@narrowbotr). Since the Twitter takeover there
have been various announcements from the new owner and official Twitter
accounts about free access to the Twitter API being suspended, although these
have usually come and gone without said suspension happening.

<!--more--> Yesterday, however, I received an email from Twitter
informing me the narrowbotr's API access had now been suspended &ndash;
surprise, it hadn't, the bot was still posting away. Twitter has also released
details of its new API access. This includes a free tier, that provides
write-only (i.e. posting) of up to 1,500 tweets per month.

While the narrowbotr would be fine with this as it tweets about 4 times a day,
I've decided instead to stop the bot posting to Twitter, and it now only posts
to its Mastodon account. It's got nearly as many followers on Mastodon as it
has on Twitter, and its not like either of those counts is at all
influential[^1]. The Twitter debacle has seen my personal engagement with
Twitter wane, though I've gone in fits and starts with using Mastodon[^2].
I suspect mainly influenced by personal goings on than wider machinations in
social media[^3]. However, I can't really be bothered tailoring the bot to the
latest whims of an egotistical billionaire, so I'm proactively stopping it
posting to Twitter.

## Hashtags

I need to write a proper post about refactoring of the bot as some general
refactoring happened around the same time as the migration to Mastodon. But
in moving solely to Mastodon I've implemented a further couple of changes to
add hashtags to the post.

There are some standard hashtags for all posts, such as `#canal`, but I've also
added a flag to the dataset for whether the location is in Wales or not to add
either a `#wales` or `#england` hashtag. When pulling a photo from Flickr the
API also provides information on the tags that users have given to the photo
(if any), these are now recycled and added to the message posted to Mastodon.

[^1]: 18 Mastodon followers, compared to 28 Twitter followers for those
interested.

[^2]: I've an account on (Fosstodon)[https://fosstodon.org/@mattkerlogue] if
you want to follow me.

[^3]: Though the fact that I follow a weird mix of folk making for a stream
that is at sometimes genuinely interesting, occasionally uplifting, but
largely (and increasingly) depressing (especially in regards LGBT matters)
isn't helping.
