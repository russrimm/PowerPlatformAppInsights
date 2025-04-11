# Monitoring and Analytics with Application Insights

This guide demonstrates how to create comprehensive monitoring dashboards and analytics for your Power Platform solutions using Azure Application Insights.

## Overview

After integrating Application Insights with your Power Platform components (Canvas Apps, Model-Driven Apps, Power Automate Flows, and Custom Connectors), you now have valuable telemetry data. This guide shows you how to:

1. Create custom dashboards for monitoring
2. Set up alerts for important metrics
3. Create advanced analytics with Kusto Query Language (KQL)
4. Implement continuous improvement based on insights

## Understanding Telemetry Correlation

One of the most powerful features of Power Platform's Application Insights integration is end-to-end telemetry correlation. This allows you to:

1. **Follow complete operation paths** from a user's click in a model-driven app through the server processing and back
2. **Identify performance bottlenecks** across the entire operation chain
3. **Debug issues holistically** across both client and server components
4. **Share consistent operation IDs** with Microsoft support for faster troubleshooting

When analyzing correlated telemetry, use the `operation_Id` field to track related events across different components. This ID is consistent across client actions, server processing, and database operations.

> **Note**: Licensing requirement - Enablement of Application Insights is limited to customers with paid/premium Dataverse licenses available for the tenant.

## Benefits of Standardized Telemetry

The Power Platform's integration with Application Insights provides several significant benefits over custom telemetry solutions:

1. **Reduced Development Overhead**: No need to write and maintain custom code for telemetry collection, reducing development and maintenance costs.

2. **Performance Improvement**: Built-in telemetry has minimal performance impact compared to custom solutions, as it's optimized by Microsoft.

3. **Consistent Experience**: Standardized telemetry follows the Application Insights data model, making it easier to analyze and correlate across different components.

4. **Partner and Support Collaboration**: System integrators and partners don't need to learn custom telemetry implementations for different environments.

5. **Technical Support Efficiency**: When contacting Microsoft support, the operation_id values from standardized telemetry enable faster issue resolution.

## Custom Telemetry for Power Platform Components

While Power Platform provides extensive built-in telemetry, you may want to supplement it with custom telemetry for specific business scenarios.

### Dataverse Plugin Custom Telemetry

For Dataverse plugins, Microsoft provides the `Microsoft.Xrm.Sdk.PluginTelemetry.ILogger` interface to write telemetry directly to your Application Insights resource:

```csharp
public void Execute(IServiceProvider serviceProvider)
{
    // Get the tracing service
    ILogger logger = (ILogger)serviceProvider.GetService(typeof(ILogger));
    
    try
    {
        // Plugin logic
        logger.LogInformation("Custom plugin operation completed", new Dictionary<string, object>
        {
            { "entityName", "account" },
            { "operationType", "create" },
            { "businessUnit", "sales" }
        });
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "Plugin execution failed");
        throw;
    }
}
```

> **Note**: Telemetry sent through the ILogger interface is sent only to your Application Insights resource and never to Microsoft.

### Canvas App Custom Telemetry

For Canvas apps, use the built-in `Trace` function to send custom telemetry:

```
Trace(
    "BusinessEvent",            // Event name
    TraceSeverity.Information,  // Severity level
    {
        EventCategory: "OrderProcessing", 
        OrderAmount: Text(OrderTotal),
        CustomerType: CustomerSegment
    }
)
```

### Understanding the Application Insights Schema

All telemetry sent to Application Insights follows a [standardized schema](https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-model) with specific tables:

- **customEvents**: Contains custom events from apps and flows
- **traces**: Contains trace messages
- **exceptions**: Contains error information
- **requests**: Contains incoming requests
- **dependencies**: Contains outgoing requests
- **pageViews**: Contains page view information
- **browserTimings**: Contains client-side performance data

Power Platform telemetry is mapped to these standard tables to provide a consistent experience when analyzing data.

## Creating Custom Dashboards

### Step 1: Create a New Dashboard

1. **In the Azure Portal**, navigate to "Dashboard" and click "New dashboard"
2. Give your dashboard a name (e.g., "Power Platform Monitoring")
3. Select a layout that works for your needs

### Step 2: Add Application Insights Tiles

1. **In the tile gallery**, find and drag Application Insights tiles to your dashboard:
   - "Application Insights: Failures" tile
   - "Application Insights: Performance" tile
   - "Application Insights: Usage" tile

2. **Configure each tile** to show the data you're interested in:
   - Select your Application Insights resource
   - Choose appropriate time ranges
   - Select relevant metrics

### Step 3: Add Custom Metric Tiles

1. **Add a "Metrics chart" tile** to your dashboard
2. **Configure the metrics chart**:
   - Select your Application Insights resource
   - Choose appropriate metrics (e.g., "Server response time", "Failed requests")
   - Set the aggregation type (e.g., Average, Count)
   - Configure the chart type and time range

### Step 4: Add Custom Query Tiles

1. **Add a "Log query" tile** to your dashboard
2. **Configure the query**:
   - Select your Application Insights resource
   - Enter a custom KQL query (examples below)
   - Choose a visualization type (table, chart, etc.)
   - Set the time range

### Step 5: Share Your Dashboard

1. **Click "Share" on your dashboard**
2. **Set up sharing**:
   - Specify who can access the dashboard
   - Choose whether to publish to the directory

## Setting Up Alerts

Alerts help you proactively monitor your Power Platform solutions.

### Step 1: Create New Alert Rule

1. **In Application Insights**, navigate to "Alerts"
2. **Click "New alert rule"**

### Step 2: Configure Alert Conditions

1. **Select a signal type** (Metrics, Logs, Activity Log)
2. For metric-based alerts:
   - Choose a metric (e.g., "Failed requests", "Server response time")
   - Set the condition type (Greater than, Less than, etc.)
   - Set the threshold value
   - Set the aggregation period and frequency

3. For log-based alerts:
   - Enter a custom KQL query
   - Set the condition type (Number of results, Metric measurement)
   - Set the threshold value
   - Set the evaluation frequency

### Step 3: Configure Alert Details

1. **Set up action groups** to define what happens when the alert fires:
   - Email notifications
   - SMS notifications
   - Webhook calls to external systems
   - Azure Functions
   - Logic Apps

2. **Configure alert details**:
   - Name your alert rule
   - Set the severity level (0-4)
   - Add a description
   - Set whether the rule is enabled

### Example Alerts for Power Platform

#### Canvas App Error Rate Alert

```kusto
customEvents
| where name == "AppInsightsErrorEvent"
| where timestamp > ago(5m)
| summarize ErrorCount=count() by bin(timestamp, 1m)
| where ErrorCount > 5
```

#### Flow Failure Alert

```kusto
customEvents
| where name == "FlowFailed"
| where timestamp > ago(15m)
| summarize FailureCount=count() by bin(timestamp, 5m)
| where FailureCount > 2
```

#### Model-Driven App Performance Alert

```kusto
customEvents
| where name == "FormLoaded"
| where timestamp > ago(30m)
| extend formLoadTime = todouble(customDimensions.loadTime)
| where formLoadTime > 5000 // More than 5 seconds to load
| summarize SlowLoadCount=count() by bin(timestamp, 10m)
| where SlowLoadCount > 3
```

## Advanced Analytics with KQL

Kusto Query Language (KQL) allows you to create powerful custom queries against your telemetry data.

### User Journey Analysis

Track the complete user journey through your Power Platform apps:

```kusto
let users = customEvents 
| where timestamp > ago(24h)
| where name == "SessionStarted"
| project userId = tostring(customDimensions.userId), sessionId = tostring(customDimensions.sessionId), sessionStartTime = timestamp;

users
| join kind=inner (
    customEvents
    | where name == "PageViewed" 
    | project userId = tostring(customDimensions.userId), sessionId = tostring(customDimensions.sessionId), pageName = tostring(customDimensions.pageName), pageViewTime = timestamp
) on userId, sessionId
| where pageViewTime > sessionStartTime
| sort by userId, sessionId, pageViewTime asc
| project userId, sessionId, pageName, pageViewTime
| summarize PageSequence = make_list(pageName) by userId, sessionId
| top 100 by array_length(PageSequence) desc
```

### Performance Analysis by User Type

Compare app performance for different user types:

```kusto
customEvents
| where name == "AppAction" 
| extend userRole = tostring(customDimensions.userRole)
| extend actionName = tostring(customDimensions.actionName)
| extend duration = todouble(customDimensions.duration)
| summarize avg(duration), percentiles(duration, 50, 95, 99) by userRole, actionName
| order by avg_duration desc
```

### Error Pattern Analysis

Identify common error patterns across your Power Platform solutions:

```kusto
exceptions
| where timestamp > ago(7d)
| extend appComponent = tostring(customDimensions.appComponent)
| extend errorType = tostring(customDimensions.errorType)
| summarize ErrorCount=count() by appComponent, errorType, outerMessage
| order by ErrorCount desc
| top 20 by ErrorCount
```

### Cross-Component Analysis

Analyze how different components of your solution interact:

```kusto
// Start with Canvas App events
let canvasEvents = customEvents
| where timestamp > ago(24h)
| where name == "FlowTriggered" 
| extend flowName = tostring(customDimensions.flowName)
| extend canvasAppName = tostring(customDimensions.appName)
| extend userId = tostring(customDimensions.userId)
| project eventTime = timestamp, canvasAppName, flowName, userId, operation_Id;

// Join with Flow events
canvasEvents
| join kind=inner (
    customEvents
    | where name == "FlowExecuted"
    | extend flowName = tostring(customDimensions.flowName)
    | extend flowDuration = todouble(customDimensions.duration)
    | extend flowSuccess = tobool(customDimensions.succeeded)
    | project flowEventTime = timestamp, flowName, flowDuration, flowSuccess, operation_Id
) on operation_Id
| where flowEventTime > eventTime
| project canvasAppName, flowName, userId, flowSuccess, flowDuration, timeDiff = datetime_diff('second', flowEventTime, eventTime)
| summarize AvgFlowDuration = avg(flowDuration), SuccessRate = countif(flowSuccess) * 100.0 / count() by canvasAppName, flowName
| order by AvgFlowDuration desc
```

## Implementing Continuous Improvement

Use Application Insights data to drive continuous improvement of your Power Platform solutions.

### Step 1: Establish Baseline Metrics

1. **Identify key performance indicators** for your Power Platform solutions:
   - Load times for forms and screens
   - Error rates for key operations
   - User engagement metrics
   - Business process completion rates

2. **Set baseline values** for these metrics using Application Insights data

### Step 2: Create Improvement Hypotheses

1. **Analyze your telemetry data** to identify areas for improvement:
   - Screens or forms with high load times
   - Components with high error rates
   - User journeys with high drop-off rates

2. **Formulate hypotheses** about how to improve these areas

### Step 3: Implement Changes and Measure Impact

1. **Make targeted changes** based on your hypotheses
2. **Use A/B testing** if possible to compare variants
3. **Compare before and after metrics** using Application Insights

Example query for before/after comparison:

```kusto
let beforePeriod = customEvents
| where timestamp between(ago(14d) .. ago(7d))
| where name == "FormLoaded" 
| extend formName = tostring(customDimensions.formName)
| extend loadTime = todouble(customDimensions.loadTime)
| summarize BeforeAvg = avg(loadTime), BeforeP95 = percentile(loadTime, 95) by formName;

let afterPeriod = customEvents
| where timestamp > ago(7d)
| where name == "FormLoaded" 
| extend formName = tostring(customDimensions.formName)
| extend loadTime = todouble(customDimensions.loadTime)
| summarize AfterAvg = avg(loadTime), AfterP95 = percentile(loadTime, 95) by formName;

beforePeriod
| join kind=inner afterPeriod on formName
| extend AvgImprovement = (BeforeAvg - AfterAvg) / BeforeAvg * 100
| extend P95Improvement = (BeforeP95 - AfterP95) / BeforeP95 * 100
| project formName, BeforeAvg, AfterAvg, AvgImprovement, BeforeP95, AfterP95, P95Improvement
| order by AvgImprovement desc
```

## Creating Executive Dashboards with Power BI

You can also use Power BI to create rich, interactive dashboards using your Application Insights data.

### Step 1: Connect Power BI to Application Insights

1. **In Power BI Desktop**, click "Get Data"
2. **Select "Azure" and then "Azure Application Insights"**
3. **Enter your Application Insights API details**:
   - Application ID
   - API Key

### Step 2: Create Custom Queries

1. **Use the "Advanced Editor" in Power BI** to write custom KQL queries
2. **Create multiple queries** for different aspects of your solution

Example Power BI query:

```
let timeRange = 30d;
customEvents
| where timestamp > ago(timeRange)
| where name == "BusinessProcess"
| extend processName = tostring(customDimensions.processName)
| extend duration = todouble(customDimensions.duration)
| extend succeeded = tobool(customDimensions.succeeded)
| summarize 
    Count = count(),
    SuccessCount = countif(succeeded == true),
    FailureCount = countif(succeeded == false),
    AvgDuration = avg(duration),
    P95Duration = percentile(duration, 95)
    by processName
| extend SuccessRate = SuccessCount * 100.0 / Count
| order by Count desc
```

### Step 3: Create Interactive Visualizations

1. **Build custom visualizations** using the Power BI visualization tools:
   - Performance trend charts
   - Success/failure rate gauges
   - User activity heat maps
   - Error frequency tables

2. **Add drill-through capabilities** for detailed analysis

### Step 4: Share and Schedule Refresh

1. **Publish your dashboard** to the Power BI service
2. **Set up scheduled refresh** to keep data current
3. **Share with stakeholders** through the Power BI portal

## Next Steps

After setting up monitoring and analytics for your Power Platform solutions, proceed to [Troubleshooting Common Issues](./08-Troubleshooting-Common-Issues.md) to learn how to diagnose and fix problems using Application Insights.

## Documentation References

- [Analyze model-driven apps and Microsoft Dataverse telemetry with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/analyze-telemetry)
- [Monitor canvas apps with Application Insights](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/application-insights)
- [Monitor cloud flows with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/app-insights-cloud-flow)
- [Write Telemetry to your Application Insights resource using ILogger](https://learn.microsoft.com/en-us/power-platform/admin/enable-use-comprehensive-auditing#write-telemetry-to-your-application-insights-resource-using-ilogger)
- [Application Insights Data Model](https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-model)
- [Application Insights Analytics](https://docs.microsoft.com/en-us/azure/azure-monitor/app/analytics)
- [Kusto Query Language Reference](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Azure Monitor Alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [Power BI and Application Insights](https://docs.microsoft.com/en-us/power-bi/connect-data/service-azure-application-insights)