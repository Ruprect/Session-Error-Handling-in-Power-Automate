# Error Handling in Power Automate - Conference Session

This repository contains presentation materials and a comprehensive Power Platform solution demonstrating best practices for error handling in Power Automate flows.

## Overview

Error handling is critical for building robust and reliable Power Automate flows. This session demonstrates various error handling patterns, from basic to advanced implementations, using real-world examples and the Try-Catch-Finally pattern.

## What's Included

### Presentation Materials

Located in the `slides/` folder:
- **PPCC25 Presentation.pdf** - Latest presentation from Power Platform Community Conference 2025
- **Old presentations/** - Previous versions from EPPC25 (European Power Platform Conference 2025)

### Power Platform Solution

The `solutions/` folder contains the **Error Handling in Power Automate** solution package (version 1.0.0.4) demonstrating:

- Progressive examples from no error handling to advanced patterns
- Try-Catch-Finally scope implementation
- Error handling with child flows
- Complex scenarios with ForEach loops and Switch statements
- Reusable error handler components
- Template flows for implementing error handling

See [SOLUTION.md](SOLUTION.md) for detailed documentation of the solution components and flows.

## Key Concepts Demonstrated

1. **Try-Catch-Finally Pattern** - Industry-standard error handling adapted for Power Automate
2. **Scope-Based Error Handling** - Using scopes to isolate and manage errors
3. **Centralized Error Handling** - Reusable child flows for consistent error management
4. **Error Context Preservation** - Capturing and passing error details for debugging
5. **Complex Flow Patterns** - Error handling in loops, switches, and nested scenarios

## Getting Started

### Prerequisites

- Power Platform environment (Developer or higher)
- Dataverse database enabled
- Microsoft Teams connection (for notifications)

### Installation

1. Download the latest solution file: `solutions/ErrorHandlinginPowerAutomate_1_0_0_4.zip`
2. Navigate to [Power Apps](https://make.powerapps.com)
3. Select your target environment
4. Go to **Solutions** > **Import solution**
5. Upload the zip file and follow the import wizard
6. Configure the environment variables:
   - `mni_ErrorHandlerEmail` - Email address for error notifications
   - `mni_TestingAPIEndpoint` - Valid API endpoint URL
   - `mni_TestingAPIEndpointInvalid` - Invalid API endpoint (for testing errors)
7. Set up connection references for Dataverse and Teams

### Using the Demo Flows

The solution includes numbered flows that demonstrate progression:

1. Start with **001 No error handling** to see what happens without error handling
2. Review **002 Basic error handling** for simple implementations
3. Study **003 Advanced error handling** for the full Try-Catch-Finally pattern
4. Explore **004** and higher for complex scenarios and child flow patterns
5. Use **999 Template Try Catch Finally scopes** as a starting point for your own flows

## Solution Architecture

The solution uses a custom Dataverse table (`mni_session_errorhandling_cars`) to demonstrate CRUD operations with proper error handling. Flows interact with this table and external APIs to trigger various error scenarios.

## Session Information

This material supports conference presentations on Power Automate error handling:
- **PPCC25** - Power Platform Community Conference 2025
- **EPPC25** - European Power Platform Conference 2025

## License

See [LICENSE](LICENSE) file for details.

## Author

Michael Nielsen - Power Platform Specialist
