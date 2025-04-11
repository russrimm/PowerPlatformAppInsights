# Troubleshooting Common Issues with Application Insights Integration

This guide provides solutions for common issues when integrating Application Insights with Power Platform components.

## Overview

Even with careful implementation, you may encounter issues when integrating Application Insights with Power Platform. This guide will help you:

1. Diagnose and fix common integration issues
2. Verify telemetry is being sent correctly
3. Resolve data collection problems
4. Address visualization challenges

## Verifying Your Integration

Before diving into specific issues, use these techniques to verify that your Application Insights integration is working properly.

### Verifying Canvas App Integration

1. **Check for initialization trace**:
   ```kusto
   traces
   | where message contains "App Insights initialized in Power App"
   | order by timestamp desc
   | take 10
   ```

2. **Verify custom events are being sent**:
   ```kusto
   customEvents
   | where customDimensions has "PowerApps" 
   | order by timestamp desc
   | take 10
   ```

3. **Manual validation in your app**:
   - Add a temporary button to your app that calls your telemetry functions
   - Use the browser developer tools (F12) to check for network requests to `https://dc.services.visualstudio.com/v2/track`
   - Check for errors in the browser console

### Verifying Model-Driven App Integration

1. **Check for app initialization events**:
   ```kusto
   customEvents
   | where name == "AppLoaded" and customDimensions has "Model-Driven"
   | order by timestamp desc
   | take 10
   ```

2. **Browser validation**:
   - Open your Model-Driven app
   - Open browser developer tools (F12)
   - Go to the Network tab and filter by "track"
   - Check for requests to Application Insights endpoints

### Verifying Flow Integration

1. **Check for flow events**:
   ```kusto
   customEvents
   | where name in ("FlowStarted", "FlowCompleted", "FlowFailed")
   | order by timestamp desc
   | take 10
   ```

2. **Manual testing**:
   - Run your flow manually
   - Check Application Insights logs within 5 minutes

## Common Issues and Solutions

### Issue 1: No Telemetry Data Appearing

#### Possible causes:
- Incorrect instrumentation key
- JavaScript initialization failure
- Network connectivity issues

#### Solutions:

1. **Verify instrumentation key**:
   - Double-check the instrumentation key in your environment variables
   - Ensure the key is being correctly accessed in your code

2. **Check for JavaScript errors**:
   - Open browser developer tools (F12) when running your app
   - Look for errors in the console related to Application Insights

3. **Test network connectivity**:
   - From the device running your app, verify that `https://dc.services.visualstudio.com` is accessible
   - Check if corporate firewall policies might be blocking the connection

4. **Verify initialization code**:
   - For Canvas apps: Check that the HTML component is loading properly
   - For Model-Driven apps: Verify web resources are loading in the correct order
   - For Flows: Check HTTP action configurations

### Issue 2: Telemetry Shows in Live View But Not in Analytics

#### Possible causes:
- Indexing delay
- Query filtering issues
- Time range selection

#### Solutions:

1. **Wait for indexing**:
   - Application Insights data typically takes 2-5 minutes to be available for querying
   - For high-volume applications, it may take longer

2. **Check query filters**:
   - Ensure your query doesn't have filters that exclude your data
   - Verify time range is set appropriately

3. **Verify event properties**:
   - Use this query to see what events are actually being recorded:
     ```kusto
     union customEvents, pageViews, traces, exceptions, requests, dependencies
     | where timestamp > ago(1h)
     | order by timestamp desc
     | project timestamp, itemType, name, operation_Name, success, resultCode, duration, customDimensions
     | take 100
     ```

### Issue 3: Custom Event Properties Not Appearing

#### Possible causes:
- Property formatting issues
- Nested property objects
- Property value type issues

#### Solutions:

1. **Check property format**:
   - Properties should be simple key-value pairs
   - Property names should not contain special characters

2. **Review property values**:
   - Values should be strings, numbers, or booleans
   - Complex objects need to be stringified
   - For Canvas apps, verify the JSON structure in your helper functions

3. **Examine raw events**:
   ```kusto
   customEvents
   | where timestamp > ago(1h)
   | project timestamp, name, customDimensions
   | order by timestamp desc
   | take 20
   ```

### Issue 4: High Data Volumes and Costs

#### Possible causes:
- Excessive event tracking
- High sampling rate
- Redundant telemetry

#### Solutions:

1. **Implement sampling**:
   - For high-volume applications, consider implementing sampling
   - For Canvas apps, add a sampling function:
     ```javascript
     function shouldSample(samplingRate) {
         return Math.random() < samplingRate; // e.g., 0.1 for 10% sampling
     }
     ```

2. **Focus on important events**:
   - Avoid tracking routine events that don't provide value
   - Focus on business-critical events and errors

3. **Optimize property data**:
   - Only include necessary properties in your events
   - Avoid including large data payloads

4. **Monitor ingestion volume**:
   - Regularly check your Application Insights resource for data volume
   - Set up alerts for unexpected spikes in data ingestion

### Issue 5: Inconsistent Correlation IDs

#### Possible causes:
- Missing operation_Id values
- Incorrect implementation of correlation

#### Solutions:

1. **Implement consistent operation IDs**:
   - For Canvas apps, generate a GUID at session start and pass it to all telemetry
   - For Model-Driven apps, use the Xrm.Utility.getGlobalContext().getClientUrl() as part of the correlation ID
   - For Flows, use the workflow run ID consistently

2. **Check correlation in queries**:
   ```kusto
   union customEvents, pageViews, traces, exceptions, requests, dependencies
   | where timestamp > ago(1h)
   | order by timestamp desc
   | project timestamp, itemType, name, operation_Id, session_Id
   | take 100
   ```

### Issue 6: Power Automate Flow HTTP Action Failures

#### Possible causes:
- Incorrect endpoint URL
- Authentication issues
- Payload formatting problems

#### Solutions:

1. **Verify endpoint URL**:
   - The correct endpoint URL is `https://dc.services.visualstudio.com/v2/track`
   - Check for typos or extra spaces

2. **Check authentication headers**:
   - Ensure you're including the correct API key in the X-Api-Key header
   - Verify Content-Type is set to application/json

3. **Validate JSON payload**:
   - Use a JSON validator to check your payload format
   - Common issues include missing quotes or commas
   - Test the payload with a tool like Postman before using in Power Automate

4. **Add error handling**:
   - Configure the HTTP action to not fail on error status codes
   - Add a condition to check the HTTP action's status code

### Issue 7: Custom Connectors Not Tracking Dependencies

#### Possible causes:
- Wrapper function implementation issues
- Incorrect dependency tracking format

#### Solutions:

1. **Review wrapper implementation**:
   - Verify that start and end times are being captured correctly
   - Check that duration calculation is using consistent time units

2. **Validate dependency format**:
   - Ensure you're following the Application Insights dependency schema
   - Required fields: name, id, resultCode, duration, success, type, target

3. **Test with simplified payload**:
   - Create a simple test flow with minimal properties
   - Gradually add complexity once basic tracking works

## Canvas App-Specific Issues

### Issue: HTML Component Not Initializing

#### Solutions:

1. **Check visibility**:
   - Ensure the HTML component is visible (at least 1x1 pixel size)
   - Position it somewhere on the screen where it won't be hidden

2. **Verify HTML content**:
   - Check for JavaScript syntax errors in your HTML component
   - Ensure script tags are properly formatted

3. **Add initialization logging**:
   - Add console.log statements to your HTML component
   - Check browser console to see if code is executing

4. **Try a simpler approach**:
   - Temporarily replace complex code with a basic implementation
   - Once working, gradually add features back

## Model-Driven App-Specific Issues

### Issue: Web Resources Not Loading

#### Solutions:

1. **Check resource dependencies**:
   - Verify web resources are added to the form in correct order
   - Ensure required web resources are included in the solution

2. **Check for script errors**:
   - Use browser developer tools to identify script loading issues
   - Look for 404 errors in the network tab

3. **Verify Xrm context**:
   - Ensure Xrm namespace is available when scripts execute
   - Add checks for Xrm availability before executing code

## Power Automate-Specific Issues

### Issue: Environmental Variable Access Problems

#### Solutions:

1. **Check variable scope**:
   - Verify variables are created at the correct scope (environment vs. solution)
   - Check that variable values are set properly

2. **Use regular expressions with care**:
   - Power Automate has limitations with regex in expressions
   - Use simpler string operations when possible

3. **Handle null values**:
   - Add null checks for all environmental variables
   - Provide default values when variables aren't found

## Diagnosing Issues with Fiddler or Network Monitor

For advanced troubleshooting, you can use Fiddler or browser network tools:

1. **Set up Fiddler**:
   - Install and configure Fiddler to capture HTTPS traffic
   - Filter traffic to only show requests to dc.services.visualstudio.com

2. **Analyze request/response pairs**:
   - Check request payload formatting
   - Look for response status codes and error messages

3. **Reproduce the issue**:
   - Perform the action that should generate telemetry
   - Check if the request is sent and the response

## Next Steps

Now that you know how to troubleshoot common issues with Application Insights integration, proceed to [Best Practices](./09-Best-Practices.md) to learn how to optimize your implementation.

## Documentation References

- [Application Insights Troubleshooting](https://docs.microsoft.com/en-us/azure/azure-monitor/app/troubleshoot)
- [Application Insights Data Model](https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-model)
- [Kusto Query Language Reference](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)