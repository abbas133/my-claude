---
name: salesforce-apex-expert
description: Expert Salesforce Apex guidance. Use when writing Apex classes, controllers, triggers, batch jobs, or scheduled classes. Provides patterns for SOQL optimization, bulkification, governor limits, separation of concerns (Controller/Service/Selector/Domain), and security best practices.
---

# Salesforce Apex Expert

## Project Context

- **API Version**: 65.0
- **Main Controller**: `DB_InternalWholeSaler_Controller_2024` (4,185 lines, `without sharing`)
- **Utility Class**: `DB_InternalWholeSaler_Utill` (working day calculations)

## Architecture Patterns

### Separation of Concerns (SOC)

```
Controller Layer  → Input validation, response formatting, thin methods
       ↓
Service Layer     → Business logic orchestration, transaction management
       ↓
Selector Layer    → Centralized SOQL queries, query optimization
       ↓
Domain Layer      → Object-specific business logic, validation rules
```

## Controller Pattern

```apex
public with sharing class InternalWholesalerController {

    @AuraEnabled(cacheable=true)
    public static DashboardDataDTO getDashboardData(Integer month, Integer year, List<Id> userIds) {
        try {
            validateInput(month, year, userIds);
            return DashboardService.getDashboardData(month, year, userIds);
        } catch (Exception e) {
            throw createAuraException(e);
        }
    }

    @AuraEnabled
    public static Boolean saveConfiguration(ActivityConfigDTO config) {
        Savepoint sp = Database.setSavepoint();
        try {
            ConfigurationService.saveConfiguration(config);
            return true;
        } catch (Exception e) {
            Database.rollback(sp);
            throw createAuraException(e);
        }
    }

    private static void validateInput(Integer month, Integer year, List<Id> userIds) {
        if (month == null || month < 1 || month > 12) {
            throw new IllegalArgumentException('Month must be between 1 and 12');
        }
    }

    private static AuraHandledException createAuraException(Exception e) {
        AuraHandledException auraEx = new AuraHandledException(e.getMessage());
        auraEx.setMessage(e.getMessage());
        return auraEx;
    }
}
```

## Service Pattern

```apex
public with sharing class ChartDataService {

    @TestVisible
    private ChartDataService() {}

    public static ChartDataDTO getCurrentMonthChartData(List<Id> userIds, Integer month, Integer year) {
        DateRange dateRange = DateRangeHelper.getMonthRange(month, year);

        List<Task> tasks = TaskSelector.getTasksByOwnerAndDateRange(userIds, dateRange.startDate, dateRange.endDate);
        List<Event> events = EventSelector.getEventsByOwnerAndDateRange(userIds, dateRange.startDate, dateRange.endDate);

        ActivityWeightConfig config = ConfigurationService.getActiveWeightConfig();
        return aggregateChartData(userIds, tasks, events, config);
    }
}
```

## Selector Pattern

```apex
public inherited sharing class TaskSelector {

    public static List<Task> getTasksByOwnerAndDateRange(List<Id> ownerIds, Date startDate, Date endDate) {
        if (ownerIds == null || ownerIds.isEmpty()) {
            return new List<Task>();
        }

        return [
            SELECT Id, Subject, OwnerId, CreatedById, ActivityDate, Status, Type,
                   RIA_Activity_Score__c, FA_Activity_Score__c, Call_Type__c, Zoom__c,
                   Owner.Name, CreatedBy.Name
            FROM Task
            WHERE OwnerId IN :ownerIds
              AND ActivityDate >= :startDate
              AND ActivityDate <= :endDate
            ORDER BY ActivityDate DESC
            LIMIT 10000
        ];
    }
}
```

## DTO Pattern

```apex
public class ChartDataDTO {
    @AuraEnabled public List<String> labels { get; set; }
    @AuraEnabled public List<Dataset> datasets { get; set; }

    public ChartDataDTO() {
        this.labels = new List<String>();
        this.datasets = new List<Dataset>();
    }

    public class Dataset {
        @AuraEnabled public String label { get; set; }
        @AuraEnabled public String backgroundColor { get; set; }
        @AuraEnabled public List<Decimal> data { get; set; }
    }
}
```

## Working Days Calculation (Project-Specific)

```apex
// Existing utility - modes: 'Remain', 'Fullmonth', 'Elapsed'
DB_InternalWholeSaler_Utill.calculateWorkingDays(Date dGivenDate, String mode)
```

## Governor Limits Best Practices

```apex
// Check limits before operations
if (Limits.getQueries() < Limits.getLimitQueries() - 10) {
    // Safe to query
}

// Use maps for efficient lookups (avoid queries in loops)
Map<Id, Account> accountMap = new Map<Id, Account>(
    [SELECT Id, Name FROM Account WHERE Id IN :accountIds]
);

// Batch for large data volumes
Database.executeBatch(new MyBatchClass(), 200);
```

## Security Checklist

- Use `with sharing` by default
- Document all `without sharing` usage with justification
- Use bind variables in SOQL (never string concatenation)
- Validate all user inputs
- Use `@AuraEnabled(cacheable=true)` only for read-only operations
