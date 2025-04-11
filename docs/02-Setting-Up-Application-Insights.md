# Setting Up Microsoft Application Insights

This guide will walk you through the process of setting up Microsoft Application Insights for use with Power Platform applications.

## Creating an Application Insights Resource

1. **Sign in to the Azure Portal**:
   - Navigate to [https://portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure account

2. **Create Application Insights Resource**:
   - Click on "Create a resource"
   - Search for "Application Insights" and select it
   - Click "Create"

   ![Create Application Insights](../images/create-app-insights.png)

3. **Configure Application Insights settings**:
   - **Subscription**: Select your Azure subscription
   - **Resource Group**: Create new or select existing
   - **Name**: Enter a unique name (e.g., "PowerPlatformMonitoring")
   - **Region**: Select a region close to your users
   - **Resource Mode**: Select "Workspace-based"
   - **Log Analytics Workspace**: Create new or select existing
   - Click "Review + create" and then "Create"

   ![Configure Application Insights](../images/configure-app-insights.png)

4. **Access Your Application Insights Resource**:
   - Once deployment is complete, click "Go to resource"
   - Take note of the following information that you'll need later:
     - **Instrumentation Key**: Found under "Configure > Properties"
     - **Connection String**: Found under "Configure > Properties"

   ![Application Insights Overview](../images/app-insights-overview.png)

## Securing Your Instrumentation Key (Optional but Recommended)

For production environments, it's recommended to store your Application Insights instrumentation key in Azure Key Vault:

1. **Create an Azure Key Vault**:
   - In the Azure Portal, create a new Key Vault resource
   - Store the instrumentation key as a secret

2. **Set up access policies**:
   - Configure appropriate access policies for your development team

## Understanding Application Insights Components

Before integrating with Power Platform, familiarize yourself with key Application Insights concepts:

- **Telemetry Data Types**:
  - **Traces**: Diagnostic logs
  - **Requests**: HTTP requests to your application
  - **Exceptions**: Tracked exceptions and crashes
  - **Dependencies**: Calls to external services
  - **Events**: Custom business events
  - **Metrics**: Performance measurements

- **Live Metrics**: Real-time monitoring dashboard
- **Application Map**: Visual representation of component dependencies
- **Availability Tests**: Automated tests to verify your application is running

## Next Steps

Now that you've set up Application Insights, proceed to [Power Platform Canvas App Integration](./03-Canvas-App-Integration.md) to learn how to integrate this with your Power Apps canvas applications.

## Documentation References

- [Application Insights Overview](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Create an Application Insights Resource](https://docs.microsoft.com/en-us/azure/azure-monitor/app/create-new-resource)
- [Azure Key Vault Documentation](https://docs.microsoft.com/en-us/azure/key-vault/)