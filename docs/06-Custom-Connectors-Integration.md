# Integrating Application Insights with Custom Connectors

This guide demonstrates how to integrate Azure Application Insights with Power Platform Custom Connectors to track usage, performance metrics, and errors.

## Overview

Custom Connectors in Power Platform don't have built-in Application Insights integration like Canvas apps do. Instead, we need to create a monitoring wrapper around custom connector calls. This guide will show you how to:

1. Design a monitoring strategy for custom connectors
2. Create wrapper flows to track connector usage
3. Implement error handling and performance tracking
4. Analyze connector telemetry in Application Insights

## Monitoring Strategy Options

There are two primary approaches to monitor custom connectors:

1. **Wrapper Flows** - Create Power Automate flows that wrap calls to your connector
2. **Proxy API** - Create an Azure API Management or Azure Functions proxy in front of your connector

This guide focuses on the Wrapper Flow approach as it's more accessible to Power Platform users without requiring additional Azure services.

## Creating a Custom Connector Monitoring Wrapper Flow

### Step 1: Create a "Track Connector Event" Flow

First, let's create a child flow that will track connector events to Application Insights. This is similar to the flow we created in the [Power Automate Integration guide](./05-Power-Automate-Integration.md), but with connector-specific properties:

1. **Create a new instant flow** named "Track Connector Event"
2. **Set up these trigger inputs**:
   - **eventName** (Text) - The name of the event to track
   - **connectorName** (Text) - The name of the custom connector
   - **operationName** (Text) - The operation being performed
   - **operationId** (Text) - ID to correlate related events
   - **properties** (JSON) - Additional properties
   - **measurements** (JSON) - Numeric measurements
   - **success** (Boolean) - Whether the operation succeeded

3. **Use the "Track Event To Application Insights" flow** created in the previous module to send the event with these properties

### Step 2: Create a Wrapper Flow for Your Connector

Now, create a wrapper flow for each connector operation you want to monitor:

1. **Create a new instant flow** that will serve as the wrapper
2. **Set up trigger inputs matching** the parameters required by your connector
3. **Add a "Track Connector Event" action** to track the start:
   ```json
   {
     "eventName": "ConnectorOperationStarted",
     "connectorName": "YourCustomConnectorName",
     "operationName": "GetRecords",
     "operationId": "@{guid()}",
     "properties": {
       "endpoint": "records",
       "parameters": "@{string(triggerBody())}",
       "environment": "@{environment().name}"
     }
   }
   ```

4. **Store the operation ID** in a variable to use for correlation
5. **Add a Try/Catch pattern** using scopes and run after configuration:
   - **In the Try scope**: Add your custom connector action
   - **In the Catch scope**: Track connector failure

6. **After successful connector call**, add another "Track Connector Event" action:
   ```json
   {
     "eventName": "ConnectorOperationCompleted",
     "connectorName": "YourCustomConnectorName",
     "operationName": "GetRecords",
     "operationId": "@{variables('operationId')}",
     "properties": {
       "endpoint": "records",
       "responseSize": "@{length(string(outputs('CustomConnectorAction')['body']))}",
       "recordCount": "@{length(outputs('CustomConnectorAction')['body']['value'])}",
       "environment": "@{environment().name}"
     },
     "measurements": {
       "duration": "@{div(sub(ticks(utcNow()), ticks(variables('startTime'))), 10000)}"
     },
     "success": true
   }
   ```

7. **Return the connector response** to the calling flow

### Step 3: Implement the Error Handling Path

In your wrapper flow's "Catch" scope:

1. **Add a "Track Connector Event" action**:
   ```json
   {
     "eventName": "ConnectorOperationFailed",
     "connectorName": "YourCustomConnectorName",
     "operationName": "GetRecords",
     "operationId": "@{variables('operationId')}",
     "properties": {
       "endpoint": "records",
       "errorMessage": "@{result('TryScope')['error']['message']}",
       "errorCode": "@{result('TryScope')['error']['code']}",
       "environment": "@{environment().name}"
     },
     "measurements": {
       "duration": "@{div(sub(ticks(utcNow()), ticks(variables('startTime'))), 10000)}"
     },
     "success": false
   }
   ```

2. **Return an error response** or handle the error appropriately

## Creating a Solution for Connector Monitoring

To make your monitoring solution reusable:

1. **Create a solution** in Power Apps that includes:
   - Your "Track Event To Application Insights" flow
   - Your "Track Connector Event" flow
   - Environment variables for Application Insights settings
   
2. **Export and import the solution** to other environments where you use your connectors

## Using Your Monitored Connector

To use your monitored connector:

1. **Replace direct connector calls** with calls to your wrapper flow
2. **Pass the same parameters** as you would to the connector
3. **Process the response** exactly as you would from the direct connector call

Example call to a monitored connector:
```
FlowInput:
{
  "connectionId": "your-connection-id",
  "entityName": "contacts",
  "filter": "firstname eq 'John'"
}
```

## Analyzing Connector Telemetry in Application Insights

Here are some useful KQL queries for analyzing connector telemetry:

### Connector Performance Analysis

```kusto
customEvents
| where name in ("ConnectorOperationStarted", "ConnectorOperationCompleted", "ConnectorOperationFailed")
| extend connectorName = tostring(customDimensions.connectorName)
| extend operationName = tostring(customDimensions.operationName)
| extend operationId = tostring(customDimensions.operationId)
| extend status = case(
    name == "ConnectorOperationCompleted", "Success",
    name == "ConnectorOperationFailed", "Failed", 
    "Started")
| extend duration = todouble(customDimensions.duration)
| order by timestamp asc
| project timestamp, connectorName, operationName, status, duration, customDimensions
```

### Connector Error Analysis

```kusto
customEvents
| where name == "ConnectorOperationFailed"
| extend connectorName = tostring(customDimensions.connectorName)
| extend operationName = tostring(customDimensions.operationName)
| extend errorMessage = tostring(customDimensions.errorMessage)
| extend errorCode = tostring(customDimensions.errorCode)
| summarize errorCount = count() by connectorName, operationName, errorMessage
| order by errorCount desc
```

### Connector Usage Analysis

```kusto
customEvents
| where name == "ConnectorOperationStarted"
| extend connectorName = tostring(customDimensions.connectorName)
| extend operationName = tostring(customDimensions.operationName)
| extend endpoint = tostring(customDimensions.endpoint)
| summarize callCount = count() by bin(timestamp, 1h), connectorName, operationName
| render timechart
```

## Advanced: Creating a More Efficient Implementation

For production environments with high connector usage, consider these enhancements:

### 1. Batch Telemetry Sending

Instead of sending telemetry for every connector call:

1. **Store events in a collection** variable
2. **Use a scheduled flow** to send batches of events periodically
3. **Configure connection pooling** for the HTTP connections to Application Insights

### 2. Selective Telemetry

To reduce data volume:

1. **Sample requests** - Only track a percentage of calls
2. **Filter by importance** - Only track important operations
3. **Track aggregated metrics** - Send summary statistics instead of individual events

### 3. Using Correlation IDs Across Systems

To trace end-to-end operations:

1. **Pass a correlation ID** through all systems involved in a business process
2. **Include the ID in all telemetry** across Power Apps, Flows, and Connectors
3. **Use the operation_Id field** in Application Insights to correlate events

## Best Practices for Custom Connector Telemetry

1. **Balance detail vs. volume** - Track enough detail to be useful without overwhelming Application Insights with data
2. **Add business context** - Include business-relevant properties in your telemetry
3. **Standardize property names** - Use consistent naming across all your connectors
4. **Include correlation IDs** - Always correlate related events with operation IDs
5. **Track performance metrics** - Always include duration measurements for performance analysis
6. **Set up alerts** for connector failures and performance degradation
7. **Create a dashboard** showing connector health and performance

## Next Steps

Now that you've set up Application Insights tracking for your custom connectors, proceed to [Monitoring and Analytics](./07-Monitoring-and-Analytics.md) to learn how to create comprehensive monitoring dashboards and alerts.

## Documentation References

- [Power Platform Custom Connectors Documentation](https://docs.microsoft.com/en-us/connectors/custom-connectors/)
- [Application Insights REST API](https://docs.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics)
- [Custom Connector Error Handling](https://docs.microsoft.com/en-us/connectors/custom-connectors/error-handling)