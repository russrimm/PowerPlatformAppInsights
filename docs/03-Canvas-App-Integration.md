# Integrating Application Insights with Power Apps Canvas Apps

This guide demonstrates how to integrate Azure Application Insights with a Power Apps Canvas app to track custom events, errors, and user behavior.

## Overview

Canvas apps don't natively integrate with Application Insights, but we can use custom code components and JavaScript to send telemetry data. This guide will show you how to:

1. Set up a Canvas app with Application Insights integration
2. Track custom events and metrics
3. Capture and log errors
4. Monitor user behavior and performance

## Adding Application Insights to a Canvas App

### Step 1: Store the Instrumentation Key

1. **Create an environment variable** to store your Application Insights instrumentation key:
   - In the [Power Apps maker portal](https://make.powerapps.com), go to your environment
   - Navigate to "Environment Variables" under "Settings"
   - Create a new environment variable (e.g., "AppInsightsKey")
   - Set the data type to "Text" and enter your instrumentation key as the default value

   ![Environment Variable](../images/canvas-environment-variable.png)

### Step 2: Add Required Components

1. **Add a HTML text component** to your Canvas app:
   - Insert a HTML component from the "Insert" menu
   - Place it somewhere inconspicuous (you can make it 1px by 1px and hide it)
   - This will be used to load the Application Insights JavaScript SDK

2. **Create a global variable** to reference your Application Insights instrumentation key:
   - Navigate to the "App" object in the Tree view
   - Set an OnStart property to initialize a global variable:

   ```
   Set(
       gblAppInsightsKey,
       EnvironmentVariables.AppInsightsKey
   );
   ```

### Step 3: Load the Application Insights JavaScript SDK

1. **Configure the HTML component** to load the Application Insights JavaScript SDK:
   - Select the HTML component
   - Set its HTMLText property to:

   ```html
   <script type="text/javascript">
   var sdkLoaded = false;
   
   function loadAppInsights() {
       if (sdkLoaded) return;
       
       var script = document.createElement("script");
       script.src = "https://js.monitor.azure.com/scripts/b/ai.2.min.js";
       script.onload = function() {
           initAppInsights();
       };
       document.head.appendChild(script);
       sdkLoaded = true;
   }
   
   function initAppInsights() {
       // Create the SDK instance
       var instrumentationKey = window.parent.appInsightsKey;
       window.appInsights = new Microsoft.ApplicationInsights.ApplicationInsights({
           config: {
               instrumentationKey: instrumentationKey,
               enableCorsCorrelation: true,
               enableRequestHeaderTracking: true,
               enableResponseHeaderTracking: true
           }
       });
       
       // Send an initial trace message to verify setup
       window.appInsights.trackTrace({message: "App Insights initialized in Power App"});
       window.appInsights.flush();
   }
   
   // Expose functions to Power Apps
   window.trackEvent = function(eventName, properties) {
       if (!window.appInsights) return;
       window.appInsights.trackEvent({name: eventName, properties: properties});
       window.appInsights.flush();
   };
   
   window.trackException = function(error, properties) {
       if (!window.appInsights) return;
       window.appInsights.trackException({exception: new Error(error), properties: properties});
       window.appInsights.flush();
   };
   
   window.trackTrace = function(message, severity, properties) {
       if (!window.appInsights) return;
       window.appInsights.trackTrace({message: message, severity: severity, properties: properties});
       window.appInsights.flush();
   };
   
   // Initialize on load
   loadAppInsights();
   </script>
   ```

2. **Pass the instrumentation key** to the HTML component:
   - Add an App OnStart property to make the key available to the JavaScript:

   ```
   Set(
       gblAppInsightsKey, 
       EnvironmentVariables.AppInsightsKey
   );
   
   // Make key available to HTML component
   UpdateContext({
       appInsightsKey: gblAppInsightsKey
   });
   ```

### Step 4: Create Helper Functions for Tracking

1. **Create a collection of Power Fx functions** to make it easier to track events:
   - Add a new screen for your helper functions or use an existing one
   - Create the following functions using the Power Fx formula bar:

2. **TrackEvent function**:
   ```
   Set(
       TrackEvent,
       With(
           {
               eventName: eventName,
               properties: properties
           },
           Html1.PostMessage(
               JSON(
                   {
                       functionName: "trackEvent",
                       eventName: eventName,
                       properties: properties
                   }
               ),
               "*"
           )
       )
   )
   ```

3. **TrackError function**:
   ```
   Set(
       TrackError,
       With(
           {
               errorMessage: errorMessage,
               properties: properties
           },
           Html1.PostMessage(
               JSON(
                   {
                       functionName: "trackException",
                       error: errorMessage,
                       properties: properties
                   }
               ),
               "*"
           )
       )
   )
   ```

4. **TrackTrace function**:
   ```
   Set(
       TrackTrace,
       With(
           {
               message: message,
               severity: severity,
               properties: properties
           },
           Html1.PostMessage(
               JSON(
                   {
                       functionName: "trackTrace",
                       message: message,
                       severity: severity,
                       properties: properties
                   }
               ),
               "*"
           )
       )
   )
   ```

## Implementing Tracking in Your App

Now that you have set up the infrastructure, you can track various activities in your Canvas app:

### Track Button Clicks

```
Button.OnSelect = 
    TrackEvent(
        "ButtonClicked", 
        {
            "screenName": "HomeScreen",
            "buttonName": "SubmitButton",
            "userId": User().Email
        }
    );
    // Rest of your button logic
```

### Track Screen Navigation

```
Screen.OnVisible = 
    TrackEvent(
        "ScreenNavigated", 
        {
            "screenName": "DetailsScreen",
            "previousScreen": Navigate.PreviousScreen.Name,
            "timestamp": Text(Now())
        }
    );
```

### Track Form Submissions

```
SubmitForm(
    Form1
);
If(
    Form1.ErrorKind = ErrorKind.None,
    TrackEvent(
        "FormSubmitted", 
        {
            "formName": "CustomerForm",
            "recordId": Form1.LastSubmitResult.id
        }
    ),
    TrackError(
        "Form Submission Failed", 
        {
            "formName": "CustomerForm",
            "errorMessage": Form1.Error
        }
    )
)
```

### Implement Error Handling

```
If(
    IsError(YourOperationHere),
    TrackError(
        "Operation Failed", 
        {
            "operationType": "DataRetrieval",
            "errorDetails": Text(Error)
        }
    ),
    // Success handling code
)
```

## Viewing Your Application Insights Data

After implementing tracking in your Canvas app:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time telemetry as you test your app
3. **Use "Logs"** to query specific events and create custom analytics
4. **View "Application Map"** to understand the end-to-end flow of your application

## Sample Queries for Analysis

Here are some useful Kusto queries you can run in the "Logs" section of Application Insights:

### Top Button Clicks

```kusto
customEvents
| where name == "ButtonClicked" 
| summarize count() by tostring(customDimensions.buttonName)
| order by count_ desc
```

### Screen Navigation Patterns

```kusto
customEvents
| where name == "ScreenNavigated" 
| project timestamp, 
    screenName = tostring(customDimensions.screenName), 
    previousScreen = tostring(customDimensions.previousScreen)
| order by timestamp desc
```

### Error Analysis

```kusto
exceptions
| where timestamp > ago(7d)
| summarize count() by outerMessage
| top 10 by count_ desc
```

## Next Steps

Now that you've set up Application Insights tracking in your Canvas app, proceed to [Power Platform Model-Driven App Integration](./04-Model-Driven-App-Integration.md) to learn how to integrate Application Insights with model-driven apps.

## Documentation References

- [Application Insights JavaScript SDK](https://docs.microsoft.com/en-us/azure/azure-monitor/app/javascript)
- [Power Apps Canvas Apps Development](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/)
- [Azure Monitor Log Queries](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview)