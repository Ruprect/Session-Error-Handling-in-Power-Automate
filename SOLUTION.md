# Error Handling in Power Automate - Solution Documentation

**Version:** 1.0.0.4
**Solution Name:** ErrorHandlinginPowerAutomate
**Publisher:** Michael Nielsen (mni)

## Overview

This Power Platform solution demonstrates best practices for implementing error handling in Power Automate cloud flows. It contains 10 demonstration flows that progressively showcase different error handling patterns, from basic implementations to advanced scenarios with child flows and complex logic.

## Solution Components

### Dataverse Table

**Table Name:** `mni_session_errorhandling_cars`

This custom table stores car information for demonstration purposes.

**Columns:**
- `mni_name` - Name (primary field)
- `mni_session_errorhandling_color` - Text field for car color
- `mni_session_errorhandling_licenseplate` - Text field for license plate
- `mni_session_errorhandling_type` - Text field for car type
- `createdon` - DateTime (system field)

### Environment Variables

Configure these environment variables after importing the solution:

| Variable | Schema Name | Purpose |
|----------|-------------|---------|
| Error Handler Email | `mni_ErrorHandlerEmail` | Email address to notify when the error handler itself fails (fallback notification) |
| Testing API Endpoint | `mni_TestingAPIEndpoint` | Valid API endpoint URL for successful API calls |
| Testing API Endpoint Invalid | `mni_TestingAPIEndpointInvalid` | Invalid API endpoint URL to intentionally trigger errors for demonstration |

**Default Values:**
- `mni_ErrorHandlerEmail`: mni@abakion.com (update to your email)

### Connection References

The solution uses the following connectors:

1. **Dataverse (Common Data Service for Apps)**
   - Used for: CRUD operations on the cars table
   - Connection Reference: `mni_sharedcommondataserviceforapps_ad497`

2. **Microsoft Teams**
   - Used for: Sending notifications and error messages
   - Connection Reference: `mni_sharedteams_752c3`

## Demonstration Flows

### 001 - No Error Handling

**Flow Name:** `001 No error handling`
**Purpose:** Demonstrates what happens when no error handling is implemented

**Flow Logic:**
1. Manual trigger (button)
2. Run child flow: Get car details
3. Add car to Dataverse table
4. List all cars from Dataverse
5. Select data for table
6. Create HTML table

**Demonstration Point:** Shows how the entire flow fails when any single action fails, with no graceful degradation or error notification.

---

### 002 - Basic Error Handling

**Flow Name:** `002 Basic error handling`
**Purpose:** Introduces basic error handling using "Configure run after" settings

**Flow Logic:**
1. Manual trigger
2. Try to perform operations
3. Error handling actions configured to run after previous step fails

**Demonstration Point:** Shows the simplest form of error handling using Power Automate's built-in "Configure run after" feature on individual actions.

---

### 003 - Advanced Error Handling

**Flow Name:** `003 Advanced error handling`
**Purpose:** Demonstrates the complete Try-Catch-Finally pattern using scopes

**Flow Logic:**

**Try Scope:**
- Run child flow: Get car details
- Add car to Dataverse
- List all cars
- Select and format data
- Create HTML table

**Catch Scope (runs after: Try has failed):**
- Compose: Capture error details from Try scope
- Post message in Teams channel with error information
- Note: Could call error handler child flow here

**Finally Scope (runs after: Try succeeded or failed):**
- Compose: Log completion status
- Perform cleanup operations
- Actions that should always run regardless of success/failure

**Demonstration Point:** This is the recommended pattern for production flows. It provides structured error handling, maintains error context, and ensures cleanup operations always execute.

---

### 004 - Advanced Error Handling with Child Flows

**Flow Name:** `004 Advanced error handling - With child flows`
**Purpose:** Shows how to implement Try-Catch-Finally with centralized error handling

**Flow Logic:**

**Try Scope:**
- Run child flow: Get car details
- Add car to Dataverse
- Additional business logic

**Catch Scope (runs after: Try failed):**
- Compose: Extract error details
- Run child flow: Error Handler (030)
  - Passes error details to centralized handler
  - Centralized logging and notification

**Finally Scope:**
- Run child flow: Send notification (040)
- Cleanup operations

**Demonstration Point:** Shows how to create reusable error handling components that can be called from multiple flows, promoting consistency and maintainability.

---

### 005 - Flow with ForEach and Switch (Without Additional Error)

**Flow Name:** `005 Flow with ForEach and Switch - Without additional error`
**Purpose:** Demonstrates error handling in complex flow patterns

**Flow Logic:**

**Try Scope:**
- Get list of items
- **ForEach loop:**
  - **Switch control:**
    - Case 1: Handle specific scenario
    - Case 2: Handle different scenario
    - Default: Handle all other cases
- Continue with business logic

**Catch Scope:**
- Handle errors from loop and switch operations

**Finally Scope:**
- Cleanup and logging

**Demonstration Point:** Shows how error handling works with loops and conditional logic. Errors within a ForEach can be handled at the item level or at the scope level.

---

### 006 - Flow with ForEach and Switch (With Additional Error Handling)

**Flow Name:** `006 Flow with ForEach and Switch and additional error extra`
**Purpose:** Advanced error handling within loops and nested scopes

**Flow Logic:**

**Try Scope:**
- **ForEach loop:**
  - **Try Scope (nested):**
    - Process individual item
    - **Switch control:**
      - Multiple cases with error-prone operations
  - **Catch Scope (nested):**
    - Handle item-level errors
    - Continue processing other items
- Additional business logic

**Catch Scope (main):**
- Handle errors from overall flow
- Call error handler if needed

**Finally Scope:**
- Overall cleanup

**Demonstration Point:** Shows nested error handling - individual items in a loop can fail without stopping the entire loop, while still maintaining overall flow error handling.

---

### 010 - Get Car Details

**Flow Name:** `010 Get car details`
**Purpose:** Child flow that retrieves car information from an external API

**Type:** Child flow (can be called from other flows)

**Flow Logic:**
1. Receive call from parent flow
2. Call external API using environment variable `mni_TestingAPIEndpoint`
3. Parse API response
4. Return structured car data:
   - Color
   - License Plate
   - Type

**Demonstration Point:** This child flow intentionally uses an environment variable for the API endpoint, allowing you to switch between valid and invalid endpoints to trigger different error scenarios.

**Usage:** Called by flows 001, 003, 004 to simulate retrieving data from external systems.

---

### 030 - Error Handler

**Flow Name:** `030 Error Handler`
**Purpose:** Centralized error handling child flow for consistent error management

**Type:** Child flow

**Flow Logic:**

**Try Scope:**
1. Receive error details from parent flow (parameters)
2. Compose comprehensive error message including:
   - Error timestamp
   - Parent flow name
   - Error message
   - Error code
   - Action that failed
   - Stack trace
3. Log error to Dataverse (optional - could add table)
4. Post detailed error message in Teams channel
5. Return success status to parent

**Catch Scope:**
- If error handler itself fails:
  - Compose fallback error message
  - Send email to `mni_ErrorHandlerEmail` environment variable
  - This ensures errors are never silently lost

**Finally Scope:**
- Update error handling metrics (optional)
- Return to parent flow

**Demonstration Point:** Having a centralized error handler ensures:
- Consistent error formatting
- Centralized error logging
- Fallback notification if primary error handling fails
- Reusability across multiple flows

**Best Practice:** Every production solution should have a similar error handler flow that can be called by any flow in the solution.

---

### 040 - Send Notification

**Flow Name:** `040 Send notification`
**Purpose:** Child flow for sending notifications via Teams

**Type:** Child flow

**Flow Logic:**
1. Receive notification parameters from parent:
   - Message text
   - Channel ID
   - Notification type (Success/Warning/Error)
2. Format message with appropriate styling
3. Post adaptive card or message in Teams
4. Return success/failure status

**Demonstration Point:** Separating notification logic into a child flow allows:
- Consistent notification formatting
- Easy switching between notification channels (Teams, Email, etc.)
- Reusability
- Testing notifications independently

---

### 999 - Template Try Catch Finally Scopes

**Flow Name:** `999 Template Try Catch Finally scopes`
**Purpose:** Ready-to-use template for implementing error handling in your own flows

**Type:** Template flow

**Flow Structure:**

```
Try Scope
├─ [Your business logic actions go here]
└─ [Add all actions that should be error-handled]

Catch Scope (Configure run after: Try has failed)
├─ Compose: Error Details
│   └─ result('Try')
├─ [Optional: Transform error data]
└─ Run child flow: Error Handler (030)
    ├─ Pass error details
    └─ Flow name
    └─ Additional context

Finally Scope (Configure run after: Try succeeded, failed, skipped, timed out)
├─ [Cleanup operations]
├─ [Logging operations]
└─ [Actions that should always run]
```

**How to Use:**
1. Copy this template flow
2. Rename it for your use case
3. Add your business logic actions inside the Try scope
4. Customize the error details captured in the Catch scope
5. Add any cleanup operations in the Finally scope
6. The error handler call is pre-configured - just update the parameters

**Demonstration Point:** This template provides the complete skeleton for implementing proper error handling. Use this as your starting point for all new flows.

---

## Error Handling Pattern: Try-Catch-Finally

### Overview

The Try-Catch-Finally pattern is implemented in Power Automate using **Scopes** with **Configure run after** settings.

### Implementation Steps

#### 1. Create Try Scope

```
Scope: Try
└─ Add all your main business logic actions here
```

#### 2. Create Catch Scope

```
Scope: Catch
└─ Configure run after: Try → has failed (check only "has failed")
└─ Add error handling actions
```

**Common Catch Actions:**
- Compose action to capture error details: `result('Try')`
- Parse error information
- Call error handler child flow
- Send notifications
- Log to database

#### 3. Create Finally Scope

```
Scope: Finally
└─ Configure run after: Try → all states checked (succeeded, failed, skipped, timed out)
└─ Add cleanup and logging actions
```

**Common Finally Actions:**
- Close connections
- Update status fields
- Log completion metrics
- Send completion notifications
- Clear temporary variables

### Error Details Expressions

**Get all results from Try scope:**
```javascript
result('Try')
```

**Get specific error information:**
```javascript
// Error message
result('Try')[0]['outputs']['body']['error']['message']

// Error code
result('Try')[0]['outputs']['statusCode']

// Failed action name
result('Try')[0]['name']

// Full error object
result('Try')[0]['error']
```

### Best Practices

1. **Always use scopes for error handling** - Don't rely on action-level "Configure run after" for complex flows

2. **Keep Try scope focused** - Only include actions that should be error-handled together

3. **Capture comprehensive error context** - Include flow name, timestamp, user info, and full error details

4. **Use centralized error handler** - Create reusable child flow for consistent error management

5. **Implement Finally scope** - Ensure cleanup operations always run

6. **Test error scenarios** - Use invalid API endpoints and bad data to verify error handling works

7. **Don't hide errors** - Always log or notify, even if you handle the error gracefully

8. **Consider error severity** - Some errors should stop the flow, others should be logged and continued

9. **Nested error handling** - Use nested Try-Catch blocks inside loops to handle item-level vs flow-level errors

10. **Document error handling** - Add comments explaining what errors are expected and how they're handled

## Testing the Solution

### Test Scenarios

1. **No Error Path**
   - Use valid API endpoint in environment variable
   - Run flows - all should succeed
   - Verify data is created in Dataverse

2. **API Error Path**
   - Change `mni_TestingAPIEndpoint` to invalid URL
   - Run flows with error handling
   - Verify error is caught, logged, and notified
   - Verify flow doesn't fail entirely

3. **Dataverse Error Path**
   - Remove required field values
   - Run flows
   - Verify Dataverse errors are caught

4. **Child Flow Error Path**
   - Modify child flow to throw error
   - Verify parent flow handles error gracefully

5. **Error Handler Failure**
   - Temporarily break error handler (030)
   - Verify fallback email notification is sent
   - Verify error isn't silently lost

### Comparison Testing

Run the same test scenario on flows with and without error handling:

1. Run **001 No error handling** with invalid API endpoint
   - Result: Entire flow fails, no notification

2. Run **003 Advanced error handling** with invalid API endpoint
   - Result: Error caught, notification sent, Finally scope executes

This demonstrates the value of proper error handling.

## Migration Guide

### Applying to Your Own Flows

1. **Start with template** - Use flow 999 as your base

2. **Add business logic to Try scope** - Move your existing actions into the Try scope

3. **Configure Catch scope**
   - Set "Configure run after" on Catch to run when Try has failed
   - Add Compose action with `result('Try')` to capture errors
   - Call error handler child flow (030) or implement your own notification

4. **Configure Finally scope**
   - Set "Configure run after" to run for all states
   - Add cleanup operations
   - Add completion logging

5. **Test thoroughly** - Test both success and failure paths

### Example: Converting Existing Flow

**Before (no error handling):**
```
Manual trigger
└─ HTTP action: Call API
└─ Parse JSON: Parse response
└─ Create item: Add to Dataverse
└─ Send email: Success notification
```

**After (with error handling):**
```
Manual trigger
└─ Try Scope
    └─ HTTP action: Call API
    └─ Parse JSON: Parse response
    └─ Create item: Add to Dataverse
    └─ Send email: Success notification
└─ Catch Scope (run after: Try failed)
    └─ Compose: result('Try')
    └─ Run child flow: Error Handler
└─ Finally Scope (run after: Try all states)
    └─ Update status: Log completion
```

## Additional Resources

### Related Documentation

- [Microsoft: Handle errors in flows](https://learn.microsoft.com/en-us/power-automate/error-handling)
- [Scope control in Power Automate](https://learn.microsoft.com/en-us/power-automate/data-operations#use-scopes)
- [Configure run after settings](https://learn.microsoft.com/en-us/power-automate/configure-run-after)

### Common Issues

**Issue:** Catch scope not running when Try fails
**Solution:** Verify "Configure run after" is set to only "has failed" for Catch scope

**Issue:** Finally scope not running
**Solution:** Verify "Configure run after" has all four states checked (succeeded, failed, skipped, timed out)

**Issue:** Error details are empty
**Solution:** Ensure you're using `result('ScopeName')` not `outputs('ScopeName')`

**Issue:** Flow fails despite error handling
**Solution:** Check that all business logic is inside Try scope, not outside it

## Support

For questions about this solution or the error handling patterns demonstrated, please refer to the presentation materials in the `slides/` folder or contact the author through the conference session channels.
