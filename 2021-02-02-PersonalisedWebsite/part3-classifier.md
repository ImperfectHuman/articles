# Personalised website 3: User data classification

In the previous parts in this series we've [given an overview](./part1-overview.md) of how we might build a personalised website, and looked at some [considerations for developing the stitcher component](./part2-stitcher.md)). In this final entry we're going to look at user data classification.

Let's start with recapping where our classifier sits within our architecture:

![Final version of diagram from part 1](./assets/004-PersonalisedStitcher.png)

The classifier takes user activity data that we have tracked, and metadata about the pages, to derive insights into user interests that are used by the stitcher to provide a personalised experience. The page metadata is needed to provide context to the user activity, and the sorts of analysis we want to perform with the classifier will dictate the nature of the metadata we need the publisher to provide.

## Metadata for user activity 

User activity events will only tell us that the user has done things like visiting certain pages, or spending a lot of time on a given page. We need to interpret that in the context of the page content to understand why they're interested. 

The most obvious sort of classification is by genre - was the page about sport? Which sport? This is useful for typical browsing experiences, where the user drills down through some sort of hierarchy to discover content that fits their mood. There's often an expectation for this sort of navigation, which makes it a good place to start for anything from a media website to the catalogue of an online shop.

We may eventually want to inspect the content in other ways. A music fan that's not normally interested in charity work might be interested in news about a favourite singer starting a charity; someone who doesn't like horror films might be interested in one made by their favourite director. Or perhaps the user likes the writing style of a given author, and are inclined to read their articles regardless of topic. All of this information can be collected as metadata about the page our user has visited, and fed into our classification process.

Those aren't the only ways of classifying our content. Is it cheerful or sad? Is it short or long? Is it a neutral tone, or a polemic call to action? Does it have graphs and tables of statistical analysis? Is it rumour and speculation or proven fact? Is it current or historical? Sarcastic and satirical? The list goes on.

We might be limited by our input. It's likely reasonable to have our content creators put in a couple of metadata tags about their content while publishing it, but unreasonable to ask them to add metadata covering every possible aspect of their content. At the start of our personalisation journey coarse classification, like genre and sub-genre, is likely enough to get us started. If we want to begin to do more nuanced classification later then we might be able to perform processing our articles for metadata - perhaps using [natural language processing](https://en.wikipedia.org/wiki/Natural_language_processing) approaches for identifying key people, or [sentiment analysis](https://en.wikipedia.org/wiki/Sentiment_analysis) to gauge tone. These automated approaches will have their problems to work through, but have the advantage of being possible to run across a back catalogue of unclassified content without needing an army of humans to review every entry and manually add a tag.

## Analysis

Having both a user activity report, showing which pages are being accessed, and a metadata report, showing what metadata those pages have, we can do some statistical analysis to classify our users. Counting page visits by genre and sub-genre we can calculate a scorecard of that user's activity. This could be a most-visited list of genres, information on the proportion of time spent in each sub-genre of a genre, or a simple interested/disinterested value associated with each. Each of these alternatives could be used for personalisation, controlling the order and presence of follow-on links on our site.

We might need to do this in relative terms. A heavy user of our site might have visited every genre ten times but some one hundred times, and a light user might have only visited one genre a couple of times. Both are expressing a genre preference in their activity, but in direct comparison one is barely interested in anything.

At some point it is likely we will need to adjust for recency; over time a user's interests may change. Perhaps an event in an area draws people to it that were previously not interested. Perhaps an interest simply wanes over time, being replaced by others. If we always use the complete history of a user's interests to provide personalised requests we won't be offering them the content they want today. We might adjust for this by having a fall-off pattern in the weighting we give data - this week's activity giving three times the weighting as the rest of the month, the month having twice the weighting of the rest of the year, and anything older than a year having just a tenth the rating of the current year. That would mean old patterns don't completely disappear, but we're guided by what the user is doing now. We could experiment to find out what fall-off pattern gave the best results.

Eventually we may want to do more advanced data analysis, such as trying to identify types of people that are using our site. We might use [cluster analysis](https://en.wikipedia.org/wiki/Cluster_analysis) techniques on our statistical data to try and develop stereotypes of our users. These can be useful in a variety of ways, from helping our marketing and design teams understand our audience, to allowing us determine content some users of a group enjoy that other users in that group are yet to experience. It might allow us to classify new users into one of these groups more quickly, allowing us to quickly improve their experience of the site and making it more likely we will get repeat business.

## Processing approach

When developing our analyses we will want to make sure they will scale as the number of users, content, and amount of historical data all increase, and to keep our results current. We want to avoid as much data reprocessing as possible to keep costs down, and to minimise the time taken to process it.

Processing the large quantity of data quickly might be done using a [MapReduce](https://en.wikipedia.org/wiki/MapReduce) approach. By dividing our data into chunks, processing those chunks in parallel on a temporary fleet of machines, then combining them back together, we can process a large amount of data in a small amount of time. Services like [Amazon EMR](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html) can help do this.

Even though we can process the data quickly we should consider how to limit the amount of work we need to do. We might be able to process only those users that have had activity since we last processed. Perhaps we can have several steps in our processing pipeline, with intermediate results stored so we don't have to go back and process all previous data. Because we'd be doing a lot of work with these intermediate result caches we might benefit from optimising our storage types and file formats to best support our data access patterns; we should consider using a variety of data storage services and data file formats (e.g. JSON, CSV, [Parquet](https://parquet.apache.org/)), and evaluate them in relation to our specific analyses.

## System development

There's a two-way feedback loop between the stitcher and the classifier. Based on the decisions we want to make in the stitcher we might need to introduce a new classification analysis, and based on something we notice from an analysis we might want to change the way the stitcher works. Both might require changes to other systems, like the publisher, and those interactions are two-way as well - if the publisher can start providing additional metadata that hasn't been considered before the stitcher and classifier might be able to make use of it.

The most important feedback is likely to be that from our customers. While developing and changing personalisation options we should be keenly monitoring our user activity. Are users spending more quality time engaging with content on our site? Are they spending time looking through lists of content, but finding nothing they want to read or watch? Are there even limits we don't want them to go beyond? Having users visit, binge on content, and never return might not be the best long-term scenario.

There are wider-reaching concerns as well. It's probably [not healthy](https://en.wikipedia.org/wiki/Digital_media_use_and_mental_health) for people to be plugged into our site all day every day, and we might have a social responsibility to try and discourage [excessive use](https://en.wikipedia.org/wiki/Internet_addiction_disorder). As an organisation we might want to balance our desire for high-consumption, which may be felt to mean high-revenue, against our ethical responsibilities and the public perception of our organisation.

## Final word

In this series I wanted to show how a system could evolve over time, guided by its business context. To demonstrate how our subsystems interlink and impact each other, and to show an incremental approach to development where our complex final product emerges over time.

We've looked at how there are multiple approaches to solving some problems, and noticed that some of these approaches might not be possible at first, but become possible later (e.g. as more data is gathered). This has sometimes led us to develop systems knowing that we might want to replace them with a different approach later.

Throughout we've been guided by some initial principals; we've particularly focused on a need to minimise cost, which has led us to approaches that will minimise (more expensive) repeat processing, even if that means using a greater amount of (cheaper) data storage. This has impacted our publishing process, customisation approach, and data analysis pipelines.

Most importantly we've thought beyond just the technology, and looked at the business context for the technology. We 
haven't tried to develop perfect systems immediately, but simpler ones that are appropriate for the needs of the business at the time, and can start to provide a return on our invested effort as soon as possible.

Thank you for joining me on this journey, I hope the experience has been helpful.
