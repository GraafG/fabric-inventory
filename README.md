# Fabric Inventory Explorer: recipient guide

## Can I start with only the notebook?

**Yes. Upload `Fabric_Inventory_Template.ipynb`. No local Python files, Azure CLI, or deployment scripts are required.**

You still need a Fabric workspace and an active Fabric capacity. The notebook does not create, buy, resize, or resume capacity.

The notebook can create a dedicated lakehouse and, later, a semantic model and Power BI report.
These are separate, explicit actions. Uploading or opening the notebook does not create resources.

## Minimum setup

1. Upload the notebook to a suitable Fabric workspace. Select a supported **PySpark** runtime.
2. Read the first Markdown cells. The shared notebook starts with `MODE = "help"`.
3. Attach an existing **dedicated, schema-enabled inventory lakehouse** as the default.
4. If none exists, set `MODE = "setup_lakehouse"` and choose `SETUP_LAKEHOUSE_NAME`. Run the notebook.
5. Attach the returned lakehouse in the notebook Explorer. Restart the session so the default binding updates.
6. Set `MODE = "collect_and_view"` and `CONFIRM_WRITE_TARGET = True`. Run the notebook.
7. Return `MODE` to `"view_saved"` for ordinary browsing.

The notebook displays the resolved workspace and lakehouse before collection.
Do not confirm a business-data lakehouse as the inventory destination.

**Normally, leave all tenant, workspace, and lakehouse ID overrides blank.**
The notebook reads the attached-lakehouse context and the signed-in tenant.
Explicit overrides must match that context. They cannot redirect writes to another target.

## What should I configure?

| Setting | Configure it when | Meaning |
|---|---|---|
| `MODE` | Every time you choose a different action | `help`, `setup_lakehouse`, `view_saved`, `collect_and_view`, or `create_report` |
| `SETUP_LAKEHOUSE_NAME` | You need a new inventory lakehouse | A dedicated name, such as `FabricInventory`; existing items are not replaced |
| `CONFIRM_WRITE_TARGET` | Before the first collection | Confirms the displayed destination; it does **not** authorize overwrite |
| `REPORT_MODEL_NAME`, `REPORT_NAME` | You want the optional Power BI output | Choose unique names if unrelated items already use the defaults |
| `COLLECT_WORKSPACE_IDS`, `COLLECT_CAPACITY_IDS`, `COLLECT_ITEM_TYPES` | You need a narrower scan | Collection filters, not viewing filters |
| `INCLUDE_DEEP_METADATA`, `INCLUDE_PERMISSIONS`, `INCLUDE_USAGE` | You want to change collection depth | Defaults collect each supported category, subject to access and service limits |
| `EXPAND_GROUPS`, `GRAPH_CREDENTIAL` | Approved Graph access is already available | Optional group expansion; no new credentials or consent are created |
| `TABLE_PREFIX` | You need a separate inventory table set | Set it before the first collection; the report uses the same prefix |
| `WRITE_MODE` | Normally leave unchanged | `append_snapshot` preserves history |
| `CONFIRM_OVERWRITE` | Only for an intentional destructive reset | Must be Boolean `True` together with `WRITE_MODE = "replace_inventory"` |
| Tenant/workspace/lakehouse overrides | Advanced use only | Normally blank; incorrect overrides stop the action with a destination-mismatch message |
| `REPORT_MODEL_ID`, `REPORT_ID` | Advanced reuse of an existing inventory deployment | Verify the IDs first; `create_report` never overwrites existing definitions |

`VIEW` controls affect saved-data exploration only. Changing them does not collect data or write tables.

## Create the optional report from the notebook

1. Complete a collection so a published snapshot exists.
2. Set `MODE = "create_report"`.
3. Review `REPORT_MODEL_NAME` and `REPORT_NAME`, then run the notebook.
4. Open the returned report link. Return `MODE` to `"view_saved"` afterward.

The action creates a Direct Lake semantic model, initializes it, and creates the report in the inventory lakehouse workspace.
All report/model definitions are embedded in the notebook.

The report is optional. Notebook views remain useful if report creation is denied.
Missing report rights do not erase inventory data or turn a failed report creation into a reported success.

Repeated report setup reuses items identified as this inventory's managed deployment.
An unrelated name collision stops setup. It does not replace or rebind that item.
If only the model was created before an error, its ID is reported for recovery.

## Which rights are required?

| Operation | Typical requirements | Important distinction |
|---|---|---|
| Read the instructions | Access to the notebook | No source inventory is queried |
| Run the notebook | Notebook execution rights and an available Fabric capacity | Workspace Contributor or higher is the straightforward authoring route |
| Browse saved inventory | Read access to the inventory lakehouse and notebook execution rights | Fabric tenant-admin rights are not needed |
| Create the lakehouse or write inventory | Suitable authoring/write rights in the destination workspace/lakehouse | Azure subscription Owner does not grant Fabric workspace access |
| Tenant-wide admin inventory | Fabric administrator role and a suitable delegated admin token | Workspace Admin is not Fabric administrator |
| Accessible-workspace fallback | Access to those workspaces/items | This is not a complete tenant inventory |
| Deep model/table metadata | Workload-specific read/metadata access; model Read/Build where required | Admin catalog access does not automatically grant data-plane access |
| Activity and tenant-wide permission APIs | Their admin role and token scopes | If unavailable, usage/permissions are labeled unavailable, not zero |
| Create a Power BI model/report | Destination authoring rights, an appropriate Pro/PPU/trial license, and source access | A capacity does not replace the report author's license |
| Expand groups | An existing supported Graph credential and approved membership-read permissions | A Fabric role does not grant Microsoft Graph permissions |

For ordinary Power BI viewing on an F capacity below F64, viewers also need an appropriate per-user license.
Default Direct Lake SSO requires the viewer to have suitable access to the inventory data.

## What happens when rights are missing?

- **Admin API denied:** permitted standard APIs are used where an equivalent exists. Scope labels show the reduced coverage.
- **Source item denied:** the catalog item remains visible. Its missing detail is not shown as an empty source.
- **Token scope missing:** the message identifies an authentication-scope issue. Adding a workspace role alone may not solve it.
- **Sign-in expired or wrong tenant:** the action stops. Sign in to the correct Fabric tenant and renew the session.
- **Capacity paused:** the notebook explains the unavailable detail. It does not resume the source.
- **API throttled:** remaining entities are deferred with a retry explanation. Previous snapshots remain intact.
- **Resource not found:** check both existence and visibility. A 404 is not proof of deletion.
- **Report creation denied:** the error names the failed action and required authoring/license checks. Any created model is identified.

Use **Diagnostics** for the collector, entity, HTTP code, service code, reason, and recommended next step.
Never enable overwrite to resolve an access problem.

## Share the template, not the results

Distribute only the clean template notebook and this guide from the external-sharing package.
The generated template has no tenant IDs, resource IDs, local paths, runtime outputs, or inventory records.
Review the code before running it. Package checksums detect changes; they do not authenticate the publisher.

Do **not** distribute the internal deployment ZIP, configured notebook, `deployment.json`, live validation files, or inventory lakehouse.
An executed notebook can retain names, user identifiers, permissions, and usage in its outputs.
Clearing outputs alone does not remove configured IDs, attached-lakehouse metadata, or embedded report filters.

If you intend to share actual inventory results, treat that as a separate access and privacy review.
Tenant metadata, identities, group membership, and usage can be sensitive.
Keep raw inventory restricted; share a minimized, approved extract when appropriate.
Power BI RLS does not protect raw lakehouse tables from people who can read them directly.
Guest access, licensing, tenant settings, and source-data permissions need separate review.

This package does not publish to the web, add guests, grant roles, or share a workspace.

## Limits and operating model

- Collection is manual. No schedule or pause/resume automation is created.
- Activity retention is rolling and limited. Older complete UTC days may fall outside the service window.
- MAU is based on observed, identifiable actors. Incomplete windows remain labeled partial.
- Permission rows are observed assignments, not a full SQL/RLS/OLS/OneLake effective-access audit.
- Snapshots preserve history. Comparisons require compatible scope and coverage.
- Do not run multiple writers against the same inventory destination.
- Package versions, service capabilities, and Preview APIs can change. Verify in a limited workspace before wider use.
