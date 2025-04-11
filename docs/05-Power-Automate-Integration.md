# Integrating Application Insights with Power Automate

This guide demonstrates how to integrate Azure Application Insights with Power Automate flows to track custom events, performance metrics, and errors.

## Overview

Power Automate flows can send telemetry data to Application Insights using HTTP requests. This integration allows you to:

1. Track flow executions and their outcomes
2. Monitor performance of your flows
3. Log custom events and business metrics
4. Record and analyze errors
5. Create dashboards for flow analytics

## Setting Up Application Insights for Power Automate Integration

### Step 1: Get Required Information from Application Insights

1. **Access your Application Insights resource** in the Azure Portal
2. **Note down the following information**:
   - Instrumentation Key (from Overview or API Access)
   - Application ID (from API Access)
   - API Key (you'll need to create one)

   ![Application Insights API Access](../images/app-insights-api.png)

3. **Create an API Key** (if you don't have one):
   - In the Application Insights resource, go to "API Access"
   - Click "Create API Key"
   - Select permissions (Read telemetry and Write annotations are sufficient)
   - Give the key a name (e.g., "PowerAutomateIntegration")
   - Copy the generated API key (you won't be able to see it again)

   ![Create API Key](../images/create-api-key.png)

### Step 2: Store Application Insights Information Securely

For secure storage of Application Insights credentials in Power Automate, use environment variables:

1. **In Power Platform admin center**:
   - Go to your environment
   - Navigate to "Environment variables"
   - Create variables for:
     - AppInsightsInstrumentationKey
     - AppInsightsApplicationId
     - AppInsightsApiKey
   - Set the data type to "Text" and mark as secure if needed
   - Set appropriate values for each variable

   ![Environment Variables](../images/flow-environment-variables.png)

## Creating Reusable Components for Telemetry

The most efficient way to integrate Application Insights with Power Automate is to create reusable components. We'll create:

1. A solution containing reusable flows
2. Custom connectors (optional for advanced scenarios)

### Creating Reusable Flows for Application Insights

#### Step 1: Create a Solution

1. Go to [Power Apps maker portal](https://make.powerapps.com)
2. Navigate to Solutions and create a new solution (e.g., "ApplicationInsightsIntegration")

#### Step 2: Create a Telemetry Helper Flow

1. In your solution, create a new "Instant cloud flow" named "Track Event To Application Insights"
2. Choose "Manually trigger a flow" as the trigger
3. Add Trigger inputs:
   - Name: EventName (Type: Text)
   - Name: Properties (Type: Object)
   - Name: Measurements (Type: Object, Optional)

4. Add an HTTP action:
   - Method: POST
   - URI: https://dc.services.visualstudio.com/v2/track
   - Headers:
     - Content-Type: application/json
     - X-Api-Key: [YourAPIKey] (use environment variable)
   - Body:

   ```json
   {
     "name": "Microsoft.ApplicationInsights.Event",
     "time": "@{utcNow()}",
     "iKey": "[YourInstrumentationKey]",
     "tags": {
       "ai.cloud.role": "PowerAutomate",
       "ai.cloud.roleInstance": "@{workflow()?['tags']['flowDisplayName']}"
     },
     "data": {
       "baseType": "EventData",
       "baseData": {
         "ver": 2,
         "name": "@{triggerBody()['EventName']}",
         "properties": @{if(empty(triggerBody()['Properties']), json('{}'), triggerBody()['Properties'])},
         "measurements": @{if(empty(triggerBody()['Measurements']), json('{}'), triggerBody()['Measurements'])}
       }
     }
   }
   ```

5. Save the flow and test it

   ![Track Event Flow](../images/track-event-flow.png)

#### Step 3: Create a Dependency Tracking Flow

1. Create another instant flow named "Track Dependency To Application Insights"
2. Add trigger inputs:
   - Name: DependencyName (Type: Text)
   - Name: DependencyType (Type: Text)
   - Name: Target (Type: Text)
   - Name: Success (Type: Boolean)
   - Name: Duration (Type: Integer)
   - Name: Properties (Type: Object, Optional)

3. Add an HTTP action with the same endpoint but different payload:

   ```json
   {
     "name": "Microsoft.ApplicationInsights.RemoteDependency",
     "time": "@{utcNow()}",
     "iKey": "[YourInstrumentationKey]",
     "tags": {
       "ai.cloud.role": "PowerAutomate",
       "ai.cloud.roleInstance": "@{workflow()?['tags']['flowDisplayName']}"
     },
     "data": {
       "baseType": "RemoteDependencyData",
       "baseData": {
         "ver": 2,
         "name": "@{triggerBody()['DependencyName']}",
         "id": "@{guid()}",
         "resultCode": "@{if(triggerBody()['Success'], '200', '500')}",
         "duration": "@{concat(triggerBody()['Duration'], ':0:0.0')}",
         "success": "@{triggerBody()['Success']}",
         "type": "@{triggerBody()['DependencyType']}",
         "target": "@{triggerBody()['Target']}",
         "properties": @{if(empty(triggerBody()['Properties']), json('{}'), triggerBody()['Properties'])}
       }
     }
   }
   ```

#### Step 4: Create an Exception Tracking Flow

1. Create another instant flow named "Track Exception To Application Insights"
2. Add trigger inputs:
   - Name: ExceptionMessage (Type: Text)
   - Name: ExceptionType (Type: Text)
   - Name: StackTrace (Type: Text, Optional)
   - Name: Properties (Type: Object, Optional)
   - Name: Measurements (Type: Object, Optional)

3. Add an HTTP action with exception tracking payload:

   ```json
   {
     "name": "Microsoft.ApplicationInsights.Exception",
     "time": "@{utcNow()}",
     "iKey": "[YourInstrumentationKey]",
     "tags": {
       "ai.cloud.role": "PowerAutomate",
       "ai.cloud.roleInstance": "@{workflow()?['tags']['flowDisplayName']}"
     },
     "data": {
       "baseType": "ExceptionData",
       "baseData": {
         "ver": 2,
         "exceptions": [
           {
             "typeName": "@{triggerBody()['ExceptionType']}",
             "message": "@{triggerBody()['ExceptionMessage']}",
             "hasFullStack": @{not(empty(triggerBody()['StackTrace']))},
             "stack": "@{if(empty(triggerBody()['StackTrace']), '', triggerBody()['StackTrace'])}"
           }
         ],
         "properties": @{if(empty(triggerBody()['Properties']), json('{}'), triggerBody()['Properties'])},
         "measurements": @{if(empty(triggerBody()['Measurements']), json('{}'), triggerBody()['Measurements'])}
       }
     }
   }
   ```

## Implementing in Your Flows

Now that you have created reusable telemetry flows, you can use them in your business flows:

### Example 1: Track Flow Execution Events

1. Open an existing flow or create a new one
2. At the beginning of the flow, add an action to call the "Track Event To Application Insights" flow:
   - EventName: "FlowStarted"
   - Properties:
     ```
     {
       "flowName": "@{workflow()?['tags']['flowDisplayName']}",
       "environment": "@{workflow()?['tags']['environmentName']}",
       "trigger": "@{triggerBody()?['triggerType']}"
     }
     ```

3. At the end of the flow, add another action to call the same flow:
   - EventName: "FlowCompleted"
   - Properties:
     ```
     {
       "flowName": "@{workflow()?['tags']['flowDisplayName']}",
       "environment": "@{workflow()?['tags']['environmentName']}",
       "duration": "@{ticks(utcNow()) - ticks(workflow()['run']['startTime'])}"
     }
     ```

   ![Flow Execution Tracking](../images/flow-execution-tracking.png)

### Example 2: Track External System Dependencies

When your flow interacts with external systems (SharePoint, Dynamics 365, etc.), track these dependencies:

1. Before calling the external system, initialize a variable to store the start time:
   ```
   Set variable 'StartTime' to @{utcNow()}
   ```

2. After the external system action, add an action to calculate duration:
   ```
   Set variable 'Duration' to @{div(sub(ticks(utcNow()), ticks(variables('StartTime'))), 10000000)}
   ```

3. Add an action to call the "Track Dependency To Application Insights" flow:
   - DependencyName: "SharePointGetItems" (or appropriate name)
   - DependencyType: "SharePoint" (or appropriate system)
   - Target: "https://yourtenant.sharepoint.com" (or appropriate target)
   - Success: @{actions('Get_items')?['status'] equals 'Succeeded'} (adjust action name)
   - Duration: @{variables('Duration')}
   - Properties:
     ```
     {
       "listName": "YourListName",
       "itemCount": "@{length(body('Get_items')?['value'])}"
     }
     ```

### Example 3: Track Business Metrics

For important business metrics you want to monitor:

1. At relevant points in your flow, add an action to call the "Track Event To Application Insights" flow:
   - EventName: "BusinessMetric"
   - Properties:
     ```
     {
       "metricName": "OrdersProcessed",
       "source": "PowerAutomate"
     }
     ```
   - Measurements:
     ```
     {
       "count": "@{length(variables('Orders'))}",
       "totalValue": "@{variables('OrderTotalValue')}"
     }
     ```

### Example 4: Error Handling with Try-Catch Pattern

Implement error handling using the scope actions in Power Automate:

1. Add a "Scope" action to wrap the steps that might fail
2. Add a "Configure run after" setting to the next action after the scope, setting it to run only if the scope fails
3. In that error handler action, call the "Track Exception To Application Insights" flow:
   - ExceptionMessage: "@{result('YourScopeActionName')?['error']['message']}"
   - ExceptionType: "@{result('YourScopeActionName')?['error']['code']}"
   - Properties:
     ```
     {
       "flowName": "@{workflow()?['tags']['flowDisplayName']}",
       "actionName": "@{first(result('YourScopeActionName')?['error']['source'])}"
     }
     ```

   ![Error Handling in Flow](../images/flow-error-handling.png)

## Viewing and Analyzing Flow Telemetry

After implementing Application Insights in your Power Automate flows:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time telemetry as your flows run
3. **Use "Logs"** to query specific events and create custom analytics

## Sample Kusto Queries for Power Automate Analysis

Here are some useful queries to analyze your Power Automate telemetry:

### Flow Execution Count and Performance

```kusto
customEvents
| where name == "FlowCompleted"
| extend flowName = tostring(customDimensions.flowName)
| extend durationInSeconds = todouble(customDimensions.duration)
| summarize count(), avg(durationInSeconds), max(durationInSeconds), min(durationInSeconds) by flowName
| order by count_ desc
```

### Flow Failures

```kusto
exceptions
| where customDimensions has "PowerAutomate"
| extend flowName = tostring(customDimensions.flowName)
| extend errorMessage = message
| summarize count() by flowName, errorMessage
| order by count_ desc
```

### Dependency Analysis

```kusto
dependencies
| where cloud_RoleName == "PowerAutomate" 
| extend dependencyType = type
| extend target = target
| summarize count(), avgDuration=avg(duration), failureCount=countif(success == false) by dependencyType, target
| extend failureRate = (failureCount * 100.0) / count_
| order by count_ desc
```

### Business Metric Tracking

```kusto
customEvents
| where name == "BusinessMetric"
| where customDimensions.metricName == "OrdersProcessed"
| extend count = todouble(customMeasurements.count)
| extend totalValue = todouble(customMeasurements.totalValue)
| summarize sum(count), sum(totalValue) by bin(timestamp, 1d)
| render timechart
```

## Next Steps

Now that you've set up Application Insights tracking in Power Automate, proceed to [Custom Connectors Integration](./06-Custom-Connectors-Integration.md) to learn how to integrate Application Insights with custom connectors.

## Documentation References

- [Power Automate Documentation](https://docs.microsoft.com/en-us/power-automate/)
- [Application Insights Data Model](https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-model)
- [Application Insights REST API](https://docs.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics)