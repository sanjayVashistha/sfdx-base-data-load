# Export/Import Data using data tree

This application is created as examples for exporting and importing data using "sfdx force:data:tree:export" and "sfdx force:data:tree:import".

**Object Schema of the application -**

In the example org we have 1 Master-Detail relationship between 2 objects and 1 Lookup relations ship between Master and another Object. All three Objects have different data type fields

	MasterObject__c (Object)
		LookupObject__c (Field - Lookup Relationship to Lookup Object)
		MultiSelectPicklist__c (Field - MultiSelect PickList)
		NumberField__c (Field - Number (16,2))
		
			ChildObject__c (Object)
				FormulaField__c (Field - Formula (Number))
				MasterObject__c (Field - Master:Detail Relationship)
				TextField__c (Field - Text (80))
	
	LookupObject__c (Object)
		DateTimeField__c (Field - Date Time)
		
# Export Process

For Export, I have 2 queries : 1 for Lookup Object records and another for Master Object and Child Object Records.
I have created 2 query files in data/Queries folder

	_*lookUpQuery.csv*_ - It contains the query for Lookup Object
	
	_*masterChildQuery.csv*_ - It contains the query for Master Object with relative Child Objects

**SFDX Commands for Export Process** 

First, I performed export of the Lookup objects using below command with --plan parameter - 

		**sfdx force:data:tree:export -q ./data/Queries/lookUpQuery.csv -d ./data --plan**
		
This created 2 files in Data folder

		_*LookupObject__cs.json*_
		_*LookupObject__c-plan.json*_ (Open the file and make sure you have saveRefs true)
		
Second, I performed export for Master Object and Child Object using below command with --plan parameter - 
		
		**sfdx force:data:tree:export -q ./data/Queries/masterChildQuery.csv -d ./data --plan**
		
This created 3 files in Data folder
		
		_*MasterObject__cs.json*_
		
		_*ChildObject__cs.json*_
		
		_*MasterObject__c-ChildObject__c-plan.json*_
		
To make import work successsfully, I had to do some changes in _*MasterObject__cs.json*_ and _*MasterObject__c-ChildObject__c-plan.json*_ files

* **Changes in MasterObject__cs.json** *

	Open MasterObject__cs.json and add Lookup Object Reference fields in both the Master Records. Fields are added in format _*"Field Name":"Value"*_, in our case field is a lookup field and its value is a reference to the lookup Object record so format would be - _*"Field Name":"@referenceId of related lookup record"*_. 
As 1st Master Object record is related to 1st Lookup Object records and 2nd Master Object record is related to 2nd Lookup Object record so Reference fields added to MasterObject_cs.json would be as below

	- For first master record add _*"LookupObject__c":"@LookupObject__cRef1"*_  **(make sure to put comma (,) after the above field to this field)**
	
	- For secocnd master record add _*"LookupObject__c":"@LookupObject__cRef2"*_  **(make sure to put comma (,) after the above field to this field)**
	
_Changes in *MasterObject__c-ChildObject__c-plan.json*_

	Copy the Lookup Object plan from _*LookupObject__c-plan.json*_ file, make sure data between [] is copied and paste it in _*MasterObject__c-ChildObject__c-plan.json*_ file just above the plan for Master Object. Make sure to separate Lookup Object and Master Object plans in _*MasterObject__c-ChildObject__c-plan.json*_ file with a comma (,).
	
	Also please make sure _resolveRefs_ is ture for Master Object.
	
After making above changes now this data is ready for import in a org. 

**Note -** If you want, you can delete the _*LookupObject__c-plan.json*_ file as it is no more needed.

# Import Process

For Import, run the below command -
 
	**sfdx force:data:tree:import -u tempTest -p ./data/MasterObject__c-ChildObject__c-plan.json**
	__-u_ would hold the alias of the org on which you want to import the data__
	__-p_ would hold the path of the plan file.__
	
# Key things to Remember 
	- Name fields should not be queried if AutoNumber.
	- Formula fields should not be queried.
	- Lookup fields should not be queried and should be added later.
	- If you want to populate Lookup Field on an object record then make resolveRefs true in plan for that object.
	- If you have queried a number field with decimal values then during import a rounded value will be populated with having 0 in decimals.
	- Value of Lookup field in Json file should always start with @