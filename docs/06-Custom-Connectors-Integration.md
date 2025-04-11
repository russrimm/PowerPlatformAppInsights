# Integrating Application Insights with Power Platform Custom Connectors

This guide demonstrates how to integrate Azure Application Insights with Power Platform custom connectors to monitor API usage, performance, and errors.

## Overview

Custom connectors in Power Platform allow you to connect to custom APIs and use them in your Power Apps and Power Automate flows. By integrating Application Insights with your custom connectors, you can:

1. Track API usage patterns
2. Monitor performance and response times
3. Detect errors and failures
4. Analyze user behavior and trends

## Implementation Approaches

There are two main approaches to integrating Application Insights with custom connectors:

1. **API-side integration**: Add Application Insights to the API that your custom connector calls
2. **Wrapper method**: Create a wrapper around your custom connector calls in Power Automate

## Approach 1: API-Side Integration

This approach involves adding Application Insights directly to the API that your custom connector is calling.

### Step 1: Add Application Insights to Your API

If you own the API that your custom connector calls, add Application Insights to that API:

#### For .NET APIs:

1. **Install the Application Insights SDK**:
   ```
   Install-Package Microsoft.ApplicationInsights.AspNetCore
   ```

2. **Configure Application Insights in your Startup.cs**:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // Add Application Insights
       services.AddApplicationInsightsTelemetry();
       
       // Other service configurations
   }
   ```

3. **Add custom properties to API telemetry**:
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class YourController : ControllerBase
   {
       private readonly TelemetryClient _telemetryClient;
       
       public YourController(TelemetryClient telemetryClient)
       {
           _telemetryClient = telemetryClient;
       }
       
       [HttpGet]
       public IActionResult Get()
       {
           // Track custom event
           _telemetryClient.TrackEvent("CustomConnectorApiCalled", 
               new Dictionary<string, string>
               {
                   { "endpoint", "GET /api/YourController" },
                   { "clientIp", HttpContext.Connection.RemoteIpAddress.ToString() }
               });
           
           // Your API logic
           return Ok(new { result = "Success" });
       }
   }
   ```

#### For Node.js APIs:

1. **Install the Application Insights SDK**:
   ```
   npm install applicationinsights --save
   ```

2. **Configure Application Insights in your app**:
   ```javascript
   const appInsights = require('applicationinsights');
   
   appInsights.setup('YOUR_INSTRUMENTATION_KEY')
       .setAutoDependencyCorrelation(true)
       .setAutoCollectRequests(true)
       .setAutoCollectPerformance(true)
       .setAutoCollectExceptions(true)
       .setAutoCollectDependencies(true)
       .setAutoCollectConsole(true)
       .setUseDiskRetryCaching(true)
       .start();
   
   const client = appInsights.defaultClient;
   ```

3. **Add custom tracking to your API endpoints**:
   ```javascript
   app.get('/api/resource', (req, res) => {
       // Track custom event
       client.trackEvent({
           name: "CustomConnectorApiCalled",
           properties: {
               endpoint: "GET /api/resource",
               clientIp: req.ip
           }
       });
       
       // Your API logic
       res.json({ result: "Success" });
   });
   ```

### Step 2: Create Custom Dimensions for Power Platform

To identify which calls are coming from Power Platform, add custom properties to your API responses:

```javascript
app.use((req, res, next) => {
    // Check for Power Platform specific headers
    const isPowerPlatform = req.headers['x-ms-client-request-id'] ? true : false;
    const clientRequestId = req.headers['x-ms-client-request-id'] || 'unknown';
    
    // Add to Application Insights context
    if (isPowerPlatform) {
        client.trackEvent({
            name: "PowerPlatformRequest",
            properties: {
                clientRequestId: clientRequestId,
                endpoint: `${req.method} ${req.path}`
            }
        });
    }
    
    next();
});
```

## Approach 2: Wrapper Method in Power Automate

If you don't own the API or can't modify it, you can create a wrapper in Power Automate that tracks telemetry before and after calling the custom connector.

### Step 1: Create Reusable Application Insights Flows

Follow the instructions in the [Power Automate Integration](./05-Power-Automate-Integration.md) guide to create reusable Application Insights flows.

### Step 2: Create a Wrapper Flow for Your Custom Connector

1. Create a new "Instant cloud flow" in Power Automate
2. Add appropriate trigger inputs that match the inputs of your custom connector
3. Implement the wrapper pattern:

   ![Wrapper Flow](../images/connector-wrapper-flow.png)

4. Add the following actions to your flow:

   a. **Initialize variables**:
   ```
   Initialize variable 'StartTime' to @{utcNow()}
   ```

   b. **Track start of connector call**:
   - Call your "Track Event" flow with:
     - EventName: "CustomConnectorCallStarted"
     - Properties:
       ```
       {
         "connectorName": "YourConnectorName",
         "operation": "YourOperationName",
         "parameters": "@{string(triggerBody())}"
       }
       ```

   c. **Try-Catch scope around connector**:
   - Add a "Scope" action to wrap your custom connector call
   - Inside the scope, add your Custom Connector action with appropriate inputs

   d. **Calculate duration**:
   ```
   Set variable 'Duration' to @{div(sub(ticks(utcNow()), ticks(variables('StartTime'))), 10000000)}
   ```

   e. **Track successful connector call**:
   - Add an action after the scope with "Configure run after" set to run only if the scope succeeds
   - Call your "Track Dependency" flow with:
     - DependencyName: "YourConnectorName"
     - DependencyType: "CustomConnector"
     - Target: "YourConnectorOperation"
     - Success: true
     - Duration: @{variables('Duration')}
     - Properties:
       ```
       {
         "statusCode": "@{outputs('YourConnectorAction')['statusCode']}",
         "responseSize": "@{length(string(outputs('YourConnectorAction')['body']))}"
       }
       ```

   f. **Track failed connector call**:
   - Add another action with "Configure run after" set to run only if the scope fails
   - Call your "Track Exception" flow with:
     - ExceptionMessage: "@{result('YourScopeActionName')?['error']['message']}"
     - ExceptionType: "CustomConnectorError"
     - Properties:
       ```
       {
         "connectorName": "YourConnectorName",
         "operation": "YourOperationName",
         "duration": "@{variables('Duration')}"
       }
       ```

   g. **Return connector response**:
   - Add a "Response" action to return the result from your connector

### Step 3: Use Your Wrapper Flow Instead of Direct Connector

Instead of calling your custom connector directly in your flows, call your wrapper flow. This provides the Application Insights telemetry while still providing the same functionality.

## Custom Connector Policy Templates (Advanced)

For enterprise scenarios where you want to add Application Insights tracking to all custom connectors, consider using policy templates:

1. Create a policy template in API Management that includes Application Insights tracking
2. Apply this policy to all APIs that back your custom connectors

Example policy template for Application Insights tracking:

```xml
<policies>
    <inbound>
        <base />
        <set-variable name="requestStartTime" value="@(DateTime.UtcNow)" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
        <set-variable name="requestDuration" value="@((DateTime.UtcNow - (DateTime)context.Variables.GetValueOrDefault<DateTime>("requestStartTime")).TotalMilliseconds)" />
        <send-request mode="new" response-variable-name="appInsightsResponse" timeout="10" ignore-error="true">
            <set-url>https://dc.services.visualstudio.com/v2/track</set-url>
            <set-method>POST</set-method>
            <set-header name="Content-Type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-header name="X-Api-Key" exists-action="override">
                <value>YOUR_API_KEY</value>
            </set-header>
            <set-body>@{
                var body = new JObject();
                body["name"] = "Microsoft.ApplicationInsights.RemoteDependency";
                body["time"] = DateTime.UtcNow.ToString("o");
                body["iKey"] = "YOUR_INSTRUMENTATION_KEY";
                
                var tags = new JObject();
                tags["ai.cloud.role"] = "APIManagement";
                tags["ai.cloud.roleInstance"] = context.Deployment.ServiceName;
                body["tags"] = tags;
                
                var data = new JObject();
                data["baseType"] = "RemoteDependencyData";
                
                var baseData = new JObject();
                baseData["ver"] = 2;
                baseData["name"] = context.Operation.Name;
                baseData["id"] = context.RequestId;
                baseData["duration"] = ((double)context.Variables.GetValueOrDefault<double>("requestDuration")).ToString("F1");
                baseData["success"] = context.Response.StatusCode >= 200 && context.Response.StatusCode < 300;
                baseData["resultCode"] = context.Response.StatusCode.ToString();
                baseData["type"] = "HTTP";
                baseData["target"] = context.Request.Url.Host;
                
                var props = new JObject();
                props["httpMethod"] = context.Request.Method;
                props["apiPath"] = context.Operation.Name;
                props["apiProduct"] = context.Product?.Name ?? "None";
                props["subscription"] = context.Subscription?.Id ?? "None";
                props["clientIp"] = context.Request.IpAddress;
                baseData["properties"] = props;
                
                data["baseData"] = baseData;
                body["data"] = data;
                
                return body.ToString();
            }</set-body>
        </send-request>
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

## Viewing and Analyzing Connector Telemetry

After implementing Application Insights for your custom connectors, you can view and analyze the telemetry:

1. **Go to the Azure Portal** and navigate to your Application Insights resource
2. **Check "Live Metrics"** to see real-time connector usage
3. **Use "Logs"** to query specific connector metrics and analyze performance

## Sample Kusto Queries for Custom Connector Analysis

Here are some useful queries to analyze your custom connector telemetry:

### Custom Connector Usage

```kusto
customEvents
| where name == "CustomConnectorCallStarted"
| extend connectorName = tostring(customDimensions.connectorName)
| extend operation = tostring(customDimensions.operation)
| summarize count() by connectorName, operation, bin(timestamp, 1h)
| render timechart
```

### Custom Connector Performance

```kusto
dependencies
| where type == "CustomConnector"
| extend connector = tostring(target)
| extend operation = tostring(name)
| summarize avgDuration=avg(duration), p95Duration=percentile(duration, 95), count() by connector, operation
| order by avgDuration desc
```

### Custom Connector Errors

```kusto
exceptions
| where customDimensions.exceptionType == "CustomConnectorError"
| extend connectorName = tostring(customDimensions.connectorName)
| extend operation = tostring(customDimensions.operation)
| extend errorMessage = message
| summarize count() by connectorName, operation, errorMessage
| order by count_ desc
```

## Next Steps

Now that you've set up Application Insights tracking for custom connectors, proceed to [Monitoring and Analytics](./07-Monitoring-and-Analytics.md) to learn how to create comprehensive monitoring dashboards for your Power Platform solutions.

## Documentation References

- [Power Platform Custom Connectors](https://docs.microsoft.com/en-us/connectors/custom-connectors/)
- [Application Insights Data Model](https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-model)
- [API Management Policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-policies)