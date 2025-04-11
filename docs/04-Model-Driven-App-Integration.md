# Integrating Application Insights with Power Apps Model-Driven Apps

This guide demonstrates how to integrate Azure Application Insights with a Power Apps Model-Driven app to track custom events, errors, and user behaviors.

## Overview

Power Apps now provides built-in Application Insights integration for Model-Driven apps, allowing you to easily track telemetry without complex custom code. This guide will show you how to:

1. Set up a Model-Driven app with Application Insights integration
2. Configure telemetry settings at the environment level
3. Review tracked telemetry data
4. Implement custom tracking for specific scenarios (optional)

## Adding Application Insights to Model-Driven Apps

### Step 1: Get Your Application Insights Connection String

1. **In the Azure Portal**, go to your Application Insights resource
2. **Navigate to "Settings > Properties"**
3. **Copy the Connection String** (not just the instrumentation key)

### Step 2: Enable Application Insights in Power Platform Admin Center

Unlike Canvas apps which are configured individually, Model-Driven apps use environment-level telemetry settings:

1. **Go to the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)**
2. **Select your environment** and click on "Settings"
3. **Under "Product" section**, select "Privacy + Optimization"
4. **Locate the "Usage and error tracking" section**
5. **Enable "Send usage data to Application Insights"**
6. **Paste your connection string** in the field provided
7. **Click "Save"**

### Step 3: Verify Configuration

1. **In the Power Platform Admin Center**, navigate to your environment
2. **Check "Settings > Privacy + Optimization"** to confirm your settings are saved
3. It might take some time (up to an hour) for telemetry to begin flowing to Application Insights

## What Gets Tracked Automatically

Once you've configured Application Insights integration, the following telemetry is automatically tracked:

### Automatic Events

- **App Launch**: When users open your Model-Driven app
- **Form Load**: When users open entity forms
- **View Load**: When users access views
- **Business Process Flow (BPF) Stage Changes**: When users move through business process stages
- **Command Execution**: When users click on ribbon buttons or other commands
- **Error Events**: Any unhandled errors or exceptions
- **Performance Metrics**: Load times and responsiveness metrics

### Automatic Properties

Each event includes helpful context like:

- **User ID**: The ID of the user using the app
- **Session ID**: Unique identifier for the user's session
- **App Name**: The name of your Model-Driven app
- **Environment**: The environment where the app is running
- **Entity/Form Name**: The specific entity or form being accessed
- **Operation Name**: The specific operation being performed

## Viewing Your Model-Driven App Telemetry in Application Insights

After implementing Application Insights in your Model-Driven app:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time telemetry as users interact with your app
3. **Use "Logs"** to query specific events and create custom analytics

## Sample Queries for Model-Driven App Analysis

Here are some useful Kusto queries you can run in the "Logs" section of Application Insights:

### Form Load Performance Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "FormLoad" 
| extend entityName = tostring(customDimensions.entityName)
| extend formType = tostring(customDimensions.formType)
| extend loadTime = todouble(customDimensions.loadTime)
| summarize avgLoadTime=avg(loadTime), p95LoadTime=percentile(loadTime, 95), count() by entityName, formType
| order by avgLoadTime desc
```

### Command Usage Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "CommandExecute" 
| extend commandName = tostring(customDimensions.commandName)
| extend entityName = tostring(customDimensions.entityName)
| summarize commandCount=count() by commandName, entityName
| order by commandCount desc
```

### Error Analysis

```kusto
exceptions
| where cloud_RoleName == "PowerApps"
| extend appName = tostring(customDimensions.appName)
| extend formName = tostring(customDimensions.formName)
| extend errorType = tostring(customDimensions.errorType)
| summarize ErrorCount=count() by appName, formName, errorType
| order by ErrorCount desc
```

### Business Process Flow Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "BPFStageChange" 
| extend processName = tostring(customDimensions.bpfName)
| extend fromStage = tostring(customDimensions.fromStage)
| extend toStage = tostring(customDimensions.toStage)
| summarize stageChangeCount=count() by processName, fromStage, toStage
| order by stageChangeCount desc
```

## Advanced: Implementing Custom Tracking with Client API

While the built-in tracking is comprehensive, you might want to track specific business events using the Dynamics 365 Client API.

### Step 1: Create a JavaScript Web Resource

Create a new JavaScript web resource to add custom telemetry:

```javascript
// Custom Application Insights Tracking
function trackCustomEvent(eventName, properties) {
    try {
        // Get the client-side telemetry service
        var telemetryService = window.appInsights;
        
        if (!telemetryService) {
            console.warn("Application Insights telemetry service not available");
            return;
        }
        
        // Add standard properties that might be useful
        var standardProps = {
            timestamp: new Date().toISOString(),
            userId: Xrm.Utility.getGlobalContext().userSettings.userId,
            userName: Xrm.Utility.getGlobalContext().userSettings.userName,
            appName: Xrm.Utility.getGlobalContext().getClientUrl()
        };
        
        // Combine custom properties with standard properties
        var allProperties = Object.assign({}, standardProps, properties || {});
        
        // Track the event
        telemetryService.trackEvent({ name: eventName, properties: allProperties });
    } catch (e) {
        console.error("Error tracking custom event", e);
    }
}
```

### Step 2: Add the Web Resource to Your Form

1. Open your form in the form editor
2. Add your web resource to the form libraries
3. Set it to load on form load

### Step 3: Call the Custom Tracking from Form Events

```javascript
function onSaveCustomTracking(executionContext) {
    var formContext = executionContext.getFormContext();
    var entityName = formContext.data.entity.getEntityName();
    var id = formContext.data.entity.getId();
    
    // Example of tracking a custom business event
    trackCustomEvent("CustomBusinessOperation", {
        entityName: entityName,
        recordId: id,
        operationType: "Save",
        businessUnit: formContext.getAttribute("businessunit").getValue()[0].name,
        recordValue: formContext.getAttribute("estimatedvalue").getValue()
    });
}
```

## Best Practices for Model-Driven App Telemetry

1. **Use the out-of-box integration** when possible instead of building custom tracking
2. **Create focused queries** to analyze specific user behaviors or performance issues
3. **Set up alerts** for critical errors or performance degradation
4. **Use custom tracking sparingly** to avoid overwhelming your logs with excessive data
5. **Consider data privacy** - don't log personally identifiable information (PII) unnecessarily
6. **Combine model-driven app telemetry with other sources** (Canvas apps, Flows) for end-to-end analysis

## Next Steps

Now that you've set up Application Insights tracking in your Model-Driven app, proceed to [Power Automate Flow Integration](./05-Power-Automate-Integration.md) to learn how to integrate Application Insights with Power Automate flows.

## Documentation References

- [Power Platform integration with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights)
- [Monitor model-driven apps with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/monitor-model-driven-apps)
- [Client API Reference for Model-driven apps](https://docs.microsoft.com/en-us/powerapps/developer/model-driven-apps/clientapi/reference)