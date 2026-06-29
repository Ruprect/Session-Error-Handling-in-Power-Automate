# Solution XML Templates

All three XML files are required inside the solution zip. Substitute `{{SOLUTION_NAME}}`,
`{{SOLUTION_DISPLAY_NAME}}`, `{{PUBLISHER_NAME}}`, `{{PUBLISHER_PREFIX}}`, `{{VERSION}}` as needed.

---

## [Content_Types].xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
  <Default Extension="xml" ContentType="application/octet-stream" />
  <Default Extension="json" ContentType="application/octet-stream" />
</Types>
```

---

## solution.xml

Replace `{{SOLUTION_NAME}}` (unique name, e.g. `ErrorHandling`),
`{{SOLUTION_DISPLAY_NAME}}` (e.g. `Error Handling`),
`{{PUBLISHER_NAME}}` (e.g. `Ruprect`),
`{{PUBLISHER_PREFIX}}` (e.g. `rup`),
`{{VERSION}}` (e.g. `1.0.3.0`).

Add one `<RootComponent>` per flow. The three standard helpers use these fixed GUIDs:
- `{3fc9a2b1-d4e5-4678-9012-3456789abcde}` — Error Handling Template
- `{5ae8b3c2-f6d7-4891-b023-456789abcdef}` — Helper - Get Error Message
- `{7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef}` — Helper - Send Notification

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImportExportXml version="9.2.26054.147" SolutionPackageVersion="9.2" languagecode="1033" generatedBy="CrmLive" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <SolutionManifest>
    <UniqueName>{{SOLUTION_NAME}}</UniqueName>
    <LocalizedNames>
      <LocalizedName description="{{SOLUTION_DISPLAY_NAME}}" languagecode="1033" />
    </LocalizedNames>
    <Descriptions>
      <Description description="Enterprise error handling template flows for Power Automate using Try-Catch-Finally pattern." languagecode="1033" />
    </Descriptions>
    <Version>{{VERSION}}</Version>
    <Managed>0</Managed>
    <Publisher>
      <UniqueName>{{PUBLISHER_NAME}}</UniqueName>
      <LocalizedNames>
        <LocalizedName description="{{PUBLISHER_NAME}}" languagecode="1033" />
      </LocalizedNames>
      <Descriptions />
      <EMailAddress xsi:nil="true"></EMailAddress>
      <SupportingWebsiteUrl xsi:nil="true"></SupportingWebsiteUrl>
      <CustomizationPrefix>{{PUBLISHER_PREFIX}}</CustomizationPrefix>
      <CustomizationOptionValuePrefix>91517</CustomizationOptionValuePrefix>
      <Addresses>
        <Address><AddressNumber>1</AddressNumber><AddressTypeCode>1</AddressTypeCode><City xsi:nil="true"></City><Country xsi:nil="true"></Country><ShippingMethodCode>1</ShippingMethodCode></Address>
        <Address><AddressNumber>2</AddressNumber><AddressTypeCode>1</AddressTypeCode><City xsi:nil="true"></City><Country xsi:nil="true"></Country><ShippingMethodCode>1</ShippingMethodCode></Address>
      </Addresses>
    </Publisher>
    <RootComponents>
      <RootComponent type="29" id="{3fc9a2b1-d4e5-4678-9012-3456789abcde}" behavior="0" />
      <RootComponent type="29" id="{5ae8b3c2-f6d7-4891-b023-456789abcdef}" behavior="0" />
      <RootComponent type="29" id="{7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef}" behavior="0" />
      <!-- Add additional <RootComponent> entries here for new flows -->
    </RootComponents>
    <MissingDependencies />
  </SolutionManifest>
</ImportExportXml>
```

---

## customizations.xml

One `<Workflow>` block per flow. The helper flows have `<Subprocess>1</Subprocess>`;
the Error Handling Template and any user-added flows have `<Subprocess>0</Subprocess>`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImportExportXml>
  <Workflows>
    <Workflow WorkflowId="{3fc9a2b1-d4e5-4678-9012-3456789abcde}" Name="Error Handling Template">
      <JsonFileName>/Workflows/ErrorHandlingTemplate-3FC9A2B1-D4E5-4678-9012-3456789ABCDE.json</JsonFileName>
      <Type>1</Type>
      <Subprocess>0</Subprocess>
      <Category>5</Category>
      <Mode>0</Mode>
      <Scope>4</Scope>
      <OnDemand>1</OnDemand>
      <TriggerOnCreate>0</TriggerOnCreate>
      <TriggerOnDelete>0</TriggerOnDelete>
      <AsyncAutodelete>0</AsyncAutodelete>
      <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
      <StateCode>1</StateCode>
      <StatusCode>2</StatusCode>
      <RunAs>1</RunAs>
      <IsTransacted>1</IsTransacted>
      <IntroducedVersion>1.0.0.0</IntroducedVersion>
      <IsCustomizable>1</IsCustomizable>
      <BusinessProcessType>0</BusinessProcessType>
      <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
      <ModernFlowType>0</ModernFlowType>
      <PrimaryEntity>none</PrimaryEntity>
      <LocalizedNames>
        <LocalizedName languagecode="1033" description="Error Handling Template" />
      </LocalizedNames>
      <Descriptions>
        <Description languagecode="1033" description="Template demonstrating Try-Catch-Finally error handling pattern. Add your business logic inside the Try scope." />
      </Descriptions>
    </Workflow>
    <Workflow WorkflowId="{5ae8b3c2-f6d7-4891-b023-456789abcdef}" Name="Helper - Get Error Message">
      <JsonFileName>/Workflows/GetErrorMessage-5AE8B3C2-F6D7-4891-B023-456789ABCDEF.json</JsonFileName>
      <Type>1</Type>
      <Subprocess>1</Subprocess>
      <Category>5</Category>
      <Mode>0</Mode>
      <Scope>4</Scope>
      <OnDemand>1</OnDemand>
      <TriggerOnCreate>0</TriggerOnCreate>
      <TriggerOnDelete>0</TriggerOnDelete>
      <AsyncAutodelete>0</AsyncAutodelete>
      <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
      <StateCode>1</StateCode>
      <StatusCode>2</StatusCode>
      <RunAs>1</RunAs>
      <IsTransacted>1</IsTransacted>
      <IntroducedVersion>1.0.1.0</IntroducedVersion>
      <IsCustomizable>1</IsCustomizable>
      <BusinessProcessType>0</BusinessProcessType>
      <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
      <ModernFlowType>0</ModernFlowType>
      <PrimaryEntity>none</PrimaryEntity>
      <LocalizedNames>
        <LocalizedName languagecode="1033" description="Helper - Get Error Message" />
      </LocalizedNames>
      <Descriptions>
        <Description languagecode="1033" description="Child flow: accepts result('Try') and callerWorkflow from caller. Extracts structured error info using coalesced XPath with validation-error support. Returns {Status, ActionName, ErrorMessage, Contents, FlowName, FlowLink}." />
      </Descriptions>
    </Workflow>
    <Workflow WorkflowId="{7cd4e5f6-a8b9-4c2d-b1e3-567890abcdef}" Name="Helper - Send Notification">
      <JsonFileName>/Workflows/HelperSendNotification-7CD4E5F6-A8B9-4C2D-B1E3-567890ABCDEF.json</JsonFileName>
      <Type>1</Type>
      <Subprocess>1</Subprocess>
      <Category>5</Category>
      <Mode>0</Mode>
      <Scope>4</Scope>
      <OnDemand>1</OnDemand>
      <TriggerOnCreate>0</TriggerOnCreate>
      <TriggerOnDelete>0</TriggerOnDelete>
      <AsyncAutodelete>0</AsyncAutodelete>
      <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
      <StateCode>1</StateCode>
      <StatusCode>2</StatusCode>
      <RunAs>1</RunAs>
      <IsTransacted>1</IsTransacted>
      <IntroducedVersion>1.0.2.0</IntroducedVersion>
      <IsCustomizable>1</IsCustomizable>
      <BusinessProcessType>0</BusinessProcessType>
      <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
      <ModernFlowType>0</ModernFlowType>
      <PrimaryEntity>none</PrimaryEntity>
      <LocalizedNames>
        <LocalizedName languagecode="1033" description="Helper - Send Notification" />
      </LocalizedNames>
      <Descriptions>
        <Description languagecode="1033" description="Helper: sends Email or Teams notification with Info/Warn/Error severity. Builds HTML and Adaptive Card templates; delivery action is a placeholder for the caller to wire up." />
      </Descriptions>
    </Workflow>
    <!-- Add additional <Workflow> entries here for new flows (Subprocess=0) -->
  </Workflows>
</ImportExportXml>
```

---

## Cross-solution scenario (Mode C): solution.xml

When creating a **new solution whose flows call helpers from a different solution**, the new
solution.xml lists **only its own business flows** in `<RootComponents>`. The helper flows are
owned by the other solution and must already be installed in the environment.

Key differences from the standard template:
- `<UniqueName>` is the **new** solution name (e.g. `MyBusinessSolution`)
- `<RootComponents>` has one entry per **new business flow** only — no helper GUIDs
- The helpers (`{5ae8b3c2...}`, `{7cd4e5f6...}`) are **intentionally absent**

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImportExportXml version="9.2.26054.147" SolutionPackageVersion="9.2" languagecode="1033" generatedBy="CrmLive" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <SolutionManifest>
    <UniqueName>MyBusinessSolution</UniqueName>
    <LocalizedNames>
      <LocalizedName description="My Business Solution" languagecode="1033" />
    </LocalizedNames>
    <Descriptions>
      <Description description="Business flows using shared error handling helpers from ErrorHandling." languagecode="1033" />
    </Descriptions>
    <Version>1.0.0.0</Version>
    <Managed>0</Managed>
    <Publisher>
      <UniqueName>Ruprect</UniqueName>
      <LocalizedNames>
        <LocalizedName description="Ruprect" languagecode="1033" />
      </LocalizedNames>
      <Descriptions />
      <EMailAddress xsi:nil="true"></EMailAddress>
      <SupportingWebsiteUrl xsi:nil="true"></SupportingWebsiteUrl>
      <CustomizationPrefix>rup</CustomizationPrefix>
      <CustomizationOptionValuePrefix>91517</CustomizationOptionValuePrefix>
      <Addresses>
        <Address><AddressNumber>1</AddressNumber><AddressTypeCode>1</AddressTypeCode><City xsi:nil="true"></City><Country xsi:nil="true"></Country><ShippingMethodCode>1</ShippingMethodCode></Address>
        <Address><AddressNumber>2</AddressNumber><AddressTypeCode>1</AddressTypeCode><City xsi:nil="true"></City><Country xsi:nil="true"></Country><ShippingMethodCode>1</ShippingMethodCode></Address>
      </Addresses>
    </Publisher>
    <RootComponents>
      <!-- Only the new business flows belong to this solution -->
      <RootComponent type="29" id="{A1B2C3D4-E5F6-7890-ABCD-EF1234567890}" behavior="0" />
    </RootComponents>
    <MissingDependencies />
  </SolutionManifest>
</ImportExportXml>
```

## Cross-solution scenario: customizations.xml

Only the new business flows are declared. No helper entries.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImportExportXml>
  <Workflows>
    <Workflow WorkflowId="{A1B2C3D4-E5F6-7890-ABCD-EF1234567890}" Name="My Business Process">
      <JsonFileName>/Workflows/MyBusinessProcess-A1B2C3D4-E5F6-7890-ABCD-EF1234567890.json</JsonFileName>
      <Type>1</Type>
      <Subprocess>0</Subprocess>
      <Category>5</Category>
      <Mode>0</Mode>
      <Scope>4</Scope>
      <OnDemand>1</OnDemand>
      <TriggerOnCreate>0</TriggerOnCreate>
      <TriggerOnDelete>0</TriggerOnDelete>
      <AsyncAutodelete>0</AsyncAutodelete>
      <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
      <StateCode>1</StateCode>
      <StatusCode>2</StatusCode>
      <RunAs>1</RunAs>
      <IsTransacted>1</IsTransacted>
      <IntroducedVersion>1.0.0.0</IntroducedVersion>
      <IsCustomizable>1</IsCustomizable>
      <BusinessProcessType>0</BusinessProcessType>
      <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
      <ModernFlowType>0</ModernFlowType>
      <PrimaryEntity>none</PrimaryEntity>
      <LocalizedNames>
        <LocalizedName languagecode="1033" description="My Business Process" />
      </LocalizedNames>
      <Descriptions>
        <Description languagecode="1033" description="Business flow using shared error handling helpers." />
      </Descriptions>
    </Workflow>
  </Workflows>
</ImportExportXml>
```
