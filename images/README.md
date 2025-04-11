# Screenshot Reference Guide

This document provides a list of all screenshots referenced in the training materials that you'll need to create.

## 02-Setting-Up-Application-Insights.md

1. **create-app-insights.png**
   - Screenshot of the Azure Portal showing the "Create a resource" page with Application Insights selected
   - Should display the search results for "Application Insights" and the option to create

2. **configure-app-insights.png**
   - Screenshot of the Application Insights creation form in the Azure Portal
   - Should show the resource group, name, region, and resource mode selections

3. **app-insights-overview.png**
   - Screenshot of the Application Insights resource overview page
   - Should highlight the instrumentation key and connection string locations

## 03-Canvas-App-Integration.md

1. **app-insights-connection-string.png**
   - Screenshot of the Application Insights Properties page in Azure Portal
   - Should highlight the Connection String field that needs to be copied

2. **canvas-telemetry-settings.png**
   - Screenshot of the Power Apps maker portal showing the Monitor settings for a Canvas app
   - Should show the toggle for "Send app telemetry to Application Insights" and the connection string field

3. **canvas-trace-example.png** (not explicitly referenced, but useful to add)
   - Screenshot showing the Power Fx formula editor with a Trace function example
   - Should display example of custom telemetry using the Trace function

## 04-Model-Driven-App-Integration.md

1. **model-environment-variable.png**
   - Screenshot showing environment variable configuration for Model-Driven apps
   - Similar to the canvas app version but in the context of Model-Driven app settings

2. **web-resource-creation.png** (not explicitly referenced, but useful to add)
   - Screenshot showing the creation of a JavaScript web resource
   - Should display the web resource properties and code editor

3. **form-event-configuration.png** (not explicitly referenced, but useful to add)
   - Screenshot showing how to add event handlers to a Model-Driven app form
   - Should display the form editor events tab with OnLoad handler configuration

## 05-Power-Automate-Integration.md

1. **app-insights-api.png**
   - Screenshot of the Application Insights API Access page
   - Should show the API keys section and Instrumentation Key information

2. **create-api-key.png**
   - Screenshot showing the creation of an API key in Application Insights
   - Should display the key creation form with permissions options

3. **flow-environment-variables.png**
   - Screenshot showing the environment variables in Power Platform admin center
   - Should display the App Insights related variables

4. **track-event-flow.png**
   - Screenshot of the "Track Event To Application Insights" flow
   - Should show the flow design with HTTP action configuration

5. **flow-execution-tracking.png**
   - Screenshot showing a flow with start and end tracking actions
   - Should display both Track Event actions in a larger flow

6. **flow-error-handling.png**
   - Screenshot showing error handling with scope actions in Power Automate
   - Should display the scope action and configured error handling path

## 06-Custom-Connectors-Integration.md

1. **connector-wrapper-flow.png**
   - Screenshot of a wrapper flow for a custom connector
   - Should show the pattern with tracking before and after connector call

## 07-Monitoring-and-Analytics.md

1. **dashboard-configuration.png**
   - Screenshot showing Azure Dashboard configuration with App Insights tiles
   - Should display the tile gallery and sample tiles added to a dashboard

2. **custom-metrics.png**
   - Screenshot showing the configuration of a custom metrics chart 
   - Should display the metrics configuration panel with selections

3. **custom-query-tile.png**
   - Screenshot showing the configuration of a log query tile
   - Should display the query editor and visualization options

4. **alert-condition.png**
   - Screenshot of alert rule condition configuration
   - Should show signal selection and threshold configuration

5. **alert-details.png**
   - Screenshot of alert rule details configuration
   - Should display action groups and rule details fields

6. **power-bi-dashboard.png**
   - Screenshot of a Power BI dashboard with App Insights data
   - Should show sample visualizations for Power Platform telemetry data

## Additional Screenshots

1. **app-insights-logs.png**
   - Screenshot of the Application Insights Logs (KQL query) interface
   - Should show a sample query and results related to Power Platform

2. **app-insights-live-metrics.png**
   - Screenshot of the Application Insights Live Metrics view
   - Should show real-time telemetry data coming in

3. **app-insights-application-map.png**
   - Screenshot of the Application Insights Application Map
   - Should show connections between Power Platform components

4. **app-insights-failures.png**
   - Screenshot of the Application Insights Failures section
   - Should show error breakdown and analysis options

5. **powerapp-telemetry-results.png**
   - Screenshot showing Application Insights query results with Power Apps telemetry
   - Should show actual telemetry data from a Canvas app with the standard schema

## Instructions for Creating Screenshots

When creating these screenshots:

1. **Remove sensitive information**: Blur or redact subscription IDs, tenant IDs, keys, connection strings, and any personal or organizational identifiers.

2. **Use consistent styling**: Try to maintain a consistent zoom level, window size, and theme across screenshots.

3. **Highlight key areas**: Consider adding red boxes or arrows to highlight important areas that readers should focus on.

4. **Use dummy data**: For Power Platform screenshots, use sample/demo apps with non-sensitive data.

5. **Image dimensions**: Keep images reasonably sized - around 1200-1400 pixels wide is generally good for documentation.

6. **File format**: Save all screenshots as PNG files for best quality and clarity.

7. **Naming convention**: Use the exact filenames referenced in this document to ensure they match the documentation references.