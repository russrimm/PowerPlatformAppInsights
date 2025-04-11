# Integrating Application Insights with Power Apps Canvas Apps

This guide demonstrates how to integrate Azure Application Insights with a Power Apps Canvas app to track custom events, errors, and user behavior.

## Overview

Power Apps provides built-in Application Insights integration for Canvas apps, allowing you to easily track telemetry without writing custom code. This guide will show you how to:

1. Set up a Canvas app with Application Insights integration
2. Configure telemetry settings
3. Review tracked telemetry data
4. Implement custom tracking for specific scenarios (optional)

## Adding Application Insights to a Canvas App

### Step 1: Get Your Application Insights Connection String

1. **In the Azure Portal**, go to your Application Insights resource
2. **Navigate to "Settings > Properties"**
3. **Copy the Connection String** (not just the instrumentation key)

### Step 2: Add the Connection String to Your Canvas App

1. **Open your Canvas app** in the Power Apps maker portal.
2. **Select the App object** in the left navigation tree view.
3. **Paste the Connection String** from your Application Insights resource into the appropriate field.
4. **Save and Publish** your app.

### Step 3: Play the Published App

1. **Play the published app** and browse through its screens.
2. Events will automatically be logged to Application Insights, including usage details like user access location, device type, and browser type.

### Step 4: Configure Telemetry Options

In the same Monitor settings panel, you can configure:

1. **Session logging level** (choose from Error, Warning, or Information)
2. **Control-level error logging** (enable/disable tracking of control-level errors)

These settings help you balance detail against data volume.

## What Gets Tracked Automatically

Once you've configured Application Insights integration, the following telemetry is automatically tracked:

### Automatic Events

- **App Start/End**: When users open and close your app
- **Screen Navigation**: When users navigate between screens
- **Error Events**: Any unhandled errors or exceptions
- **Performance Metrics**: Load times and responsiveness metrics

### Automatic Properties

Each event includes helpful context like:

- **User ID**: The ID of the user using the app
- **Session ID**: Unique identifier for the user's session
- **App Name**: The name of your canvas app
- **App Version**: The version of your canvas app
- **Environment**: The environment where the app is running

## Viewing Your Canvas App Telemetry in Application Insights

After implementing Application Insights in your Canvas app:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time telemetry as users interact with your app
3. **Use "Logs"** to query specific events and create custom analytics

## Sample Queries for Canvas App Analysis

Here are some useful Kusto queries you can run in the "Logs" section of Application Insights:

### App Usage Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "AppLaunch" 
| summarize LaunchCount=count() by bin(timestamp, 1d)
| render timechart
```

### Screen Navigation Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "ScreenView" 
| extend screenName = tostring(customDimensions.screenName)
| summarize ViewCount=count() by screenName
| order by ViewCount desc
```

### Error Analysis

```kusto
exceptions
| where cloud_RoleName == "PowerApps"
| extend appName = tostring(customDimensions.appName)
| extend screenName = tostring(customDimensions.screenName)
| extend errorType = tostring(customDimensions.errorType)
| summarize ErrorCount=count() by appName, screenName, errorType
| order by ErrorCount desc
```

### Performance Analysis

```kusto
customEvents
| where cloud_RoleName == "PowerApps"
| where name == "ScreenView" 
| extend screenName = tostring(customDimensions.screenName)
| extend loadTime = todouble(customDimensions.loadTime)
| summarize avgLoadTime=avg(loadTime), p95LoadTime=percentile(loadTime, 95) by screenName
| order by avgLoadTime desc
```

## Optional: Implementing Custom Tracking

While the built-in tracking covers most scenarios, you can add custom tracking using Power Fx formulas to capture specific business events.

### Using the Trace Function for Custom Logging

The `Trace` function allows you to send custom events to Application Insights:

```
Trace(
    "BusinessEvent",            // Event name
    TraceSeverity.Information,  // Severity level
    {
        EventCategory: "OrderProcessing",  // Custom property
        OrderAmount: Text(OrderTotal),     // Custom property
        CustomerType: CustomerSegment      // Custom property
    }
)
```

### Adding Custom Tracking to Button Click

```
Button1.OnSelect = 
If(
    !IsBlank(TextInput1.Text),
    Trace(
        "OrderSubmitted", 
        TraceSeverity.Information,
        {
            OrderId: OrderID,
            Amount: OrderAmount,
            Items: CountItems
        }
    );
    // Rest of your button logic
)
```

### Using Trace for Error Handling

```
Button1.OnSelect = 
If(
    IsError(
        Set(Result, YourFunctionThatMightFail())
    ),
    Trace(
        "BusinessOperationFailed",
        TraceSeverity.Error,
        {
            OperationType: "OrderSubmission",
            ErrorDetails: Text(Error)
        }
    ),
    // Success path
)
```

## Best Practices for Canvas App Telemetry

1. **Start with the default settings** and adjust based on your monitoring needs
2. **Use Information level** for development/debugging, then switch to Warning or Error for production
3. **Set up alerts** for critical errors in your application
4. **Review telemetry regularly** to identify patterns and improvement opportunities
5. **Add custom Trace events** only for important business processes to avoid excess data
6. **Consider data privacy** - don't log personally identifiable information (PII) unless necessary

## Next Steps

Now that you've set up Application Insights tracking in your Canvas app, proceed to [Power Platform Model-Driven App Integration](./04-Model-Driven-App-Integration.md) to learn how to integrate Application Insights with model-driven apps.

## Documentation References

- [Power Platform integration with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights)
- [Monitor canvas apps with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/monitor-canvas-apps)
- [Azure Monitor Log Queries](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview)