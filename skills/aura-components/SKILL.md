---
name: aura-components
description: Expert Aura Lightning Component guidance. Use when working with existing Aura components (.cmp), JavaScript controllers/helpers, Apex integration via $A.enqueueAction, Chart.js in Aura, and event handling. Specific to the DB_InternalWholeSaler_Cmp_2025 component.
---

# Aura Components

## Project Context

- **Main Component**: `DB_InternalWholeSaler_Cmp_2025` (~9,800 lines)
- **API Version**: 65.0
- **Chart Library**: Chart.js via `ChartJS_1` static resource
- **Controller**: `DB_InternalWholeSaler_Controller_2024` (4,185 lines)

## Component Structure

```
force-app/main/default/aura/DB_InternalWholeSaler_Cmp_2025/
├── DB_InternalWholeSaler_Cmp_2025.cmp              # Markup
├── DB_InternalWholeSaler_Cmp_2025Controller.js     # UI event handlers
├── DB_InternalWholeSaler_Cmp_2025Helper.js         # Business logic
├── DB_InternalWholeSaler_Cmp_2025Renderer.js       # Custom rendering
├── DB_InternalWholeSaler_Cmp_2025.css              # Styles
└── DB_InternalWholeSaler_Cmp_2025.cmp-meta.xml     # Metadata
```

## Component Markup (.cmp)

```xml
<aura:component controller="DB_InternalWholeSaler_Controller_2024"
                implements="flexipage:availableForAllPageTypes">

    <!-- Attributes -->
    <aura:attribute name="chartData" type="Object" />
    <aura:attribute name="isLoading" type="Boolean" default="false" />
    <aura:attribute name="selectedMonth" type="Integer" />

    <!-- Handlers -->
    <aura:handler name="init" value="{!this}" action="{!c.doInit}" />
    <aura:handler name="render" value="{!this}" action="{!c.onRender}" />

    <!-- Static Resource -->
    <ltng:require scripts="{!$Resource.ChartJS_1}"
                  afterScriptsLoaded="{!c.scriptsLoaded}" />

    <!-- Embedded LWC -->
    <c:lwcUsageMetricsService aura:id="metricsService" />

    <!-- Chart Canvas -->
    <canvas aura:id="myChart" class="chart-canvas"></canvas>
</aura:component>
```

## Controller (.js) - UI Events Only

```javascript
({
    doInit: function(component, event, helper) {
        helper.loadInitialData(component);
    },

    scriptsLoaded: function(component, event, helper) {
        helper.initializeCharts(component);
    },

    handleFilterChange: function(component, event, helper) {
        var selectedValue = event.getSource().get("v.value");
        component.set("v.selectedFilter", selectedValue);
        helper.refreshChartData(component);
    },

    handleMonthChange: function(component, event, helper) {
        var month = component.get("v.selectedMonth");
        helper.loadMonthData(component, month);
    }
})
```

## Helper (.js) - Business Logic & Apex Calls

```javascript
({
    loadInitialData: function(component) {
        component.set("v.isLoading", true);

        var action = component.get("c.getOnLoadRecords");
        action.setParams({
            "month": component.get("v.selectedMonth"),
            "year": component.get("v.selectedYear")
        });

        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                var data = response.getReturnValue();
                component.set("v.chartData", data);
                this.renderChart(component, data);
            } else if (state === "ERROR") {
                this.handleErrors(component, response.getError());
            }
            component.set("v.isLoading", false);
        });

        $A.enqueueAction(action);
    },

    renderChart: function(component, data) {
        var ctx = component.find("myChart").getElement().getContext("2d");

        // Destroy existing chart if present
        if (component.get("v.chartInstance")) {
            component.get("v.chartInstance").destroy();
        }

        var chart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: data.labels,
                datasets: data.datasets
            },
            options: this.getChartOptions(component)
        });

        component.set("v.chartInstance", chart);
    },

    getChartOptions: function(component) {
        return {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { beginAtZero: true }
            },
            plugins: {
                legend: { position: 'bottom' }
            }
        };
    },

    handleErrors: function(component, errors) {
        var message = 'Unknown error';
        if (errors && errors[0] && errors[0].message) {
            message = errors[0].message;
        }
        console.error('Error:', message);
        this.showToast(component, 'Error', message, 'error');
    },

    showToast: function(component, title, message, type) {
        var toastEvent = $A.get("e.force:showToast");
        toastEvent.setParams({
            title: title,
            message: message,
            type: type
        });
        toastEvent.fire();
    }
})
```

## Apex Controller Integration

```javascript
// Calling @AuraEnabled methods
var action = component.get("c.methodName");
action.setParams({
    "param1": value1,
    "param2": value2
});

action.setCallback(this, function(response) {
    var state = response.getState();
    if (state === "SUCCESS") {
        // Handle success
        var result = response.getReturnValue();
    } else if (state === "ERROR") {
        // Handle error
        var errors = response.getError();
    }
});

$A.enqueueAction(action);
```

## Component Events

```javascript
// Fire component event
var cmpEvent = component.getEvent("onDataChange");
cmpEvent.setParams({ "data": payload });
cmpEvent.fire();

// Handle in parent
<c:ChildComponent onDataChange="{!c.handleDataChange}" />
```

## Application Events

```javascript
// Fire application event
var appEvent = $A.get("e.c:MyAppEvent");
appEvent.setParams({ "recordId": recordId });
appEvent.fire();

// Register handler
<aura:handler event="c:MyAppEvent" action="{!c.handleAppEvent}" />
```

## Aura-LWC Interop

```xml
<!-- Embed LWC in Aura -->
<c:lwcUsageMetricsService aura:id="metricsService" />
```

```javascript
// Call LWC method from Aura Helper
var lwcComponent = component.find("metricsService");
lwcComponent.trackUsage(eventData);
```

## Common Patterns in This Project

### Iteration
```xml
<aura:iteration items="{!v.items}" var="item" indexVar="idx">
    <li data-index="{!idx}">{!item.name}</li>
</aura:iteration>
```

### Conditional Rendering
```xml
<aura:if isTrue="{!v.showSection}">
    <div>Visible content</div>
    <aura:set attribute="else">
        <div>Hidden content</div>
    </aura:set>
</aura:if>
```

### Dynamic Class
```xml
<div class="{!v.isActive ? 'slds-is-active' : ''}">
```

## Async Callback Pattern

```javascript
// Use $A.getCallback() for async operations outside Aura context
setTimeout($A.getCallback(function() {
    component.set("v.value", newValue);
}), 1000);

// Check component validity in callbacks
action.setCallback(this, function(response) {
    if (component.isValid()) {
        // Safe to update component
    }
});
```

## Chart.js Gotchas

1. **Destroy before recreate**: Always call `chart.destroy()` before creating new chart
2. **Scripts loaded handler**: Initialize charts only after `afterScriptsLoaded` fires
3. **Canvas reference**: Use `component.find("auraId").getElement()` to get canvas
4. **Global Chart object**: Chart.js creates global `Chart` object after loading

## Debug Tips

```javascript
// Log component state
console.log('Component:', JSON.stringify(component.get("v.attributeName")));

// Check if component is valid
console.log('Is valid:', component.isValid());

// Inspect action state
console.log('Action state:', response.getState());
console.log('Action errors:', response.getError());
```
