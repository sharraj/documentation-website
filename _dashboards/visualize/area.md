---
layout: default
title: Using area charts
parent: Visualization types
grand_parent: Visualize
nav_order: 5
---

# Using area charts

An area chart is a line chart with the area between the line and the axis shaded with a color, and is a primary visualization type used to display time series data. You can create area charts in Dashboards using the Area visualization type or using the Time Series Visual Builder (TSVB), Vega, or VisBuilder visualization tools. For this tutorial, you'll use the Area visualization type.

![Demonstration of the area chart tutorial steps]({{site.url}}{{site.baseurl}}/images/dashboards/area-tutorial.gif)

# Try it: Create a simple aggregation-based area chart

In this tutorial you'll create a simple area chart using sample data and aggregations in OpenSearch Dashboards by connecting to [http://localhost:5601](http://localhost:5601) from a browser.

You have several aggregation options in Dashboards, and the choice influences your analysis. The use cases for aggregations vary from analyzing data in real time to using Dashboards to create a visualization dashboard. If you need an overview of aggregations in OpenSearch, see [Aggregations]({{site.url}}{{site.baseurl}}/opensearch/aggregations/) before starting this tutorial.

Make sure you have [installed the latest version of Dashboards](https://opensearch.org/docs/latest/install-and-configure/install-dashboards/index/) and added the sample data before continuing with this tutorial. _This tutorial uses Dashboards version 2.4.1_.
{: .note}

## Set up the area chart

1. Access Dashboards by connecting to [http://localhost:5601](http://localhost:5601) from a browser.
1. Select **Visualize** from the menu and then select **Create visualization**.
1. Select **Area** from the window.
1. Select **opensearch_dashboards_sample_data_flights** in the **New Area/Choose a source** window.
1. Select the calendar icon and set the time filter to **Last 7 days**.
1. Select **Update**.

## Add aggregations to the area chart

Continuing with the area chart created in the preceding steps, you'll create an area chart that displays the top five logs for flights delayed for every three hours over the last seven days:

1. Add a **Metrics** aggregation.
   1. Under **Metrics**, select the **Aggregation** dropdown and choose **Average** and then select the **Field** dropdown and choose **FlightDelayMin**.
   1. Under **Metrics**, select **Add** to add another Y-axis aggregation. 
   1. Select the **Aggregation** dropdown and choose **Max** and then select the **Field** dropdown and choose **FlightDelayMin**.
1. Add a **Buckets** aggregation.
   1. Select **Add** and then **X-axis** in the **Add Bucket** window.
   1. From the **Aggregation** dropdown, select **Date Histogram**. 
   1. From the **Field** dropdown, select **timestamp**. 
   1. Select **Update**. 
1. Add a sub-aggregation.
   1. Select **Add** and then in the **Add Sub-Buckets** window, select **Split series**.
   1. From the **Sub aggregation** dropdown, select **Terms**.
   1. From the **Field** dropdown, select **FlightDelay**.
   1. Select **Update** to reflect these parameters in the graph.  

You've now created the following aggregation-based area chart.

![Resulting aggregation-based area chart]({{site.url}}{{site.baseurl}}/images/dashboards/area-aggregation-tutorial.png)

# Related links

- [Visualize]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)
- [Visualization types in OpenSearch Dashboards]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-types/)
- [Install and configure OpenSearch Dashboards]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [Aggregations]({{site.url}}{{site.baseurl}}/opensearch/aggregations/)