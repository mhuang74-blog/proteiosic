+++
title = "Google Ads Asset-based Ad Extensions"
description = "Quick look at adoption"
date = 2022-04-12

[taxonomies]
categories = ["PPC"]
tags = ["chart", "google ads", "zola"]
+++

In another [post](@/posts/quick_look_at_charts.md), I write about plotting charts within [Zola](https://www.getzola.org/). Here I apply it to examine adoption of Google Ads Asset-based Ad Extensions since start of 2022.

Raw data is available [here](/all_asset_ad_extensions_ytd_cleaned.csv).

<!-- more -->


## Getting Data

Google provides the [schedule](https://ads-developers.googleblog.com/2022/01/revised-schedule-for-auto-migration-of.html) for auto migration for each ad extension type. It would be interesting to see customer adoption and actual traffic patterns with respect to each of migration dates.

I use a tool called [mccfind](https://github.com/mhuang74/mcc-find-rs) (one of my Rust learning projects) to grab extension level daily traffic via below [GAQL](https://developers.google.com/google-ads/api/docs/query/overview). Query is applied across ~1000 Google Ads accounts and aggregated by (Date, Extension Type) and written to csv.

```
mccfind -q asset_field_type_view_ytd --groupby "asset_field_type_view.field_type" --groupby "segments.date"  -o all_asset_field_type_view_ytd.csv
```

__GAQL to get YTD traffic across all Asset-based Ad Extension Types__

```sql
	SELECT 
	  asset_field_type_view.field_type, 
	  segments.date,
	  metrics.impressions, 
	  metrics.clicks, 
	  metrics.cost_micros,
	  customer.currency_code  
	FROM asset_field_type_view 
	WHERE 
	  segments.year IN (2022)
	  AND metrics.impressions > 1
	ORDER BY 
	  asset_field_type_view.field_type, segments.date

```

Output file is cleaned up a bit to remove extra `"` quotation marks in date column and `.` periods from header (e.g. from `segments.date` to just `date`).

```
$ cat all_asset_field_type_view_ytd.csv | sed 's/[a-z_]*\.//g' | sed 's/"//g' > all_asset_field_type_view_ytd_cleaned.csv
```

Here's a peek at cleaned up csv file. Please note that cost column isn't quite right since it's aggregated without currency conversion.

__a peek of all_asset_field_type_view_ytd_cleaned.csv__
```
field_type,date,impressions_sum_sum,clicks_sum_sum,cost_micros_sum_sum
Call,2022-02-11,63,2,139000000
Call,2022-02-12,63,4,242000000
Call,2022-02-14,95,7,301000000
Call,2022-02-15,93,5,326000000
Call,2022-02-16,404,8,196000000
Call,2022-02-17,443,13,460000000
Call,2022-02-18,454,10,274000000
Call,2022-02-19,467,7,177000000
Call,2022-02-20,355,7,171000000

```

## Traffic Trend by Extension Type

Ad Extensions are extra advertisement material that appear next to Ads. They help reinforce branding or product niche, but may not be clicked on. Hence, lets look at daily total _impressions_ from the 4 major Asset-based Ad Extensions as opposed to _click_ or _cost_. 


Here's the code to visualize daily impressions trend.

__vega-lite specs__
```json
\{% vega_chart(id="vis") %\}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/all_asset_field_type_view_ytd_cleaned.csv"},
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "date",
                "type": "temporal",
                "bin": false, 
                "sort": "x"
            },
            "y": {
                "field": "impressions_sum_sum",
                "type": "quantitative",
            },
            "color": {"field": "field_type", "type": "nominal"},
        }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

\{% end %\}
```

__YTD Daily Impressions by Extension Type__

{% vega_chart(id="vis") %}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/all_asset_field_type_view_ytd_cleaned.csv"},
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "date",
                "type": "temporal",
                "bin": false, 
                "sort": "x"
            },
            "y": {
                "field": "impressions_sum_sum",
                "type": "quantitative",
            },
            "color": {"field": "field_type", "type": "nominal"},
        }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

{% end %}

From the plot, it's clear that most of the traffic comes from Callout and Sitelinks Extensions. Structured Snippets has a minor share and rest of the extension types don't even show up.

In terms of migration dates, compared to original announced migration date of Jan 10th, 2022, traffic didn't start to increase significantly until after Feb 9th, reaching a plateau around Feb 17th.

## Zoom in on Sitelinks Migration

To see effects of migration, lets plot YTD Daily Impressions from both Asset-based and Feeditem-based Sitelinks.

First step is to download Feeditem-based traffic via this GAQL.

```
mccfind -q feed_placeholder_view_ytd --groupby "feed_placeholder_view.placeholder_type" --groupby "segments.date"  -o all_feed_placeholder_view_ytd.csv
```

__GAQL to download Feed-based traffic for all Ad Extensions__
```
	SELECT
	  feed_placeholder_view.placeholder_type,
	  segments.date,
	  metrics.impressions,
	  metrics.clicks,
	  metrics.cost_micros,
	  customer.currency_code
	FROM feed_placeholder_view
	WHERE
	  segments.year IN (2022) 
	  AND metrics.impressions > 1
	ORDER BY 
	  feed_placeholder_view.placeholder_type, segments.date
```

To combine Feeditem and Asset Sitelinks traffic into a single plot, the two csv files need to be combined, with new _type_ column indicating the type of Sitelink.


```
$ echo "type,date,impressions,clicks,cost_micros,currency" > combined_sitelinks_ytd.csv
$ grep Sitelink all_feed_placeholder_view_ytd.csv | sed 's/Sitelink/SL_Feed/g' | sed 's/"//g' >> combined_sitelinks_ytd.csv
$ grep Sitelink all_asset_field_type_view_ytd.csv | sed 's/Sitelink/SL_Asset/g' | sed 's/"//g' >> combined_sitelinks_ytd.csv   
```

To create a second plot on same page, we use `vis2` as the container name instead.

```
\{% vega_chart(id="vis2") %\}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/combined_sitelinks_ytd.csv"},
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "date",
                "type": "temporal",
                "bin": false, 
                "sort": "x"
            },
            "y": {
                "field": "impressions",
                "type": "quantitative",
            },
            "color": {"field": "type", "type": "nominal"},
        }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis2', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

\{% end %\}
```

__YTD Daily Impressions for Sitelinks__

{% vega_chart(id="vis2") %}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/combined_sitelinks_ytd.csv"},
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "date",
                "type": "temporal",
                "bin": false, 
                "sort": "x"
            },
            "y": {
                "field": "impressions",
                "type": "quantitative",
            },
            "color": {"field": "type", "type": "nominal"},
        }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis2', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

{% end %}

## Summary

Visualizing the daily impression changes across Asset-based Ad Extension migration reveals a number of trends:

* some customers had Asset-based Sitelinks impressions since Jan 1st, 2002 to experiement with the new format
* bulk of Sitelinks migration occured around Feb 17th, 2002
* As of Apr 12, ~10% of Sitelinks are still in Feeditem form and trafficking
* there appears to be downward trend of Sitelinks daily impresisons starting end of March, a month after they are migrated to Asset-based


## Next steps:

* compare Asset-based traffic with Feeditem-based Ad Extension traffic to gauge percentage of traffic already migrated to Asset-based.
** can't limit to self-clicks since Ads doesn't provide that for Asset-based

* generate Feeditem vs Asset across all ad extensions, and use filter to create separate plots

```
        "transform": [
            {"filter": {"field": "field_type", "equal": "Sitelink"}}
        ],
```