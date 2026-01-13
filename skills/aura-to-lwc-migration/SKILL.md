---
name: aura-to-lwc-migration
description: Guide for migrating Aura components to Lightning Web Components. Use when converting Aura markup, JavaScript controllers/helpers, event handling, or Apex integration patterns to LWC equivalents.
---

# Aura to LWC Migration

## Project Context

- **Source**: `DB_InternalWholeSaler_Cmp_2025` (Aura, ~9,800 lines)
- **Target**: Modern LWC with modular architecture
- **API Version**: 65.0

## File Mapping

| Aura File | LWC Equivalent |
|-----------|----------------|
| `component.cmp` | `component.html` |
| `componentController.js` | `component.js` (merged) |
| `componentHelper.js` | `component.js` (merged) |
| `component.css` | `component.css` (scoped) |
| `componentRenderer.js` | `renderedCallback()` |

## Attribute to Property

**Aura:**
```xml
<aura:attribute name="userId" type="String" default=""/>
<aura:attribute name="showSpinner" type="Boolean" default="false"/>
<aura:attribute name="userList" type="List" default="[]"/>
```

**LWC:**
```javascript
import { LightningElement, api, track } from 'lwc';

export default class MyComponent extends LightningElement {
    @api userId = '';           // Public reactive
    showSpinner = false;        // Private reactive (primitives auto-tracked)
    @track userList = [];       // @track for mutating objects/arrays
}
```

## Event Handling

**Aura:**
```xml
<aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
<lightning:button onclick="{!c.handleClick}" label="Click"/>
```
```javascript
// Controller
handleClick: function(component, event, helper) {
    helper.processClick(component, event);
}
// Helper
processClick: function(component, event) {
    var userId = component.get("v.userId");
}
```

**LWC:**
```javascript
export default class MyComponent extends LightningElement {
    @api userId;

    connectedCallback() {
        // Replaces aura:handler name="init"
        this.initialize();
    }

    handleClick(event) {
        // Direct handler, use this.userId
        console.log(this.userId);
    }
}
```

## Apex Integration

**Aura:**
```javascript
var action = component.get("c.getRecords");
action.setParams({ "userId": component.get("v.userId") });
action.setCallback(this, function(response) {
    if (response.getState() === "SUCCESS") {
        component.set("v.records", response.getReturnValue());
    }
});
$A.enqueueAction(action);
```

**LWC Wire:**
```javascript
import getRecords from '@salesforce/apex/MyController.getRecords';

@wire(getRecords, { userId: '$userId' })
wiredRecords({ error, data }) {
    if (data) this.records = data;
    else if (error) this.error = error;
}
```

**LWC Imperative:**
```javascript
async loadRecords() {
    try {
        this.records = await getRecords({ userId: this.userId });
    } catch (error) {
        this.handleError(error);
    }
}
```

## Iteration

**Aura:**
```xml
<aura:iteration items="{!v.items}" var="item" indexVar="idx">
    <li data-index="{!idx}">{!item.name}</li>
</aura:iteration>
```

**LWC:**
```html
<template for:each={items} for:item="item" for:index="idx">
    <li key={item.id} data-index={idx}>{item.name}</li>
</template>
```

## Conditional Rendering

**Aura:**
```xml
<aura:if isTrue="{!v.showSection}">
    <div>Visible</div>
    <aura:set attribute="else">
        <div>Hidden</div>
    </aura:set>
</aura:if>
```

**LWC:**
```html
<template lwc:if={showSection}>
    <div>Visible</div>
</template>
<template lwc:else>
    <div>Hidden</div>
</template>
```

## Static Resources (Chart.js)

**Aura:**
```xml
<ltng:require scripts="{!$Resource.ChartJS_1}" afterScriptsLoaded="{!c.afterScriptsLoaded}"/>
```

**LWC:**
```javascript
import { loadScript } from 'lightning/platformResourceLoader';
import ChartJS from '@salesforce/resourceUrl/ChartJS_1';

async renderedCallback() {
    if (this.chartInitialized) return;
    await loadScript(this, ChartJS);
    this.chartInitialized = true;
    this.initChart();
}
```

## Application Events → Lightning Message Service

**Aura:**
```javascript
var appEvent = $A.get("e.c:MyAppEvent");
appEvent.setParams({ "recordId": recordId });
appEvent.fire();
```

**LWC:**
```javascript
import { publish, MessageContext } from 'lightning/messageService';
import RECORD_SELECTED from '@salesforce/messageChannel/RecordSelected__c';

@wire(MessageContext) messageContext;

handleClick() {
    publish(this.messageContext, RECORD_SELECTED, { recordId: this.recordId });
}
```

## Common Pitfalls

| Issue | Aura | LWC |
|-------|------|-----|
| Case | `<c:MyComponent>` | `<c-my-component>` (kebab-case) |
| Binding | `{!v.property}` | `{property}` |
| Event names | `onSelect` | `onselect` (lowercase) |
| DOM access | `component.find("id")` | `this.template.querySelector()` |

## Migration Checklist

- [ ] Convert `aura:attribute` to class properties
- [ ] Replace `aura:handler name="init"` with `connectedCallback()`
- [ ] Convert `ltng:require` to `loadScript/loadStyle`
- [ ] Replace `$A.get("c.method")` with wire or imperative Apex
- [ ] Convert `aura:iteration` to `for:each`
- [ ] Convert `aura:if` to `lwc:if`
- [ ] Replace component events with CustomEvent
- [ ] Replace application events with LMS
- [ ] Update CSS selectors (scoped styling)
- [ ] Add accessibility attributes (ARIA)
- [ ] Write Jest tests
