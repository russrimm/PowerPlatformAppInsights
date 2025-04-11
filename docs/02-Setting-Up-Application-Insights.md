# Setting Up Microsoft Application Insights

## Creating an Application Insights Resource

1. **Sign in to the Azure Portal**:
   - Navigate to [https://portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure account

2. **Create Application Insights Resource**:
   - Click on "Create a resource"
   - Search for "Application Insights" and select it
   - Click "Create"

3. **Configure Application Insights settings**:
   - **Subscription**: Select your Azure subscription
   - **Resource Group**: Create new or select existing
   - **Name**: Enter a unique name (e.g., "PowerPlatformMonitoring")
   - **Region**: Select a region close to your users
   - **Log Analytics Workspace**: Create new or select existing
   - Click "Review + create" and then "Create"

4. **Access Your Application Insights Resource**:
   - Once deployment is complete, click "Go to resource"
   - Take note of the following information that you'll need later:
     - **Connection String**: Found under "Configure > Properties"

## Configuring Continuous Export from Power Platform

Power Platform offers built-in capabilities to continuously export telemetry data to your Application Insights resource. This enables centralized monitoring and better integration with your Azure monitoring ecosystem. Follow these steps to set it up:

### Step 1: Prerequisites

Before configuring export, ensure you have:

1. **Admin access** to the Power Platform environment
2. **Managed Environment** Enable the environment(s) to be monitored as Managed Environment(s)
3. **An Application Insights resource** (created in the steps above)
4. **The Application Insights connection string** From the **Overview** Pane of the App Insights Resource

### Step 2: Enable Export in Power Platform Admin Center

1. **Go to the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)**.
2. **Select "Data export"** in the navigation pane.
3. **Go to the "App Insights" tab** and select "New data export".
4. **Provide a friendly name** for the export package to identify the Azure Application Insights instance.
5. **Select the specific data types** to export, such as:
   - Dataverse diagnostics and performance
   - Power Automate (e.g., cloud flow runs, triggers, actions)
6. **Apply filters** to view specific, filtered data (optional).
7. **Select the environment** to export data from and the Azure subscription, resource group, and Application Insights environment to export data to.
8. **Review the details** and select "Create" to set up the data export connection.

### Step 3: Configure Export Scope

1. **Select the data types** to monitor, such as:
   - Power Apps: Usage and errors from canvas and model-driven apps
   - Power Automate: Cloud flow execution data
   - Dataverse diagnostics and performance: Entity operation telemetry
   - Dynamics Customer Service: Usage and performance data for Dynamics
2. **Configure sampling rate** (optional):
   - Default is 100% (all telemetry).
   - Can be reduced to handle high-volume environments.

### Step 4: Verify Data Export

1. **Wait up to 24 hours** for the first data to be exported.
2. **Open your Application Insights resource** in Azure Portal.
3. **Navigate to Logs** and run a simple query:

```kusto
customEvents
| where cloud_RoleName startswith "PowerPlatform"
| take 10
```

4. **Check for data** from the Power Platform environment.

### Step 5: Set Up Data Retention (Optional)

By default, Application Insights retains data for 90 days. To modify this:

1. **In Azure Portal**, go to your **Log Analytics workspace**
2. **Navigate to Usage and estimated costs > Data Retention**
3. **Adjust the retention period** (up to 730 days)
4. **Click "Save"**

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

Now that you've set up Application Insights and configured continuous export from Power Platform, proceed to [Power Platform Canvas App Integration](./03-Canvas-App-Integration.md) to learn how to integrate this with your Power Apps canvas applications.

## Documentation References

- [Application Insights Overview](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Create an Application Insights Resource](https://docs.microsoft.com/en-us/azure/azure-monitor/app/create-new-resource)
- [Set up export to Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/set-up-export-application-insights)
- [Power Platform integration with Application Insights](https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights)
- [Azure Key Vault Documentation](https://docs.microsoft.com/en-us/azure/key-vault/)
