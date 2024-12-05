# Commerce Demo Generator

## Useful Links

* [📖 → Scratch Org Features Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs_def_file_config_values.htm)
* [📖 → Metadata CommerceSettings Documentation](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_commercesettings.htm)

## Via API & SF CLI

### 0. Prerequisites

#### Salesforce CLI

Ensure the Salesforce CLI is installed and up to date by running the following commands in a terminal

```bash
sf -v # prints cli version
```

```bash
sf update # updates cli
```

### 1. Create Scratch Org

Create Scratch Org using the _sf cli_

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_create_scratch_unified)

```bash
sf org create scratch 
--alias commerce-demo
--target-dev-hub devhub@user.com # dev-hub username 
--username scratchorg@user.com # scratch-org username
--description "" # provide meaningful description what the org is used for
--name "" # name of the org
--definition-file config/project-scratch-def.json # path to definition file
--no-ancestors
--no-namespace
--duration-days 30
--wait 15 # timout 15min for creation
--json
```

### 2. Deploy default metadata

Deploy default metadata as base for org and commerce functionality. This might include:

* apex classes
* permission sets & groups
* lightning web components
* digital experience bundle
  * to be connected to new stores

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_project_commands_unified.htm#cli_reference_project_deploy_start_unified)

> [!WARNING] 
> You must run this command from within a project.

```bash
sf project deploy start 
-o scratchorg@user.com # scratch-org username
--source-dir force-app # path to 'force-app' project metadata-root-folder
--ignore-conflicts 
--dry-run
```

### 3. Import Data

Import simple or complex data via different sf cli or rest api calls.
In general multiple data plans are used to import unrelated records as independent as possible from each other. (In future use that makes it easier to track created records, abort import and roll back _by deleting created records_).

Exmaple folder scturcture:

* commerceDataImport/
  * commerceData-plan.json
  * Account.json
  * Contact.json
  * WebStore.json
  * Catalog.json
  * WebStoreCatalog.json
  * [...]
* setupImport/
  * setup-plan.json
  * CspTrustedSite.json
* productImport/
  * product-import-mapping.json
  * product-data.csv

[📖 → Import Tree by Plan Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_data_commands_unified.htm#cli_reference_data_import_tree_unified) | [📖 → Import Product Data Documentation](./README.md#33-import-product-data)

#### 3.1 Import Commerce Data

This includes regular data like accounts and contact, as well as commerce setup object that have references like catalog and the webstore itself - also includes the required junctions objects eg. _WebStoreCatalog_.

> [!WARNING] 
> Save results - RecordIds needed for import of Product Data and its mapping

```bash
sf data import tree 
-o scratchorg@user.com # scratch-org username
--plan commerceDataImport/commerceData-plan.json # path to data plan json file
```

#### 3.2 Import Setup Data

Import records that are controlling behaviour like a setup setting would, but is not a setting or metadata but rather regular data-record (eg. _CspTrustedSite_).

```bash
sf data import tree 
-o scratchorg@user.com # scratch-org username
--plan setupImport/setup-plan.json # path to data plan json file
```

#### 3.3 Import Product Data

> __*TODO*__: Check if entitlement in CSV or Mapping!

##### 3.3.1 Adjust & Upload CSV

Create or adjust the CSV file as needed. [📖 → CSV Structure Documentation](https://help.salesforce.com/s/articleView?id=commerce.comm_prod_import_csv.htm&type=5)

[📖 → CSV File Upload Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_data_commands_unified.htm#cli_reference_data_create_file_unified)

```bash
sf data create file 
-o scratchorg@user.com # scratch-org username
-t productImportCsvFile # file/document name 
-f productImport/product-data.csv # path to file to upload
```

##### 3.3.2 Adjust Mapping & Trigger Import

Review the mapping json file that used when triggering the import job. It ensure the usage of the cirrect store etc.

[📖 → Import Product- Import-Mapping REST Resquest Documentation](https://developer.salesforce.com/docs/atlas.en-us.chatterapi.meta/chatterapi/connect_resources_commerce_import_product_job_create.htm)

```json
{
   "importConfiguration": {
      "importSource": {
         "contentVersionId": "record id of uploaded csv-file (068)"
      },
      "importSettings": {
         "category": {
            "productCatalogId": "record id of created product-catalog of store (0ZS)"
         },
         "price": {
            "pricebookAliasToIdMapping": {
               "csvHeaderAlias": "record id of created pricebook2 for alias from CSV column header (01s)"
            }
         },
         "entitlement": {
            "defaultEntitlementPolicyId": "record id of created entitlement policy for that store (1Ce)"
         },
         "webstore": {
            "webstoreId": " record id of the created webstore (0ZE)"
         }
      }
   }
}
```

Now start the actual import job via a call to the rest api. Either use [_sf api request rest_ from the CLI](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_api_commands_unified.htm#cli_reference_api_request_rest_unified) or a direct http request via eg _cURL_. Get access-token with [get org details command](./README.md#get-org-details)

```bash
curl --location 'https://<scratch-org>.scratch.my.salesforce.com/services/data/v<API-Version>/commerce/management/import/product/jobs' 
--header 'Content-Type: application/json' 
--header 'Authorization: Bearer <access-token>' 
--data '{
   "importConfiguration": {
      "importSource": {
         "contentVersionId": "record id of uploaded csv-file (068)"
      },
      "importSettings": {
         "category": {
            "productCatalogId": "record id of created product-catalog of store (0ZS)"
         },
         "price": {
            "pricebookAliasToIdMapping": {
               "csvHeaderAlias": "record id of created pricebook2 for alias from CSV column header (01s)"
            }
         },
         "entitlement": {
            "defaultEntitlementPolicyId": "record id of created entitlement policy for that store (1Ce)"
         },
         "webstore": {
            "webstoreId": " record id of the created webstore (0ZE)"
         }
      }
   }
}'
# alternatively specify the adjusted file to be used directly instead of copy the contents:
#-d @productImport/product-import-mapping.json
```

This starts an asynchronus import job and returns that job id in the a json structure omething like the following:

```json
{
    "contentVersionId": "068KF000001vZdUYAU",
    "endTime": null,
    "errorMessages": {},
    "jobId": "750[...]",
    "startTime": 1733300056454,
    "startedBy": "005[...]",
    "status": "UploadComplete",
    "warningMessages": {},
    "[...]"
}
```

##### 3.3.3 Job Results

The CLI (eg. `sf data bulk results`) is not supporting commerce bulk operation from the _/commerce/management_ endpoints. So there are the following options to get information about the import job

1. In Salesforce navigatio to the pdoruct import page `/lightning/page/commerceProductImport` to see the status of the latest import. To open the UI, login automatically and navigate to that page use the following coammdn (if lacy 😉): `sf org open --target-org scratchorg@user.com --path "/lightning/page/commerceProductImport"`
2. [_sf api request rest_ from the CLI](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_api_commands_unified.htm#cli_reference_api_request_rest_unified)
3. a direct http request via eg _cURL_  

```bash
    curl --location 'https://<scratch-org>.scratch.my.salesforce.com/services/data/v62.0/commerce/management/import/product/jobs/<jobId>' 
    --header 'Content-Type: application/json' 
    --header 'Authorization: Bearer <access-token>'
```

Example json esult when the job is finished:

```json
{
  "cmsWorkspaceId": null,
  "commerceEntitlementPolicyId": "1Ce",
  "commerceEntitlementProductsCreated": 0,
  "contentVersionId": "068",
  "endTime": 1733300059673,
  "errorMessages": {},
  "jobId": "750",
  "numberError": 0,
  "numberSuccess": 2,
  "numberToProcess": 2,
  "numberWarning": 2,
  "pricebookAliasToIdMapping": {
    "sales": "01s"
  },
  "pricebookEntriesCreated": 2,
  "pricebookEntriesUpdated": 2,
  "processTime": 3219,
  "productAttributeSetProductsCreated": 0,
  "productAttributesCreated": 0,
  "productAttributesUpdated": 0,
  "productCatalogId": "0ZS",
  "productCategoriesCreated": 0,
  "productCategoryProductsCreated": 0,
  "productMediaCreated": 0,
  "productMediaUpdated": 0,
  "productSellingModelCreated": 0,
  "productSellingModelOptionCreated": 0,
  "productsCreated": 0,
  "productsUpdated": 2,
  "sampleData": null,
  "slugsCreated": 0,
  "slugsUpdated": 0,
  "startTime": 1733300056454,
  "startedBy": "005",
  "status": "JobComplete",
  "warningMessages": {
    "2": [
      "Add a URL for Media STANDARD 2."
    ]
  },
  "webstoreId": "0ZE"
}
```

## Utils

### [📖 → JSON Responses & Schemas Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_dev_cli_json_support.htm)

### Generate Password

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs_passwd.htm)

```bash
sf org generate password 
--target-org scratchorg@user.com # scratch-org username
```

### Get Limits

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_list_limits_unified)

```bash
sf org list limits
 --target-org scratchorg@user.com # scratch-org username
```

### Get Org Details

> [!WARNING] 
> Exposes critical info like __password__ and __access-token__

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_display_unified)

```bash
sf org display 
-o scratchorg@user.com # scratch-org username
```

### Open Org Page

[📖 → Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_open_unified)

```bash
sf org open 
--target-org scratchorg@user.com # scratch-org username
--path "/lightning/setup/AsyncApiJobStatus/page?address=%2F750"
```
