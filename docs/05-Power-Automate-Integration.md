# Integrating Application Insights with Power Automate Flows

This guide demonstrates how to integrate Azure Application Insights with Power Automate Cloud Flows to track flow execution, performance metrics, and diagnostics.

## Overview

Microsoft now provides native support for integrating Power Automate Cloud Flows with Application Insights. This enables you to:

1. Track flow executions with minimal configuration
2. Monitor flow performance and error rates
3. Create custom dashboards and alerts for your flows
4. Correlate flow telemetry with other Power Platform components

## Integration Methods

There are two approaches to integrating Power Automate with Application Insights:

1. **Environment-level configuration** (recommended): Configure once for all flows in an environment
2. **Custom telemetry approach**: Use HTTP requests to send tailored telemetry to Application Insights API

This guide covers both methods, starting with the simpler environment-level approach.

## Method 1: Environment-Level Integration (Recommended)

### Step 1: Get Your Application Insights Connection String

1. **In the Azure Portal**, go to your Application Insights resource
2. **Navigate to "Settings > Properties"**
3. **Copy the Connection String** (not just the instrumentation key)

### Step 2: Enable Application Insights in Power Platform Admin Center

1. **Go to the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)**
2. **Select your environment** and click on "Settings"
3. **Navigate to "Product > Privacy + Optimization"**
4. **Find the "Usage and error tracking" section**
5. **Toggle "Send usage data to Application Insights" to On**
6. **Paste your connection string** in the field provided
7. **Click "Save"**

### Step 3: Verify Configuration

1. **Wait for telemetry to flow** - It might take up to an hour for data to begin appearing
2. **In the Azure Portal**, go to your Application Insights resource
3. **Navigate to "Logs"** and run a simple query to check for flow data:

```kusto
customEvents
| where cloud_RoleName == "PowerAutomate"
| take 10
```

## What Gets Tracked Automatically

With environment-level integration enabled, the following telemetry is automatically tracked:

### Events Tracked

- **Flow Run Started**: When a flow execution begins
- **Flow Run Completed**: When a flow execution completes successfully
- **Flow Run Failed**: When a flow execution fails
- **Action Started**: When an action within a flow begins execution
- **Action Completed**: When an action completes successfully
- **Action Failed**: When an action fails

### Properties Tracked

Each event includes valuable context such as:

- **Flow Name**: The name of the flow being executed
- **Flow ID**: The unique identifier of the flow
- **Run ID**: The unique identifier of the specific flow run
- **Environment Name**: The Power Platform environment name
- **Action Name**: (For action events) The name of the action
- **Error Details**: (For failure events) The error message and code
- **Duration**: Execution time in milliseconds

## Analyzing Flow Telemetry with KQL Queries

Here are some useful Kusto Query Language (KQL) queries for analyzing your Power Automate telemetry:

### Flow Execution Overview

```kusto
customEvents
| where cloud_RoleName == "PowerAutomate"
| where name in ("Microsoft.Flow.Run.Started", "Microsoft.Flow.Run.Completed", "Microsoft.Flow.Run.Failed")
| extend flowName = tostring(customDimensions.flowDisplayName)
| extend flowId = tostring(customDimensions.flowId)
| extend runId = tostring(customDimensions.runId)
| extend status = case(
    name == "Microsoft.Flow.Run.Completed", "Success",
    name == "Microsoft.Flow.Run.Failed", "Failed", 
    "Started")
| extend duration = iff(name == "Microsoft.Flow.Run.Completed", todouble(customDimensions.duration), 0)
| project timestamp, flowName, status, duration, runId, itemId=tostring(customDimensions.itemId)
| order by timestamp desc
```

### Failed Cloud Flows

```kusto
dependencies 
| where success == false
| extend error = todynamic(tostring(customDimensions.error))
| extend tags = todynamic(tostring(customDimensions.tags))
| project 
    timestamp, 
    target, 
    operation_Id, 
    operation_ParentId, 
    name, 
    error.code, 
    error.message, 
    customDimensions.signalCategory,
    tags.capabilities,
    tags.environmentName,
    directlink=strcat("https://make.powerautomate.com/environments/", tags.environmentName, "/flows/",  target, "/runs/", operation_ParentId)
```

### Cloud Flows in use

```kusto
requests
| where timestamp > ago(1h)
| extend requestData = todynamic(tostring(customDimensions.Data))
| join kind=leftouter (dependencies | where type == "Cloud Flow/Cloud flow triggers" | project TriggerName = name, target) on $left.name == $right.target
| summarize FlowDisplayNameRuns = count() by tostring(requestData.FlowDisplayName), TriggerName, tostring(requestData.FlowDisplayName), tostring(requestData.tags.createdBy)
| project TriggerName,  tostring(requestData_FlowDisplayName), FlowDisplayNameRuns, requestData_tags_createdBy
```

### Cloud Flow runs with direct link to flow runs

```kusto
requests 
| where customDimensions.resourceProvider == 'Cloud Flow'
| extend data = todynamic(tostring(customDimensions.Data))
| project timestamp, operation_Id,  operation_ParentId, success, duration, customDimensions.signalCategory, name, data.FlowDisplayName, data.tags.xrmWorkflowId , data.tags.createdBy, 
data, customDimensions, directlink=strcat("https://make.powerautomate.com/environments/", data.tags.environmentName, "/flows/",  name, "/runs/", operation_Id)
```

### Flow Performance Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerAutomate"
| where name == "Microsoft.Flow.Run.Completed"
| extend flowName = tostring(customDimensions.flowDisplayName)
| extend duration = todouble(customDimensions.duration)
| summarize
    avgDuration = avg(duration),
    p95Duration = percentile(duration, 95),
    maxDuration = max(duration),
    executions = count()
    by flowName
| order by avgDuration desc
```

### Summary of flow runs

```kusto
requests
| extend cd = parse_json(customDimensions)
| where cd.resourceProvider has "Cloud Flow"
| extend data = parse_json(tostring(cd.Data))
| extend FlowName = tostring(data.FlowDisplayName)
|summarize nRun = count(), nFailed = countif(success==false) by FlowName
| extend nSuccess = nRun-toint(nFailed)
| project FlowName, nRun,nSuccess,nFailed
| order by toint(nRun) desc
```

## Method 2: Custom Telemetry Tracking

While the built-in tracking provides excellent baseline telemetry, you may want to track additional custom business events from your flows.

### Step 1: Create an API Key in Application Insights

1. **In the Azure Portal**, go to your Application Insights resource
2. **Navigate to "Settings > API Access"**
3. **Click "Create API key"**
4. **Enter a name** for your key (e.g., "PowerAutomateAccess")
5. **Select "Write telemetry" permission**
6. **Click "Generate key"** 
7. **Copy and securely store this key** - you won't be able to retrieve it later

### Step 2: Store Connection Information in Environment Variables

Create environment variables to store your Application Insights configuration:

1. **In the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)**, select your environment
2. **Go to "Settings > Product > Features"**
3. **In the "Environment Variables" section**, create these variables:
   - **AppInsightsInstrumentationKey** - Your Application Insights Instrumentation Key
   - **AppInsightsApiKey** - The API key you generated in the previous step
   - **AppInsightsEndpoint** - Set as `https://dc.services.visualstudio.com/v2/track`

### Step 3: Create a Custom Event Tracking Flow

Now create a reusable flow that can track custom events:

1. **In Power Automate**, create a new **Instant cloud flow**
2. **Name it** "Track Custom Event To Application Insights"
3. **Select "Manually trigger a flow"** as the trigger
4. **Add these trigger inputs**:
   - **eventName** (Text) - The name of the event to track
   - **properties** (JSON) - Custom properties for the event (optional)
   - **measurements** (JSON) - Custom measurements for the event (optional)
   - **correlationId** (Text) - ID to correlate with automatic telemetry (optional)

5. **Add an "Initialize variable" action** to create a timestamp variable:
   - **Name**: timestamp
   - **Type**: String
   - **Value**: `@{utcNow()}`

6. **Add an "Initialize variable" action** to build the request body:
   - **Name**: requestBody
   - **Type**: String
   - **Value**: Copy the following JSON:
   ```json
   {
     "name": "Microsoft.ApplicationInsights.Event",
     "time": "@{variables('timestamp')}",
     "iKey": "@{variables('AppInsightsInstrumentationKey')}",
     "data": {
       "baseType": "EventData",
       "baseData": {
         "ver": 2,
         "name": "@{triggerBody()['eventName']}",
         "properties": @{if(empty(triggerBody()['properties']), {}, triggerBody()['properties'])},
         "measurements": @{if(empty(triggerBody()['measurements']), {}, triggerBody()['measurements'])}
       }
     },
     "tags": {
       "ai.cloud.role": "PowerAutomate",
       "ai.cloud.roleInstance": "Custom",
       "ai.operation.id": "@{if(empty(triggerBody()['correlationId']), guid(), triggerBody()['correlationId'])}"
     }
   }
   ```

7. **Add a "Get environment variable" action** to retrieve the Instrumentation Key:
   - **Environment Variable Name**: AppInsightsInstrumentationKey

8. **Add a "Get environment variable" action** to retrieve the API Key:
   - **Environment Variable Name**: AppInsightsApiKey

9. **Add a "Set variable" action** to update the request body with the instrumentation key:
   - **Name**: requestBody
   - **Value**: `@{replace(variables('requestBody'), '@{variables(\'AppInsightsInstrumentationKey\')}', outputs('Get_environment_variable')?['value'])}`

10. **Add an "HTTP" action** to send telemetry to Application Insights:
    - **Method**: POST
    - **URI**: `https://dc.services.visualstudio.com/v2/track`
    - **Headers**:
      - **Content-Type**: application/json
      - **X-Api-Key**: `@{outputs('Get_environment_variable_2')?['value']}`
    - **Body**: `@{variables('requestBody')}`

11. **Save** your flow

### Step 4: Track Business Events in Your Flows

Now you can track custom business events from any flow:

1. **In your business flow**, at appropriate points, **add a "Run a Child Flow" action** 
2. **Select the "Track Custom Event To Application Insights" flow**
3. **Set Inputs** with business-relevant information:
   ```json
   {
     "eventName": "BusinessProcessCompleted",
     "properties": {
       "processName": "InvoiceApproval",
       "requesterId": "user123",
       "invoiceAmount": "1250.00",
       "approvalStatus": "Approved"
     },
     "measurements": {
       "approvalTime": 45.5,
       "invoiceAmount": 1250.00
     },
     "correlationId": "@{workflow()['run']['name']}"
   }
   ```

4. Make sure to use the **same correlationId as the flow run ID** to correlate with automatic telemetry

## Combining Both Methods for Comprehensive Monitoring

For best results, use both approaches:

1. **Enable environment-level tracking** for automatic telemetry collection
2. **Use custom event tracking** for specific business events and metrics

This combination provides:
- Technical monitoring (automatic telemetry)
- Business process monitoring (custom events)
- End-to-end correlation between both

## Creating Alerts for Flow Issues

Once your telemetry is flowing, set up alerts for critical flow failures:

1. **In your Application Insights resource**, go to "Alerts"
2. **Click "Create > Alert rule"**
3. **For Resource**, select your Application Insights instance
4. **For Condition**, create a new condition:
   - **Signal type**: Log
   - **Signal name**: Custom log search
   - **Search query**:
   ```kusto
   customEvents 
   | where cloud_RoleName == "PowerAutomate"
   | where name == "Microsoft.Flow.Run.Failed"
   | where customDimensions.flowDisplayName == "YourCriticalFlowName"
   ```
   - **Alert logic**: Based on number of results > 0
   - **Evaluation frequency**: How often to run the query (e.g., 5 minutes)
   
5. **Add an Action Group** to notify the appropriate people
6. **Set alert details** like name, severity, etc.
7. **Create** the alert rule

## Best Practices for Flow Telemetry

1. **Use descriptive flow names** to make telemetry more readable
2. **Track business metrics** that matter to your organization, not just technical metrics
3. **Correlate custom events with automatic telemetry** using the flow run ID
4. **Create a dashboard** in Azure Portal to visualize your most important flow metrics
5. **Set up alerts** for critical flow failures
6. **Track SLA-relevant metrics** like duration and success rate
7. **Consider data volume** - track important events without logging excessive data

## Next Steps

Now that you've set up Application Insights tracking in your Power Automate flows, proceed to [Custom Connectors Integration](./06-Custom-Connectors-Integration.md) to learn how to integrate Application Insights with custom connectors.

## Documentation References

- [Power Platform integration with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights)
- [Monitor cloud flows with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/app-insights-cloud-flow)
- [Application Insights REST API](https://docs.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics)
- [Power Automate Environment Variables](https://docs.microsoft.com/en-us/power-automate/environment-variables)