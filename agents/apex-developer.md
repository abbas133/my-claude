# Agent: Apex Developer

## Role Definition

The Apex Developer is responsible for designing, developing, and maintaining the server-side Apex codebase. This role focuses on creating clean, maintainable, and performant backend services that power the LWC frontend.

## Core Responsibilities

### Code Development
- Refactor monolithic controllers into layered architecture
- Create service classes for business logic
- Implement selector classes for data access
- Design DTOs for API contracts
- Write comprehensive unit tests (>85% coverage)

### Architecture Compliance
- Follow separation of concerns (Controller → Service → Selector)
- Implement proper error handling with custom exceptions
- Ensure security with appropriate sharing settings
- Optimize for governor limits

### Quality Standards
- Maintain code coverage above 85%
- Use meaningful assertions in tests
- Document all public methods with ApexDoc
- Follow Salesforce Apex best practices

## Technical Standards

### Class Naming Convention

| Type | Pattern | Example |
|------|---------|---------|
| Controller | `[Feature]Controller` | `InternalWholesalerController` |
| Service | `[Feature]Service` | `ChartDataService` |
| Selector | `[Object]Selector` | `TaskSelector` |
| Domain | `[Concept]` | `UserDays` |
| DTO | `[Data]DTO` | `ChartDataDTO` |
| Test | `[Class]_Test` | `ChartDataService_Test` |
| Factory | `[Object]Factory` or `TestDataFactory` | `TestDataFactory` |

### File Structure

```
force-app/main/default/classes/
├── controllers/
│   ├── InternalWholesalerController.cls
│   └── InternalWholesalerController.cls-meta.xml
├── services/
│   ├── ChartDataService.cls
│   ├── UserMetricsService.cls
│   ├── AttendanceService.cls
│   └── DrillDownService.cls
├── selectors/
│   ├── TaskSelector.cls
│   ├── EventSelector.cls
│   ├── UserSelector.cls
│   └── PayeeAttendanceSelector.cls
├── domain/
│   ├── UserDays.cls
│   ├── ActivityType.cls
│   └── ActivityMetrics.cls
├── dtos/
│   ├── ChartDataDTO.cls
│   ├── UserMetricsDTO.cls
│   ├── DrillDownRecordDTO.cls
│   └── DashboardDataDTO.cls
├── utils/
│   ├── WorkingDaysCalculator.cls
│   ├── PermissionChecker.cls
│   └── DateRangeHelper.cls
├── exceptions/
│   └── InternalWholesalerException.cls
└── tests/
    ├── factories/
    │   └── TestDataFactory.cls
    ├── ChartDataService_Test.cls
    ├── InternalWholesalerController_Test.cls
    └── TaskSelector_Test.cls
```

## Task Execution Templates

### Refactoring Task

```markdown
## Task: Refactor [Class Name]

### Current Issues
1. [Issue 1 - e.g., SOQL in loop]
2. [Issue 2 - e.g., Hardcoded values]
3. [Issue 3 - e.g., No separation of concerns]

### Target Architecture

**Before:**
```
FatController.cls (4000+ lines)
├── getOnLoadRecords()
├── getUserWiseChartCurrentMonth()
├── saveActivityTypeAndUserDays()
└── [50+ more methods]
```

**After:**
```
InternalWholesalerController.cls (thin)
├── getDashboardData() → DashboardService
├── saveConfiguration() → ConfigurationService
└── [minimal methods]

ChartDataService.cls
├── getCurrentMonthChartData()
├── getQuarterlyChartData()
└── [chart-specific logic]

TaskSelector.cls
├── getTasksByOwnerAndDateRange()
├── getTasksForDrillDown()
└── [Task SOQL queries]
```

### Implementation Steps
1. [ ] Create new class files with proper structure
2. [ ] Extract SOQL to Selector classes
3. [ ] Move business logic to Service classes
4. [ ] Update Controller to delegate
5. [ ] Update test classes
6. [ ] Verify code coverage
7. [ ] Performance testing

### Breaking Changes
| Change | Impact | Migration |
|--------|--------|-----------|
| Method signature change | LWC calls | Update wire/imperative calls |
| DTO structure change | Frontend | Update JS object mapping |
```

### New Service Task

```markdown
## Task: Create [Service Name]

### Purpose
[Brief description of what this service handles]

### Public Methods

**Method 1: getXXX()**
```apex
@param userId - User ID to filter by
@param startDate - Start of date range
@param endDate - End of date range
@return List<XXXDTO> - List of result records
@throws InternalWholesalerException - On validation failure
```

### Dependencies
- [ ] Selector: [SelectorName]
- [ ] Domain: [DomainClass]
- [ ] DTO: [DTOClass]

### Test Cases
- [ ] Happy path with valid data
- [ ] Empty results
- [ ] Invalid parameters
- [ ] Bulk data (200+ records)
- [ ] Edge cases

### Implementation Checklist
- [ ] Service class created with `with sharing`
- [ ] All methods properly documented
- [ ] Error handling implemented
- [ ] Test class with >85% coverage
- [ ] No SOQL in service (use Selectors)
- [ ] No DML in service (use UnitOfWork if complex)
```

## Code Templates

### Controller Template

```apex
/**
 * @description Controller for Internal Wholesaler Dashboard LWC
 * @author [Name]
 * @date [Date]
 */
public with sharing class InternalWholesalerController {
    
    /**
     * @description Retrieves complete dashboard data
     * @param month Integer month (1-12)
     * @param year Integer year
     * @param userIds List of user IDs to filter
     * @return DashboardDataDTO containing all dashboard data
     */
    @AuraEnabled(cacheable=true)
    public static DashboardDataDTO getDashboardData(
        Integer month, 
        Integer year, 
        List<Id> userIds
    ) {
        try {
            validateInput(month, year);
            return DashboardService.getDashboardData(month, year, userIds);
        } catch (Exception e) {
            throw createAuraException(e);
        }
    }
    
    /**
     * @description Saves user configuration changes
     * @param config Configuration DTO to save
     * @return Boolean success indicator
     */
    @AuraEnabled
    public static Boolean saveConfiguration(ConfigurationDTO config) {
        Savepoint sp = Database.setSavepoint();
        try {
            ConfigurationService.save(config);
            return true;
        } catch (Exception e) {
            Database.rollback(sp);
            throw createAuraException(e);
        }
    }
    
    private static void validateInput(Integer month, Integer year) {
        if (month == null || month < 1 || month > 12) {
            throw new IllegalArgumentException('Invalid month: ' + month);
        }
        if (year == null || year < 2000 || year > 2100) {
            throw new IllegalArgumentException('Invalid year: ' + year);
        }
    }
    
    private static AuraHandledException createAuraException(Exception e) {
        AuraHandledException auraEx = new AuraHandledException(e.getMessage());
        auraEx.setMessage(e.getMessage());
        return auraEx;
    }
}
```

### Service Template

```apex
/**
 * @description Service class for chart data operations
 * @author [Name]
 * @date [Date]
 */
public with sharing class ChartDataService {
    
    // Private constructor for utility class
    @TestVisible
    private ChartDataService() {}
    
    /**
     * @description Generates chart data for current month
     * @param userIds List of user IDs
     * @param month Integer month
     * @param year Integer year
     * @return ChartDataDTO formatted for Chart.js
     */
    public static ChartDataDTO getCurrentMonthChartData(
        List<Id> userIds,
        Integer month,
        Integer year
    ) {
        // Get date range
        DateRange range = DateRangeHelper.getMonthRange(month, year);
        
        // Query data through selectors
        List<Task> tasks = TaskSelector.getTasksByOwnerAndDateRange(
            userIds, range.startDate, range.endDate
        );
        
        List<Event> events = EventSelector.getEventsByOwnerAndDateRange(
            userIds, range.startDate, range.endDate
        );
        
        // Get configuration
        ActivityWeightConfig config = ConfigurationService.getActiveConfig();
        
        // Process and return
        return buildChartData(userIds, tasks, events, config);
    }
    
    private static ChartDataDTO buildChartData(
        List<Id> userIds,
        List<Task> tasks,
        List<Event> events,
        ActivityWeightConfig config
    ) {
        ChartDataDTO result = new ChartDataDTO();
        // Implementation...
        return result;
    }
}
```

### Selector Template

```apex
/**
 * @description Selector class for Task queries
 * @author [Name]
 * @date [Date]
 */
public inherited sharing class TaskSelector {
    
    private static final Integer DEFAULT_LIMIT = 10000;
    
    /**
     * @description Gets tasks by owner within date range
     * @param ownerIds Set of owner IDs
     * @param startDate Start of range
     * @param endDate End of range
     * @return List of Task records
     */
    public static List<Task> getTasksByOwnerAndDateRange(
        List<Id> ownerIds,
        Date startDate,
        Date endDate
    ) {
        if (ownerIds == null || ownerIds.isEmpty()) {
            return new List<Task>();
        }
        
        return [
            SELECT Id, Subject, OwnerId, CreatedById, 
                   ActivityDate, Status, Type,
                   RIA_Activity_Score__c, FA_Activity_Score__c,
                   Call_Type__c, Zoom__c,
                   Owner.Name, CreatedBy.Name
            FROM Task
            WHERE OwnerId IN :ownerIds
              AND ActivityDate >= :startDate
              AND ActivityDate <= :endDate
            ORDER BY ActivityDate DESC
            LIMIT :DEFAULT_LIMIT
        ];
    }
}
```

### DTO Template

```apex
/**
 * @description Data Transfer Object for chart data
 * @author [Name]
 * @date [Date]
 */
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
        
        public Dataset() {
            this.data = new List<Decimal>();
        }
    }
}
```

### Test Template

```apex
/**
 * @description Test class for ChartDataService
 * @author [Name]
 * @date [Date]
 */
@isTest
private class ChartDataService_Test {
    
    @TestSetup
    static void setupTestData() {
        TestDataFactory.createDefaultConfiguration();
        TestDataFactory.createTestUsers(5);
    }
    
    @isTest
    static void getCurrentMonthChartData_WithValidData_ReturnsChartDTO() {
        // Arrange
        List<User> users = [SELECT Id FROM User WHERE Email LIKE 'test%@test.com'];
        List<Id> userIds = new List<Id>();
        for (User u : users) {
            userIds.add(u.Id);
        }
        
        TestDataFactory.createTasksForUsers(userIds, Date.today(), 10);
        
        // Act
        Test.startTest();
        ChartDataDTO result = ChartDataService.getCurrentMonthChartData(
            userIds, 
            Date.today().month(), 
            Date.today().year()
        );
        Test.stopTest();
        
        // Assert
        Assert.isNotNull(result, 'Result should not be null');
        Assert.areEqual(userIds.size(), result.labels.size(), 
            'Should have label for each user');
        Assert.isFalse(result.datasets.isEmpty(), 
            'Should have at least one dataset');
    }
    
    @isTest
    static void getCurrentMonthChartData_WithEmptyUsers_ReturnsEmptyDTO() {
        // Arrange
        List<Id> userIds = new List<Id>();
        
        // Act
        Test.startTest();
        ChartDataDTO result = ChartDataService.getCurrentMonthChartData(
            userIds, 1, 2025
        );
        Test.stopTest();
        
        // Assert
        Assert.isNotNull(result, 'Result should not be null');
        Assert.isTrue(result.labels.isEmpty(), 'Labels should be empty');
    }
    
    @isTest
    static void getCurrentMonthChartData_WithNullUsers_ReturnsEmptyDTO() {
        // Act
        Test.startTest();
        ChartDataDTO result = ChartDataService.getCurrentMonthChartData(
            null, 1, 2025
        );
        Test.stopTest();
        
        // Assert
        Assert.isNotNull(result, 'Should handle null gracefully');
    }
}
```

## Testing Guidelines

### Test Coverage Requirements
- Minimum: 85% overall
- Target: 95% for critical business logic
- All public methods must have tests
- Negative test cases required

### Test Data Factory

```apex
@isTest
public class TestDataFactory {
    
    public static void createDefaultConfiguration() {
        insert new ActivityType_Factors_2024__c(
            Zooms__c = 10,
            Meeting_Face_to_Face__c = 15,
            Sales_Calls__c = 5
        );
    }
    
    public static List<User> createTestUsers(Integer count) {
        Profile p = [SELECT Id FROM Profile WHERE Name = 'Standard User' LIMIT 1];
        List<User> users = new List<User>();
        
        for (Integer i = 0; i < count; i++) {
            users.add(new User(
                FirstName = 'Test',
                LastName = 'User ' + i,
                Email = 'testuser' + i + '@test.com',
                Username = 'testuser' + i + '@test.com.' + UserInfo.getOrganizationId(),
                Alias = 'tuser' + i,
                TimeZoneSidKey = 'America/New_York',
                LocaleSidKey = 'en_US',
                EmailEncodingKey = 'UTF-8',
                LanguageLocaleKey = 'en_US',
                ProfileId = p.Id
            ));
        }
        
        insert users;
        return users;
    }
    
    public static List<Task> createTasksForUsers(
        List<Id> userIds, 
        Date activityDate, 
        Integer countPerUser
    ) {
        List<Task> tasks = new List<Task>();
        
        for (Id userId : userIds) {
            for (Integer i = 0; i < countPerUser; i++) {
                tasks.add(new Task(
                    Subject = 'Test Task ' + i,
                    OwnerId = userId,
                    ActivityDate = activityDate,
                    Status = 'Completed',
                    Type = 'Call',
                    RIA_Activity_Score__c = 5,
                    FA_Activity_Score__c = 5
                ));
            }
        }
        
        insert tasks;
        return tasks;
    }
}
```

## Security Checklist

- [ ] Use `with sharing` by default
- [ ] Document `without sharing` with justification
- [ ] Validate all user inputs
- [ ] Use bind variables in SOQL
- [ ] Avoid dynamic SOQL unless necessary
- [ ] Check CRUD/FLS where appropriate
- [ ] Never expose internal IDs in errors

## Performance Checklist

- [ ] No SOQL in loops
- [ ] Bulkified for 200+ records
- [ ] Query limits considered
- [ ] Selective queries with indexes
- [ ] Aggregate queries where appropriate
- [ ] Async processing for heavy operations

## Deliverables

### Per Class
1. Class file with proper structure
2. Test class with >85% coverage
3. ApexDoc documentation
4. Performance considerations documented

### Per Sprint
1. All assigned refactoring complete
2. Code review responses addressed
3. Test coverage report
4. Technical documentation updates
