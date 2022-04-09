+++
title = "Visualize NYC 311 Call Stats via Zola"
description = "Play with chart.xkcd and vega-lite"
date = 2022-04-09

[taxonomies]
categories = ["Demo"]
tags = ["chart", "open data"]
+++

I want to explore rendering charts via static site generators. Here are a couple of libraries that are easy to integrate with [Zola](https://www.getzola.org/).

In this post, I plot open data from NYC 311 call records using:

* [chart.xkcd](https://timqian.com/chart.xkcd/) - a chart library that plots “sketchy”, “cartoony” or “hand-drawn” styled charts.

* [vega-lite](https://vega.github.io/vega-lite/) - a high-level grammar of interactive graphics.

Raw data is available [here](/data.csv), or viewed interactively via my test post on [DataTables](@/posts/test_table.md).

<!-- more -->

## [chart.xkcd](https://timqian.com/chart.xkcd/)

[chart.xkcd](https://timqian.com/chart.xkcd/) comes integrated with [deep-thought](https://github.com/RatanShreshtha/DeepThought) theme. It uses JSON to define axis and dataset, but does not come with any data loading APIs.

Since I don't want to write data wrangling code via javscript, I manually enter pre-aggregated complaint counts into JSON definition below.

Chart is generated via the `chart()` shortcode, which is defined in `themes/deep-thought/templates/shortcodes/chart.html`, and doesn't do much besides creating the the svg container and passing along the JSON definition.



__chart() short code__
```
<svg class="chart">{{body | safe}}</svg>
```


__call chart() shortcode with JSON definition to generate chart__
* note: escaped `{%` and `%}` due to Zola rendering issues
```
\{% chart() %\}
{
  "type": "Bar",
  "title": "Top 10 NYC 311 Complaint Types",
  "xLabel": "Complaint Type",
  "yLabel": "Count",
  "data": {
    "labels": ["Noise - Residential", "Heat/Hot Water", "Illegal Parking", "Street Condition", "Blocked Driveway", "Street Light Condition", "Plumbing", "Heating", "Water System", "Nosie - Street/Sidewalk"],
    "datasets": [
      {
        "data": [162, 129, 91, 86, 85, 71, 70, 70, 63, 63]
      }
    ]
  }
}
\{% end %\}
```

__output__
{% chart() %}
{
  "type": "Bar",
  "title": "Top 10 NYC 311 Complaint Types",
  "xLabel": "Complaint Type",
  "yLabel": "Count",
  "data": {
    "labels": ["Noise - Residential", "Heat/Hot Water", "Illegal Parking", "Street Condition", "Blocked Driveway", "Street Light Condition", "Plumbing", "Heating", "Water System", "Nosie - Street/Sidewalk"],
    "datasets": [
      {
        "data": [162, 129, 91, 86, 85, 71, 70, 70, 63, 63]
      }
    ]
  }
}
{% end %}

Overall, chart.xkcd is simple to use, looks nice, and tooltip works out of the box.

## [vega-lite](https://vega.github.io/vega-lite/)

It is fairly easy to integrate [vega-lite](https://vega.github.io/vega-lite/) into Zola.

First, override theme base template `base.html` to include vega-related javascript libraries in header. Zola support overriding either at file or block level. I hijacked a predefined block in the header section called `user_custom_stylesheet` which works just fine for my purpose.

__templates/base.html__
```
{% extends "deep-thought/templates/base.html" %}

{% block user_custom_stylesheet %}
{% if config.extra.vega_chart.enabled %}
<script src="https://cdn.jsdelivr.net/npm/vega@5.21.0" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-lite@5.2.0" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-embed@6.20.2" crossorigin="anonymous"></script>
{% endif %}
{% endblock %}
```

`vega_chart()` invokes custom shortcode defined in `templates/shortcodes/vega_chart.html`, which just creates a div container and passes along the javascript body. Chart is defined via vega-lite spec expressed in JSON format. Javascript code takes the JSON spec and calls [vega-embed](https://github.com/vega/vega-embed) api to compile and render the view.

__vega_chart() shortcode__
```
<div id="vis"></div>
<script>{{body | safe}}</script>
```

__call vega_chart() shortcode with Javascript body to render chart__
* Note: vega-lite spec is expressed in JSON format
```
\{% vega_chart() %\}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "data": {"url": "/data.csv"},
        "transform": [
          {
            "aggregate": [{
                "op": "count",
                "field": "Complaint Type",
                "as": "Complaint_Count",
            }],
            "groupby": ["Complaint Type"],

          },
        ],
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "Complaint Type",
                "type": "ordinal",
                "bin": false, 
                "sort": "-y"
            },
            "y": {
                "field": "Complaint_Count",
                "type": "quantitative",
                "sort": "-y"
            }
        }
    };


    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

\{% end %\}
```

{% vega_chart() %}

    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
        "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
        "data": {"url": "/data.csv"},
        "transform": [
          {
            "aggregate": [{
                "op": "count",
                "field": "Complaint Type",
                "as": "Complaint_Count",
            }],
            "groupby": ["Complaint Type"],

          },
        ],
        "mark": {
            "type": "bar",
            "tooltip": true
        },
        "encoding": {
            "x": {
                "field": "Complaint Type",
                "type": "ordinal",
                "bin": false, 
                "sort": "-y"
            },
            "y": {
                "field": "Complaint_Count",
                "type": "quantitative",
                "sort": "-y"
            }
        }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis', vlSpec).then(function(result) {
    // Access the Vega view instance (https://vega.github.io/vega/docs/api/view/) as result.view
    }).catch(console.error);

{% end %}


vega-lite offers vastly more flexibility in terms of data aggregation, sorting, filtering, and rendering options, but it's also harder to debug. For example, I notice that the x-axis labels are rotated 90 degrees automatically for readability.

Here's an attempt to filter for top complaint counts (> 50) by adding a __filter__ step in __transform__.
Doesn't work. Looks like filter is being applied to csv data stream instead of output of aggregate.

```
        "transform": [
          {
            "aggregate": [{
                "op": "count",
                "field": "Complaint Type",
                "as": "Complaint_Count",
            }],
            "groupby": ["Complaint Type"],

          },
          {
            "filter": [{
                "field": "Complaint_Count",
                "gt": 50
            }]
          }
        ],

```

