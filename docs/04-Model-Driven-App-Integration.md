# Integrating Application Insights with Power Apps Model-Driven Apps

This guide demonstrates how to integrate Azure Application Insights with a Power Apps Model-Driven app to track custom events, errors, and user behaviors.

## Overview

Model-Driven apps provide more integration options with Application Insights compared to Canvas apps, as we can leverage the form scripting and client API to implement telemetry tracking. This guide will show you how to:

1. Set up a Model-Driven app with Application Insights integration
2. Track custom events and metrics using form scripts
3. Implement global error handling
4. Monitor user behavior and performance
5. Use ribbon workbench for custom button tracking

## Adding Application Insights to a Model-Driven App

### Step 1: Store the Instrumentation Key

1. **Create an environment variable** to store your Application Insights instrumentation key:
   - In the [Power Apps maker portal](https://make.powerapps.com), go to your environment
   - Navigate to "Environment Variables" under "Settings"
   - Create a new environment variable (e.g., "AppInsightsKey")
   - Set the data type to "Text" and enter your instrumentation key as the default value

   ![Environment Variable](../images/model-environment-variable.png)

### Step 2: Create a Web Resource for Application Insights

1. **Create a JavaScript web resource** to load the Application Insights SDK:
   - In the Power Apps maker portal, go to "Solutions" and open your solution
   - Navigate to "Web Resources" and click "New"
   - Set the "Name" (e.g., "new_applicationinsights.js")
   - Set "Type" to "JavaScript (JScript)"
   - Enter the following code:

   ```javascript
   // Application Insights Integration for Model-Driven Apps
   var AppInsights = {
       instrumentationKey: null,
       isInitialized: false,
       client: null,
       
       init: function(instrumentationKey) {
           if (!instrumentationKey) {
               console.error("No instrumentation key provided for Application Insights");
               return;
           }
           
           if (this.isInitialized) {
               return;
           }
           
           // Load the Application Insights script
           var scriptElement = document.createElement("script");
           scriptElement.src = "https://js.monitor.azure.com/scripts/b/ai.2.min.js";
           scriptElement.onload = function() {
               AppInsights.instrumentationKey = instrumentationKey;
               AppInsights.client = new Microsoft.ApplicationInsights.ApplicationInsights({
                   config: {
                       instrumentationKey: instrumentationKey,
                       enableAutoRouteTracking: true
                   }
               });
               
               // Set up global error handler
               window.addEventListener("error", function(errorEvent) {
                   AppInsights.trackException(errorEvent.error || {
                       message: errorEvent.message,
                       stack: "No stack, window.onerror event"
                   });
               });
               
               // Track initial load
               AppInsights.trackEvent("AppLoaded", {
                   appName: Xrm.Utility.getGlobalContext().userSettings.appName,
                   userId: Xrm.Utility.getGlobalContext().userSettings.userId
               });
               
               AppInsights.isInitialized = true;
               console.log("Application Insights initialized successfully");
           };
           
           document.head.appendChild(scriptElement);
       },
       
       trackEvent: function(name, properties) {
           if (!this.isInitialized || !this.client) {
               console.warn("Application Insights not initialized. Cannot track event: " + name);
               return;
           }
           
           // Add standard properties to all events
           var standardProps = {
               timestamp: new Date().toISOString(),
               appName: Xrm.Utility.getGlobalContext().userSettings.appName,
               userId: Xrm.Utility.getGlobalContext().userSettings.userId,
               orgName: Xrm.Utility.getGlobalContext().organizationSettings.uniqueName
           };
           
           // Combine custom properties with standard properties
           var allProperties = Object.assign({}, standardProps, properties || {});
           
           // Track the event
           this.client.trackEvent({name: name, properties: allProperties});
           this.client.flush();
       },
       
       trackException: function(exception, properties) {
           if (!this.isInitialized || !this.client) {
               console.error("Application Insights not initialized. Cannot track exception");
               return;
           }
           
           // Add standard properties
           var standardProps = {
               timestamp: new Date().toISOString(),
               appName: Xrm.Utility.getGlobalContext().userSettings.appName,
               userId: Xrm.Utility.getGlobalContext().userSettings.userId,
               orgName: Xrm.Utility.getGlobalContext().organizationSettings.uniqueName
           };
           
           // Combine custom properties with standard properties
           var allProperties = Object.assign({}, standardProps, properties || {});
           
           // Track the exception
           this.client.trackException({exception: exception, properties: allProperties});
           this.client.flush();
       },
       
       trackPageView: function(pageName, properties) {
           if (!this.isInitialized || !this.client) {
               console.warn("Application Insights not initialized. Cannot track page view: " + pageName);
               return;
           }
           
           // Add standard properties
           var standardProps = {
               timestamp: new Date().toISOString(),
               appName: Xrm.Utility.getGlobalContext().userSettings.appName,
               userId: Xrm.Utility.getGlobalContext().userSettings.userId,
               orgName: Xrm.Utility.getGlobalContext().organizationSettings.uniqueName
           };
           
           // Combine custom properties with standard properties
           var allProperties = Object.assign({}, standardProps, properties || {});
           
           // Track the page view
           this.client.trackPageView({name: pageName, properties: allProperties});
           this.client.flush();
       }
   };
   ```

2. **Save and publish** the web resource

### Step 3: Create a Utility Web Resource for Environment Variables

1. **Create a JavaScript web resource** to access environment variables:
   - In the Power Apps maker portal, create another web resource
   - Set the "Name" (e.g., "new_environmentvariables.js")
   - Set "Type" to "JavaScript (JScript)"
   - Enter the following code:

   ```javascript
   var EnvironmentVariables = {
       cache: {},
       
       getValue: function(schemaName) {
           var _this = this;
           
           return new Promise(function(resolve, reject) {
               // Check cache first
               if (_this.cache[schemaName] !== undefined) {
                   resolve(_this.cache[schemaName]);
                   return;
               }
               
               // Not in cache, retrieve from environment variable
               Xrm.WebApi.retrieveMultipleRecords("environmentvariabledefinition", 
                   "?$filter=schemaname eq '" + schemaName + "'&$expand=environmentvariabledefinition_environmentvariablevalue")
                   .then(function(result) {
                       if (result.entities.length > 0 && 
                           result.entities[0].environmentvariabledefinition_environmentvariablevalue && 
                           result.entities[0].environmentvariabledefinition_environmentvariablevalue.length > 0) {
                           
                           var value = result.entities[0].environmentvariabledefinition_environmentvariablevalue[0].value;
                           _this.cache[schemaName] = value;
                           resolve(value);
                       } else {
                           // Not found or no value
                           _this.cache[schemaName] = null;
                           resolve(null);
                       }
                   })
                   .catch(function(error) {
                       console.error("Error retrieving environment variable: " + schemaName, error);
                       reject(error);
                   });
           });
       }
   };
   ```

2. **Save and publish** the web resource

### Step 4: Create a Configuration Web Resource for Application Insights

1. **Create a JavaScript web resource** to initialize Application Insights with your environment variable:
   - In the Power Apps maker portal, create another web resource
   - Set the "Name" (e.g., "new_appinsights_config.js")
   - Set "Type" to "JavaScript (JScript)"
   - Enter the following code:

   ```javascript
   function initializeAppInsights() {
       // Get the Application Insights key from environment variables
       EnvironmentVariables.getValue("AppInsightsKey")
           .then(function(instrumentationKey) {
               if (instrumentationKey) {
                   // Initialize Application Insights with the key
                   AppInsights.init(instrumentationKey);
               } else {
                   console.warn("Application Insights key not found in environment variables");
               }
           })
           .catch(function(error) {
               console.error("Failed to initialize Application Insights", error);
           });
   }

   // Initialize when the form loads
   document.addEventListener("DOMContentLoaded", function() {
       initializeAppInsights();
   });
   ```

2. **Save and publish** the web resource

### Step 5: Include Web Resources in Your Model-Driven App

1. **Add web resources to your solution**:
   - Add all three web resources to your solution if they aren't already included

2. **Add web resources to your app**:
   - Open your Model-Driven app for editing
   - Go to the app designer
   - Under "Components", navigate to "Web Resources"
   - Add all three web resources to your app
   - Save and publish the app

3. **Include web resources in form load**:
   - Open the entity form where you want to add tracking
   - Go to "Form Properties"
   - Navigate to "Events" tab
   - Under "Form Libraries", add your three web resources in this order:
     1. new_environmentvariables.js
     2. new_applicationinsights.js
     3. new_appinsights_config.js
   - Save and publish the form

## Implementing Tracking in Your Model-Driven App

Now that the infrastructure is set up, you can track various activities in your Model-Driven app:

### Track Form Events

1. **Track form load events**:
   - In the form editor, go to "Form Properties" > "Events"
   - On the "Form OnLoad" event, add a JavaScript function:

   ```javascript
   function trackFormLoad(executionContext) {
       try {
           var formContext = executionContext.getFormContext();
           var entityName = formContext.data.entity.getEntityName();
           var recordId = formContext.data.entity.getId();
           var formType = formContext.ui.getFormType(); // 1=Create, 2=Update, 3=Read Only, etc.
           var formTypeLabel = "";
           
           switch (formType) {
               case 1: formTypeLabel = "Create"; break;
               case 2: formTypeLabel = "Update"; break;
               case 3: formTypeLabel = "Read Only"; break;
               default: formTypeLabel = "Form Type: " + formType;
           }
           
           // Track form load event in Application Insights
           if (typeof AppInsights !== "undefined") {
               AppInsights.trackPageView(
                   entityName + " - " + formTypeLabel,
                   {
                       entityName: entityName,
                       recordId: recordId,
                       formType: formTypeLabel
                   }
               );
           }
       } catch (e) {
           console.error("Error in trackFormLoad", e);
       }
   }
   ```

2. **Track form save events**:
   - In the form editor, on the "Form OnSave" event, add a JavaScript function:

   ```javascript
   function trackFormSave(executionContext) {
       try {
           var formContext = executionContext.getFormContext();
           var entityName = formContext.data.entity.getEntityName();
           var recordId = formContext.data.entity.getId();
           var saveMode = executionContext.getEventArgs().getSaveMode();
           var saveModeLabel = "";
           
           // Determine save mode type
           switch (saveMode) {
               case 1: saveModeLabel = "Save"; break;
               case 2: saveModeLabel = "Save and Close"; break;
               case 5: saveModeLabel = "Save and New"; break;
               default: saveModeLabel = "Save Mode: " + saveMode;
           }
           
           // Track form save event in Application Insights
           if (typeof AppInsights !== "undefined") {
               AppInsights.trackEvent(
                   "FormSave",
                   {
                       entityName: entityName,
                       recordId: recordId, 
                       saveMode: saveModeLabel
                   }
               );
           }
       } catch (e) {
           console.error("Error in trackFormSave", e);
       }
   }
   ```

### Track Field Changes

1. **Track field value changes**:
   - In the form editor, select a field
   - In the "Properties" pane, under "Events", add an "OnChange" event handler:

   ```javascript
   function trackFieldChange(executionContext) {
       try {
           var formContext = executionContext.getFormContext();
           var entityName = formContext.data.entity.getEntityName();
           var recordId = formContext.data.entity.getId();
           
           // Get the changed attribute
           var attribute = executionContext.getEventSource();
           var attributeName = attribute.getName();
           var oldValue = attribute.getInitialValue();
           var newValue = attribute.getValue();
           
           // Convert values to strings for tracking
           oldValue = oldValue !== null ? oldValue.toString() : "null";
           newValue = newValue !== null ? newValue.toString() : "null";
           
           // Don't track if values haven't changed
           if (oldValue === newValue) {
               return;
           }
           
           // Track field change event in Application Insights
           if (typeof AppInsights !== "undefined") {
               AppInsights.trackEvent(
                   "FieldChanged",
                   {
                       entityName: entityName,
                       recordId: recordId,
                       fieldName: attributeName,
                       oldValue: oldValue,
                       newValue: newValue
                   }
               );
           }
       } catch (e) {
           console.error("Error in trackFieldChange", e);
       }
   }
   ```

### Track Business Process Flow Stage Changes

```javascript
function trackBPFStageChange(executionContext) {
    try {
        var formContext = executionContext.getFormContext();
        var entityName = formContext.data.entity.getEntityName();
        var recordId = formContext.data.entity.getId();
        
        // Get BPF info
        var processId = formContext.data.process.getSelectedProcess().getId();
        var processName = formContext.data.process.getSelectedProcess().getName();
        var stageId = formContext.data.process.getSelectedStage().getId();
        var stageName = formContext.data.process.getSelectedStage().getName();
        
        // Track BPF stage change event in Application Insights
        if (typeof AppInsights !== "undefined") {
            AppInsights.trackEvent(
                "BPFStageChanged",
                {
                    entityName: entityName,
                    recordId: recordId,
                    processId: processId,
                    processName: processName,
                    stageId: stageId,
                    stageName: stageName
                }
            );
        }
    } catch (e) {
        console.error("Error in trackBPFStageChange", e);
    }
}
```

## Ribbon Button Tracking

To track ribbon button clicks, you can use the Ribbon Workbench to customize ribbon buttons and add telemetry tracking:

1. **Install the Ribbon Workbench** from [https://www.develop1.net/public/rwb/ribbonworkbench.aspx](https://www.develop1.net/public/rwb/ribbonworkbench.aspx)

2. **Open the Ribbon Workbench** for your solution

3. **Edit a button's command** and add JavaScript to track button clicks:

   ```javascript
   // Track the button click
   if (typeof AppInsights !== "undefined") {
       AppInsights.trackEvent(
           "RibbonButtonClicked",
           {
               entityName: entityName,
               buttonName: "New_CustomButton",
               recordId: recordId
           }
       );
   }
   
   // Then continue with the original button action
   ```

## Viewing Your Application Insights Data

After implementing tracking in your Model-Driven app:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time telemetry as you test your app
3. **Use "Logs"** to query specific events and create custom analytics

## Sample Queries for Analysis

Here are some useful Kusto queries you can run in the "Logs" section of Application Insights:

### Most Frequently Viewed Forms

```kusto
customEvents
| where name == "FormLoaded"
| summarize count() by tostring(customDimensions.entityName), tostring(customDimensions.formType)
| order by count_ desc
```

### Most Common Form Saves

```kusto
customEvents
| where name == "FormSave"
| summarize count() by tostring(customDimensions.entityName), tostring(customDimensions.saveMode)
| order by count_ desc
```

### Most Frequently Changed Fields

```kusto
customEvents
| where name == "FieldChanged"
| summarize count() by tostring(customDimensions.entityName), tostring(customDimensions.fieldName)
| order by count_ desc
```

### BPF Stage Analysis

```kusto
customEvents
| where name == "BPFStageChanged"
| summarize count() by tostring(customDimensions.processName), tostring(customDimensions.stageName)
| order by count_ desc
```

## Next Steps

Now that you've set up Application Insights tracking in your Model-Driven app, proceed to [Power Automate Flow Integration](./05-Power-Automate-Integration.md) to learn how to integrate Application Insights with Power Automate.

## Documentation References

- [Model-driven apps documentation](https://docs.microsoft.com/en-us/powerapps/maker/model-driven-apps/)
- [Client API reference for model-driven apps](https://docs.microsoft.com/en-us/powerapps/developer/model-driven-apps/clientapi/reference)
- [Application Insights JavaScript SDK](https://docs.microsoft.com/en-us/azure/azure-monitor/app/javascript)