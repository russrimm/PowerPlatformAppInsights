# Power Platform Integration with Microsoft Application Insights

## Training Guide

This repository contains a comprehensive training guide on integrating Power Platform solutions with Microsoft Application Insights. This guide is designed to help you understand how to monitor, track, and gain insights into your Power Platform applications using Azure Application Insights.

## Table of Contents

1. [Introduction to Application Insights](#introduction-to-application-insights)
2. [Prerequisites](./docs/01-Prerequisites.md)
3. [Setting Up Application Insights](./docs/02-Setting-Up-Application-Insights.md)
4. [Power Platform Canvas App Integration](./docs/03-Canvas-App-Integration.md)
5. [Power Platform Model-Driven App Integration](./docs/04-Model-Driven-App-Integration.md)
6. [Power Automate Flow Integration](./docs/05-Power-Automate-Integration.md)
7. [Custom Connectors Integration](./docs/06-Custom-Connectors-Integration.md)
8. [Monitoring and Analytics](./docs/07-Monitoring-and-Analytics.md)
9. [Troubleshooting Common Issues](./docs/08-Troubleshooting-Common-Issues.md)
10. [Best Practices](./docs/09-Best-Practices.md)
11. [Additional Resources](#additional-resources)

## Introduction to Application Insights

Microsoft Application Insights is an extensible Application Performance Management (APM) service that helps you monitor your live applications. It automatically detects performance anomalies and includes powerful analytics tools to help you diagnose issues and understand what users actually do with your app.

Integrating Application Insights with Power Platform solutions allows you to:

- Track custom events and metrics
- Monitor application performance
- Capture and analyze telemetry data
- Set up alerts for specific conditions
- Create custom dashboards for visualization

## What is Application Insights?

Application Insights is an extension of Azure Monitor and provides application performance monitoring (APM) features. It's a service that monitors your live applications, automatically detects performance anomalies, and includes powerful analytics tools to help you diagnose issues and understand what users actually do with your apps.

As part of Azure Monitor, Application Insights offers OpenTelemetry integration, providing a vendor-neutral approach to collecting and analyzing telemetry data across different platforms, enabling comprehensive observability of your applications.

### Key capabilities of Application Insights:

- **Live metrics and real-time monitoring**: Observe your application's live metrics in real-time with sub-second latency to help diagnose issues as they happen
- **Distributed tracing**: Track requests and dependencies across application components, helping you identify performance bottlenecks and failure points
- **Application Map**: Visual representation showing the topology of your application, making it easier to identify performance bottlenecks or failure hotspots
- **Availability monitoring**: Set up web tests to continuously test your application's availability and responsiveness from various locations globally
- **Usage analysis**: Understand user behavior, including which features are most popular and how users navigate your application
- **Smart detection and alerts**: Automatically detect performance anomalies and failures with built-in machine learning capabilities
- **Integration with DevOps tools**: Seamlessly integrate with Azure DevOps, GitHub, Power Platform, and other development tools
- **Open source SDK**: Use open-source SDKs for popular platforms that integrate with your CI/CD processes
- **Exportability and extensibility**: Export data to storage or build custom extensions to customize the monitoring experience

### Key investigation tools:

- **Application dashboard**: An at-a-glance assessment of your application's health and performance
- **Live metrics**: A real-time analytics dashboard for insight into application activity and performance
- **Transaction search**: Trace and diagnose transactions to identify issues and optimize performance
- **Availability view**: Proactively monitor and test the availability and responsiveness of application endpoints
- **Failures view**: Identify and analyze failures in your application to minimize downtime
- **Performance view**: Review application performance metrics and potential bottlenecks

### Key metrics Application Insights collects:

- **Request rates, response times, and failure rates** - Find out which pages are most popular, at what times of day, and where your users are
- **Dependency rates, response times, and failure rates** - Determine if external services are slowing you down
- **Exceptions** - Analyze the aggregated statistics, or pick specific instances and drill into the stack trace and related requests
- **Page views and load performance** - Reported by your users' browsers
- **AJAX calls** from web pages - rates, response times, and failure rates
- **User and session counts**
- **Performance counters** from your Windows or Linux server machines, such as CPU, memory, and network usage

### Data Collection, Retention and Storage:

- Data is sent to an Application Insights Log Analytics workspace
- You can choose the retention period for raw data, from 30 to 730 days
- Aggregated data is retained for 90 days, and debug snapshots for 15 days
- Most Application Insights data has a latency of under 5 minutes

### Supported Languages and Platforms:

Application Insights supports various application types and languages:

- **Automatic instrumentation** without code changes for several environments
- **OpenTelemetry Distro** for ASP.NET Core, .NET, Java, Node.js, and Python
- **Client-side JavaScript SDK** including support for React, React Native, and Angular
- **Azure service integration** with Virtual Machines, App Service, Functions, Spring Apps, and Cloud Services

By integrating Application Insights with Power Platform, you gain deep visibility into how your applications are performing and being used, enabling you to make data-driven decisions about improvements and troubleshooting.

## Getting Started

To begin your implementation, navigate to the [Prerequisites](./docs/01-Prerequisites.md) section to ensure you have everything needed.

## Additional Resources

- [Official Microsoft Application Insights Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/)
- [Azure Portal](https://portal.azure.com)