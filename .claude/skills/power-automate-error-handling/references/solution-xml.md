# Solution XML Templates

All three XML files are required inside the solution zip. Substitute `{{SOLUTION_NAME}}`,
`{{SOLUTION_DISPLAY_NAME}}`, `{{PUBLISHER_NAME}}`, `{{PUBLISHER_PREFIX}}`, `{{VERSION}}` as needed.

GUIDs shown in examples below are **placeholders only** — the skill generates fresh GUIDs per
deployment using `[System.Guid]::NewGuid()`. See Step 2A in SKILL.md.

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
`{{VERSION}}` (e.g. `1.0.0.0`).

Add one `<RootComponent>` per flow using the GUIDs generated at build time.
**GUIDs must be lowercase** — uppercase causes a false "component is not declared as a root component" error.

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
      <RootComponent type="29" id="{{{GUID_TEMPLATE}}}" behavior="0" />
      <RootComponent type="29" id="{{{GUID_GET_ERROR}}}" behavior="0" />
      <RootComponent type="29" id="{{{GUID_SEND_NOTIF}}}" behavior="0" />
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

When any connectors are wired up (Office 365 and/or Teams), include a `<connectionreferences>`
block **before** the closing `</ImportExportXml>` tag. Omit it entirely when no connectors are used.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImportExportXml>
  <Workflows>
    <Workflow WorkflowId="{{{GUID_TEMPLATE}}}" Name="Error Handling Template">
      <JsonFileName>/Workflows/ErrorHandlingTemplate-{{GUID_TEMPLATE_UPPER}}.json</JsonFileName>
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
    <Workflow WorkflowId="{{{GUID_GET_ERROR}}}" Name="Helper - Get Error Message">
      <JsonFileName>/Workflows/GetErrorMessage-{{GUID_GET_ERROR_UPPER}}.json</JsonFileName>
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
      <IntroducedVersion>1.0.0.0</IntroducedVersion>
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
    <Workflow WorkflowId="{{{GUID_SEND_NOTIF}}}" Name="Helper - Send Notification">
      <JsonFileName>/Workflows/HelperSendNotification-{{GUID_SEND_NOTIF_UPPER}}.json</JsonFileName>
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
      <IntroducedVersion>1.0.0.0</IntroducedVersion>
      <IsCustomizable>1</IsCustomizable>
      <BusinessProcessType>0</BusinessProcessType>
      <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
      <ModernFlowType>0</ModernFlowType>
      <PrimaryEntity>none</PrimaryEntity>
      <LocalizedNames>
        <LocalizedName languagecode="1033" description="Helper - Send Notification" />
      </LocalizedNames>
      <Descriptions>
        <Description languagecode="1033" description="Helper: sends Email (Office 365) or Teams adaptive card notification with Info/Warn/Error severity." />
      </Descriptions>
    </Workflow>
    <!-- Add additional <Workflow> entries here for new flows (Subprocess=0) -->
  </Workflows>

  <!-- Include this block only when connectors are wired (Q6 != Neither). Omit entirely otherwise. -->
  <connectionreferences>
    <!-- Include when Email connector is wired -->
    <connectionreference connectionreferencelogicalname="{{PUBLISHER_PREFIX}}_office365_errorhandling">
      <connectionreferencedisplayname>Office 365 Outlook</connectionreferencedisplayname>
      <connectorid>/providers/Microsoft.PowerApps/apis/shared_office365</connectorid>
      <iscustomizable>1</iscustomizable>
      <statecode>0</statecode>
      <statuscode>1</statuscode>
    </connectionreference>
    <!-- Include when Teams connector is wired -->
    <connectionreference connectionreferencelogicalname="{{PUBLISHER_PREFIX}}_teams_errorhandling">
      <connectionreferencedisplayname>Microsoft Teams</connectionreferencedisplayname>
      <connectorid>/providers/Microsoft.PowerApps/apis/shared_teams</connectorid>
      <iscustomizable>1</iscustomizable>
      <statecode>0</statecode>
      <statuscode>1</statuscode>
    </connectionreference>
  </connectionreferences>

</ImportExportXml>
```

---

## Cross-solution scenario (Mode C): solution.xml

When creating a **new solution whose flows call helpers from a different solution**, the new
solution.xml lists **only its own business flows** in `<RootComponents>`. The helper flows are
owned by the other solution and must already be installed in the environment.

The helper GUIDs in the business flow JSON's `workflowReferenceName` must match what is **currently
installed** in the environment — discovered by exporting the helper solution (see Step 2C in SKILL.md).

Key differences from the standard template:
- `<UniqueName>` is the **new** solution name (e.g. `MyBusinessSolution`)
- `<RootComponents>` has one entry per **new business flow** only — no helper GUIDs
- The helpers are **intentionally absent**

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
      <!-- GUIDs in RootComponent id MUST be lowercase -->
      <RootComponent type="29" id="{a1b2c3d4-e5f6-7890-abcd-ef1234567890}" behavior="0" />
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
    <Workflow WorkflowId="{a1b2c3d4-e5f6-7890-abcd-ef1234567890}" Name="My Business Process">
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
