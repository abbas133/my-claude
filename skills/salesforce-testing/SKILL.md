---
name: salesforce-testing
description: Expert Salesforce Apex testing guidance. Use when writing test classes, creating test data factories, mocking callouts, testing async operations, or debugging test failures. Provides patterns for bulk testing, governor limit validation, and achieving code coverage.
---

# Salesforce Testing

## Project Context

- **Test Classes**: `DB_InternalWholeSaler_Ctrlr_2024_Test`, `DB_InternalWholeSaler_Utill_Test`
- **Coverage Target**: >85% for production deployment
- **API Version**: 65.0

## Test Class Structure

```apex
@IsTest
private class MyService_Test {

    @TestSetup
    static void setupTestData() {
        // Create test data once for all test methods
        TestDataFactory.createDefaultConfiguration();
        TestDataFactory.createTestUsers(5);
    }

    @IsTest
    static void methodName_Scenario_ExpectedResult() {
        // Arrange
        List<Id> userIds = TestDataFactory.getTestUserIds();

        // Act
        Test.startTest();
        ChartDataDTO result = ChartDataService.getCurrentMonthChartData(userIds, 1, 2025);
        Test.stopTest();

        // Assert
        Assert.isNotNull(result, 'Result should not be null');
        Assert.areEqual(5, result.labels.size(), 'Should have 5 labels');
    }
}
```

## Test Data Factory

```apex
@IsTest
public class TestDataFactory {

    public static void createDefaultConfiguration() {
        insert new ActivityType_Factors_2024__c(
            Zooms__c = 10,
            Meeting_Face_to_Face__c = 15,
            Sales_Calls__c = 5,
            Service_Calls__c = 3,
            Sales_Email__c = 2,
            ETF_Leads__c = 8
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

    public static List<Task> createTasks(Id ownerId, Date activityDate, Integer count) {
        List<Task> tasks = new List<Task>();

        for (Integer i = 0; i < count; i++) {
            tasks.add(new Task(
                Subject = 'Test Task ' + i,
                OwnerId = ownerId,
                ActivityDate = activityDate,
                Status = 'Completed',
                Type = 'Call',
                RIA_Activity_Score__c = 5,
                FA_Activity_Score__c = 5
            ));
        }

        insert tasks;
        return tasks;
    }

    public static List<Event> createEvents(Id ownerId, Datetime startTime, Integer count) {
        List<Event> events = new List<Event>();

        for (Integer i = 0; i < count; i++) {
            events.add(new Event(
                Subject = 'Test Event ' + i,
                OwnerId = ownerId,
                StartDateTime = startTime.addHours(i),
                EndDateTime = startTime.addHours(i + 1),
                Type = 'Meeting'
            ));
        }

        insert events;
        return events;
    }

    public static PayeeSummary__c createPayeeSummary(Id userId, Integer month, Integer year) {
        PayeeSummary__c summary = new PayeeSummary__c(
            User__c = userId,
            Month__c = month,
            Year__c = year
        );
        insert summary;
        return summary;
    }

    public static PayeeUser_Attendance__c createAttendance(Id userId, Integer month, Integer year) {
        PayeeUser_Attendance__c attendance = new PayeeUser_Attendance__c(
            User__c = userId,
            Month__c = month,
            Year__c = year
        );
        insert attendance;
        return attendance;
    }

    public static NationalHolidays__c createHoliday(Date holidayDate, String name) {
        NationalHolidays__c holiday = new NationalHolidays__c(
            Holiday_Date__c = holidayDate,
            Name = name,
            Year__c = holidayDate.year()
        );
        insert holiday;
        return holiday;
    }

    public static List<Id> getTestUserIds() {
        return new List<Id>(new Map<Id, User>([
            SELECT Id FROM User WHERE Email LIKE 'testuser%@test.com' LIMIT 10
        ]).keySet());
    }
}
```

## Project-Specific Test Data

For this project, tests must create:
- User records
- Task records (with Activity_Type__c, FA_Activity_Score_Final__c, RIA_Activity_Score_Final__c)
- Event records
- PayeeSummary__c records
- PayeeUser_Attendance__c records
- ActivityType_Factors_2024__c (Org Defaults)
- NationalHolidays__c records

## Test Patterns

### Positive Test
```apex
@IsTest
static void getDashboardData_ValidInput_ReturnsData() {
    // Arrange
    List<Id> userIds = TestDataFactory.getTestUserIds();
    TestDataFactory.createTasks(userIds[0], Date.today(), 10);

    // Act
    Test.startTest();
    DashboardDataDTO result = InternalWholesalerController.getDashboardData(1, 2025, userIds);
    Test.stopTest();

    // Assert
    Assert.isNotNull(result, 'Should return dashboard data');
    Assert.isFalse(result.chartData.labels.isEmpty(), 'Should have chart labels');
}
```

### Negative Test
```apex
@IsTest
static void getDashboardData_NullUserIds_ReturnsEmpty() {
    // Act
    Test.startTest();
    DashboardDataDTO result = InternalWholesalerController.getDashboardData(1, 2025, null);
    Test.stopTest();

    // Assert
    Assert.isNotNull(result, 'Should return empty DTO, not null');
    Assert.isTrue(result.chartData.labels.isEmpty(), 'Labels should be empty');
}
```

### Exception Test
```apex
@IsTest
static void getDashboardData_InvalidMonth_ThrowsException() {
    // Arrange
    List<Id> userIds = TestDataFactory.getTestUserIds();

    // Act & Assert
    Test.startTest();
    try {
        InternalWholesalerController.getDashboardData(13, 2025, userIds);
        Assert.fail('Should have thrown exception');
    } catch (AuraHandledException e) {
        Assert.isTrue(e.getMessage().contains('Invalid month'), 'Should contain error message');
    }
    Test.stopTest();
}
```

### Bulk Test
```apex
@IsTest
static void processRecords_BulkData_HandlesCorrectly() {
    // Arrange - Create 200 records
    List<Id> userIds = TestDataFactory.getTestUserIds();
    for (Id userId : userIds) {
        TestDataFactory.createTasks(userId, Date.today(), 200);
    }

    // Act
    Test.startTest();
    List<ChartDataDTO> results = ChartDataService.getBulkChartData(userIds);
    Test.stopTest();

    // Assert
    Assert.areEqual(userIds.size(), results.size(), 'Should process all users');
}
```

## Testing Async Operations

### Future Method
```apex
@IsTest
static void myFutureMethod_ValidInput_ProcessesCorrectly() {
    // Arrange
    Account acc = new Account(Name = 'Test');
    insert acc;

    // Act
    Test.startTest();
    MyClass.myFutureMethod(acc.Id);
    Test.stopTest();  // Forces future to complete

    // Assert
    acc = [SELECT Status__c FROM Account WHERE Id = :acc.Id];
    Assert.areEqual('Processed', acc.Status__c);
}
```

### Batch Job
```apex
@IsTest
static void myBatch_ValidData_ProcessesAllRecords() {
    // Arrange
    TestDataFactory.createTestUsers(200);

    // Act
    Test.startTest();
    Database.executeBatch(new MyBatchClass(), 200);
    Test.stopTest();

    // Assert
    List<User> users = [SELECT Id, Processed__c FROM User WHERE Email LIKE 'testuser%'];
    for (User u : users) {
        Assert.isTrue(u.Processed__c, 'All users should be processed');
    }
}
```

### Queueable
```apex
@IsTest
static void myQueueable_ValidInput_Executes() {
    // Arrange
    String testData = 'test';

    // Act
    Test.startTest();
    System.enqueueJob(new MyQueueable(testData));
    Test.stopTest();

    // Assert - check side effects
}
```

## Mocking HTTP Callouts

```apex
@IsTest
private class MyCallout_Test implements HttpCalloutMock {

    public HTTPResponse respond(HTTPRequest req) {
        HttpResponse res = new HttpResponse();
        res.setHeader('Content-Type', 'application/json');
        res.setBody('{"success": true, "data": []}');
        res.setStatusCode(200);
        return res;
    }

    @IsTest
    static void makeCallout_ValidRequest_ReturnsSuccess() {
        // Arrange
        Test.setMock(HttpCalloutMock.class, new MyCallout_Test());

        // Act
        Test.startTest();
        String result = MyService.makeCallout();
        Test.stopTest();

        // Assert
        Assert.areEqual('success', result);
    }
}
```

## Run As Different User

```apex
@IsTest
static void restrictedMethod_AsAdmin_Succeeds() {
    // Arrange
    User adminUser = [SELECT Id FROM User WHERE Profile.Name = 'System Administrator' AND IsActive = true LIMIT 1];

    System.runAs(adminUser) {
        // Act
        Test.startTest();
        Boolean result = PermissionChecker.isAdminUser();
        Test.stopTest();

        // Assert
        Assert.isTrue(result, 'Admin user should have access');
    }
}
```

## Test Commands

```bash
# Run specific test class
sf apex run test --class-names DB_InternalWholeSaler_Ctrlr_2024_Test --result-format human

# Run all project tests
sf apex run test --class-names DB_InternalWholeSaler_Ctrlr_2024_Test,DB_InternalWholeSaler_Utill_Test --result-format human --code-coverage

# Run single test method
sf apex run test --tests DB_InternalWholeSaler_Ctrlr_2024_Test.testGetChartData_Success

# Run with output directory
sf apex run test --class-names DB_InternalWholeSaler_Ctrlr_2024_Test --output-dir ./test-results --code-coverage
```

## Coverage Requirements

- **Production Deployment**: Minimum 75% overall coverage
- **Best Practice**: Target >85% for critical business logic
- **Each Trigger**: Must have some test coverage
- **Assertions Required**: Tests without assertions are discouraged

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Mixed DML | Setup/non-setup objects in same transaction | Use `System.runAs()` |
| SOQL 101 | Queries in loops | Bulkify with maps |
| Test isolation | Tests depend on org data | Use `@TestSetup` |
| Async not completing | Missing `Test.stopTest()` | Add after async call |
| Coverage gaps | Untested branches | Add negative tests |

## Assertion Best Practices

```apex
// Use Assert class (modern)
Assert.isNotNull(result, 'Result should not be null');
Assert.areEqual(expected, actual, 'Values should match');
Assert.isTrue(condition, 'Condition should be true');
Assert.fail('Should not reach here');

// Avoid System.assert (legacy)
// System.assertEquals(expected, actual);  // Avoid
```
