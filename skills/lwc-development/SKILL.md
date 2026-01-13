---
name: lwc-development
description: Expert Lightning Web Component guidance. Use when building LWC components, implementing wire services, handling events, integrating Chart.js, or writing Jest tests. Provides patterns for container/presentational components, SLDS2 styling, and accessibility.
---

# Lightning Web Component Development

## Project Context

- **API Version**: 65.0
- **Chart Library**: Chart.js (static resource `ChartJS_1`)
- **Target**: Migration from Aura `DB_InternalWholeSaler_Cmp_2025`

## Component Structure

```
myComponent/
├── myComponent.html          # Template
├── myComponent.js            # Controller
├── myComponent.css           # Styles (optional)
├── myComponent.js-meta.xml   # Metadata
└── __tests__/
    └── myComponent.test.js   # Jest tests
```

## Container Component Pattern

```javascript
import { LightningElement, wire, track } from 'lwc';
import { refreshApex } from '@salesforce/apex';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import getDashboardData from '@salesforce/apex/InternalWholesalerController.getDashboardData';

export default class InternalWholesalerDashboard extends LightningElement {
    @api defaultYear;

    selectedMonth = new Date().getMonth() + 1;
    selectedYear = new Date().getFullYear();
    @track selectedUserIds = [];

    wiredDashboardResult;

    get isLoading() {
        return !this.dashboardData && !this.error;
    }

    @wire(getDashboardData, {
        month: '$selectedMonth',
        year: '$selectedYear',
        userIds: '$selectedUserIds'
    })
    wiredDashboard(result) {
        this.wiredDashboardResult = result;
        const { data, error } = result;

        if (data) {
            this.dashboardData = data;
            this.error = undefined;
        } else if (error) {
            this.handleError(error);
        }
    }

    async handleRefresh() {
        await refreshApex(this.wiredDashboardResult);
    }

    handleError(error) {
        const message = error?.body?.message || 'An unexpected error occurred';
        this.dispatchEvent(new ShowToastEvent({
            title: 'Error',
            message,
            variant: 'error'
        }));
    }
}
```

## Chart.js Integration

```javascript
import { LightningElement, api } from 'lwc';
import { loadScript } from 'lightning/platformResourceLoader';
import ChartJS from '@salesforce/resourceUrl/ChartJS_1';

export default class ChartComponent extends LightningElement {
    @api chartType = 'bar';
    _chartData;
    chart;
    chartInitialized = false;

    @api
    get chartData() {
        return this._chartData;
    }
    set chartData(value) {
        this._chartData = value;
        if (this.chartInitialized && this.chart) {
            this.updateChart();
        }
    }

    async renderedCallback() {
        if (this.chartInitialized) return;

        try {
            await loadScript(this, ChartJS);
            this.chartInitialized = true;
            this.initChart();
        } catch (error) {
            console.error('Chart load error', error);
        }
    }

    initChart() {
        const ctx = this.template.querySelector('canvas').getContext('2d');
        this.chart = new Chart(ctx, {
            type: this.chartType,
            data: this._chartData || { labels: [], datasets: [] },
            options: this.getOptions()
        });
    }

    updateChart() {
        if (this.chart && this._chartData) {
            this.chart.data = this._chartData;
            this.chart.update('active');
        }
    }

    disconnectedCallback() {
        if (this.chart) {
            this.chart.destroy();
        }
    }
}
```

## Custom Events

```javascript
// Child dispatches
this.dispatchEvent(new CustomEvent('select', {
    detail: { selectedId: id },
    bubbles: true,
    composed: true
}));

// Parent handles (HTML)
<c-child-component onselect={handleSelect}></c-child-component>

// Parent handles (JS)
handleSelect(event) {
    const selectedId = event.detail.selectedId;
}
```

## Jest Testing

```javascript
import { createElement } from 'lwc';
import MyComponent from 'c/myComponent';
import getDashboardData from '@salesforce/apex/InternalWholesalerController.getDashboardData';

jest.mock('@salesforce/apex/InternalWholesalerController.getDashboardData',
    () => ({ default: jest.fn() }),
    { virtual: true }
);

describe('c-my-component', () => {
    afterEach(() => {
        while (document.body.firstChild) {
            document.body.removeChild(document.body.firstChild);
        }
        jest.clearAllMocks();
    });

    it('displays loading spinner initially', () => {
        const element = createElement('c-my-component', { is: MyComponent });
        document.body.appendChild(element);

        const spinner = element.shadowRoot.querySelector('lightning-spinner');
        expect(spinner).not.toBeNull();
    });
});
```

## Accessibility Requirements

```html
<lightning-button
    label="Refresh"
    icon-name="utility:refresh"
    aria-label="Refresh dashboard data"
    onclick={handleRefresh}>
</lightning-button>

<div role="region" aria-label="Activity Chart">
    <canvas></canvas>
</div>

<template lwc:if={isLoading}>
    <div aria-live="polite" aria-busy="true">
        <lightning-spinner alternative-text="Loading"></lightning-spinner>
    </div>
</template>
```

## SLDS Styling

```css
:host {
    --card-padding: var(--lwc-spacingMedium, 1rem);
}

.chart-container {
    position: relative;
    height: 300px;
    width: 100%;
}
```
