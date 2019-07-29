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

	lookUpQuery.csv - It contains the query for Lookup Object
	
	masterChildQuery.csv - It contains the query for Master Object with relative Child Objects

**SFDX Commands for Export Process** 

First, I performed export of the Lookup objects using below command with --plan parameter - 

		sfdx force:data:tree:export -q ./data/Queries/lookUpQuery.csv -d ./data/SampleData --plan
		
This created 2 files in Data/SampleData folder

		LookupObject__cs.json
		LookupObject__c-plan.json (Open the file and make sure you have saveRefs true)
		
Second, I performed export for Master Object and Child Object using below command with --plan parameter - 
		
		sfdx force:data:tree:export -q ./data/Queries/masterChildQuery.csv -d ./data/SampleData --plan
		
This created 3 files in Data/SampleData folder
		
		MasterObject__cs.json
		
		ChildObject__cs.json
		
		MasterObject__c-ChildObject__c-plan.json
		
To make import work successsfully, I had to do some changes in **_MasterObject__cs.json_** and **_MasterObject__c-ChildObject__c-plan.json_** files

**Changes in MasterObject__cs.json**

Open MasterObject__cs.json and add Lookup Object Reference fields in both the Master Records. Fields are added in format "Field Name":"Value", in our case field is a lookup field and its value is a reference to the lookup Object record so format would be - "Field Name":"@referenceId of related lookup record".

As 1st Master Object record is related to 1st Lookup Object records and 2nd Master Object record is related to 2nd Lookup Object record so Reference fields added to MasterObject_cs.json would be as below

	- For first master record add "LookupObject__c":"@LookupObject__cRef1"  (make sure to put comma (,) after the above field to this field)
	
	- For secocnd master record add "LookupObject__c":"@LookupObject__cRef2" (make sure to put comma (,) after the above field to this field)
	
**Changes in MasterObject__c-ChildObject__c-plan.json**

Copy the Lookup Object plan from LookupObject__c-plan.json file, make sure data between [] is copied and paste it in MasterObject__c-ChildObject__c-plan.json file just above the plan for Master Object. Make sure to separate Lookup Object and Master Object plans in MasterObject__c-ChildObject__c-plan.json file with a comma (,).
	
Also please make sure _resolveRefs_ is ture for Master Object.
	
After making above changes now this data is ready for import in a org. 

**Note -** If you want, you can delete the **_LookupObject__c-plan.json_** file as it is no more needed.

# Import Process

For Import, run the below command -
 
	sfdx force:data:tree:import -u tempTest -p ./data/SampleData/MasterObject__c-ChildObject__c-plan.json
	-u_ would hold the alias of the org on which you want to import the data
	-p_ would hold the path of the plan file.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Another Example including Account, Contact, Opportunity, Case and Operating Hours (Lookup on Account)
In Another example I have some Accounts which have related Contact, Opportunity and Case Records. Account have 1 lookup Relationship with Operating Hours object. So first you would need to fetch Operating Hours Object records, using the query in Data/Queries/OperatingHours.csv. I used below command to fetch Operating Hours data. Keeping all fetched data in separate Accounts folder under Data folder.

sfdx force:data:tree:export -q ./data/Queries/OperatingHoursQuery.csv -d ./data/Accounts --plan
		
This created 3 files in Data/Accounts folder

		OperatingHours-Account-plan.json
		OperatingHourss.json
		Accounts.json
		
Second, I performed export for Account and its Child Objects (Contact, Opportunity and Case) using below command with --plan parameter - 
		
		sfdx force:data:tree:export -q ./data/Queries/AccountQuery.csv -d ./data/Accounts --plan
		
This created 4 files in Data/Accounts folder and updated the Accounts.Json file
		
		Accounts.json (Changes done in previous import will be overriden)
		
		Cases.json

		Contacts.Json

		Opportunitys.Json
		
		Account-Case-Contact-Opportunity-plan.json
		
To make import work successsfully, I had to do some changes in **_Accounts.json_** and **_Account-Case-Contact-Opportunity-plan.json_** files

**Changes in Accounts.json**

Open Accounts.json and add Operating Hours Reference fields in all the Account Records. Fields are added in format "Field Name":"Value", in our case field is a lookup field and its value is a reference to the Operating Hours record so format would be - "Field Name":"@referenceId of related lookup record".

	- "OperatingHoursId": "@OperatingHoursRef1"  (make sure to put comma (,) after the above field to this field)
	
**Changes in Account-Case-Contact-Opportunity-plan.json**

Copy the Operating Hours plan from OperatingHours-Account-plan.json file and paste it just above Accounts plan in **Account-Case-Contact-Opportunity-plan.json**. Make sure to separate Operating Hours and other plans in Account-Case-Contact-Opportunity-plan.json file with a comma (,).
	
Also please make sure _resolveRefs_ is ture for Accounts Object.
	
After making above changes now this data is ready for import in a org. Run the following command to import the data in any org:

	sfdx force:data:tree:import -u tempTest -p ./data/Accounts/Account-Case-Contact-Opportunity-plan.json
	-u would hold the alias of the org on which you want to import the data
	-p would hold the path of the plan file.