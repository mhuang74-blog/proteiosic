+++
title = "Google Ads Asset-based Ad Extensions"
description = "Quick look at adoption"
date = 2022-04-09

[taxonomies]
categories = ["PPC"]
tags = ["chart", "google ads", "zola"]
+++

In another [post](@/posts/quick_look_at_charts.md), I write about plotting charts within [Zola](https://www.getzola.org/). Here I apply it to examine adoption of Google Ads Asset-based Ad Extensions since start of 2022.

Raw data is available [here](/asset_ad_extensions_last_ytd.csv).

<!-- more -->


## [Google Ads Asset-based Ad Extensions Migration](https://developers.google.com/google-ads/api/docs/extensions/assets/migrating-extensions)

Google provides the [schedule](https://ads-developers.googleblog.com/2022/01/revised-schedule-for-auto-migration-of.html) for auto migration for each ad extension type. It would be interesting to see customer adoption and actual traffic patterns with respect to each of migration dates.

I use a tool called [mccfind](https://github.com/mhuang74/mcc-find-rs) (one of my Rust learning projects) to grab campaign-level traffic data via query below for about ~1000 Google Ads accounts, and aggregate them together by Date and Extension Type. Only the 4 major extention types (Sitelink, Callout, Call, and App) are included.

__GAQL to get YTD traffic on the 4 major Asset-based Ad Extensions__

```sql
# YTD Traffic from Asset-based Ad Extensions
campaign_asset_ad_extensions_ytd = """
	SELECT 
	  campaign.id,
	  segments.date,
	  asset_field_type_view.field_type, 
	  metrics.impressions, 
	  metrics.clicks, 
	  metrics.cost_micros,
	  customer.currency_code  
	FROM asset_field_type_view 
	WHERE 
	  segments.year IN (2022)
	  AND asset_field_type_view.field_type IN ('SITELINK', 'CALL', 'CALLOUT', 'MOBILE_APP') 
	  AND metrics.impressions > 1
	ORDER BY 
	  segments.date, campaign.id
```

Here's the first few rows of the raw csv data, with header names corresponding to [Google Ads GAQL](https://developers.google.com/google-ads/api/docs/query/overview) field names.

__a peek at raw csv data__
```
field_type,date,impressions,clicks,cost_micros
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

And here's the code to render line chart.

__vega-lite specs__
```json
\{% vega_chart() %\}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/asset_ad_extensions_last_ytd.csv"},
        "mark": {
            "type": "line",
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
                "field": "clicks",
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

__output__

{% vega_chart() %}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "width": 450,
        "height": 300,
        "data": {"url": "/asset_ad_extensions_last_ytd.csv"},
        "mark": {
            "type": "line",
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
                "field": "clicks",
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

## Summary

Looks like migration reakly started on Feb 9th, 2022, reaching first plateau around Feb 17th, and second plateau around March 9th.

Next steps:

* compare Asset-based traffic with Feeditem-based Ad Extension traffic to gauge percentage of traffic already migrated to Asset-based.