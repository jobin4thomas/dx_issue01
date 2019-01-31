##dx_issue01
To reproduce the issue regarding Field Dependencies between Managed package picklist fields. This issue is not specific to DX, but still happens with Force.com migration utility.

## Prerequisites 
1. Salesfore CLI 
2. Salesforce DX org and DevHub enabled
3. Any Managed package from Appexchange.
    For ex: **Lightning Carousel and Banner package** [Link](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N3A00000EFp50UAD)


##Steps to reproduce

1. Clone the repo to local
2. Create a new Scratch org (alias = "poc1")
```
sfdx force:org:create -f config\project-scratch-def.json --setalias poc1 --durationdays 30 --setdefaultusername
```
3. Install the managed package **Lightning Carousel and Banner package** on the scratch org ("poc1")
```
sfdx force:package:install --package 04tB00000009XuZIAU --wait 15 --noprompt -u poc1
```
4. Push the source from the repo to scratch org ("poc1")
```
sfdx force:source:push -u poc1
```

##Source push results in following error
```
PS C:\vscode_space\dx_issues01> sfdx force:source:push
=== Pushed Source
STATE  FULL NAME                                                       TYPE         PROJECT PATH
─────  ──────────────────────────────────────────────────────────────  ───────────  ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Add    cloudx_cms__SS_Carousel_Slide__c.cloudx_cms__Text_Alignment__c  CustomField  force-app\main\default\objects\cloudx_cms__SS_Carousel_Slide__c\fields\cloudx_cms__Text_Alignment__c.field-meta.xml
Add    cloudx_cms__SS_Carousel_Slide__c.cloudx_cms__Text_Position__c   CustomField  force-app\main\default\objects\cloudx_cms__SS_Carousel_Slide__c\fields\cloudx_cms__Text_Position__c.field-meta.xml


=== Push Errors
PROJECT PATH                                                                                                         ERROR
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────  ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
force-app\main\default\objects\cloudx_cms__SS_Carousel_Slide__c\fields\cloudx_cms__Text_Alignment__c.field-meta.xml  Cannot modify managed object:
entity=CustomFieldDefinition, component=00N0l000002zssQ, field=PicklistControllerEnumOrId, state=installed (3:13)
```

Note: The source push is partially successfull. **This is also an issue as you can see bit later in next section**
![alt text](https://raw.githubusercontent.com/jobin4thomas/dx_issue01/master/images/FirstPush.png)



##Manual Workaround
1. Go to the scratch org ("poc1")
2. Go to Object>Carousel Slide (cloudx_cms__SS_Carousel_Slide__c)
3. Update the field dependencies as shown below
![alt text](https://raw.githubusercontent.com/jobin4thomas/dx_issue01/master/images/Before.png)
Select Caption Position as Controlling Field and Text Alignment as Dependent Field.
![alt text](https://raw.githubusercontent.com/jobin4thomas/dx_issue01/master/images/After.png)

4. Now do a source push, its says "No results found" as the previous push was **partially successfull**
```
PS C:\vscode_space\dx_issues01> sfdx force:source:push
=== Pushed Source
No results found
```

5. Add an empty space to the cloudx_cms__Text_Alignment__c.field-meta.xml so that we can push the source the third time. It results in success.
```
00:23:48.27 sfdx force:source:push
=== Pushed Source
STATE  FULL NAME                                                       TYPE         PROJECT PATH
─────  ──────────────────────────────────────────────────────────────  ───────────  ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Add    cloudx_cms__SS_Carousel_Slide__c.cloudx_cms__Text_Alignment__c  CustomField  force-app\main\default\objects\cloudx_cms__SS_Carousel_Slide__c\fields\cloudx_cms__Text_Alignment__c.field-meta.xml
Add    cloudx_cms__SS_Carousel_Slide__c.cloudx_cms__Text_Position__c   CustomField  force-app\main\default\objects\cloudx_cms__SS_Carousel_Slide__c\fields\cloudx_cms__Text_Position__c.field-meta.xml
00:23:55.958 sfdx force:source:push ended with exit code 0
```
![alt text](https://raw.githubusercontent.com/jobin4thomas/dx_issue01/master/images/FirstPush.png)


##Conclusion
When we create a field dependency between managed package picklist fields, the same cannot be pushed to a new scratch org without the manual work-around. 

This repo also highlight another issue with Salesforce CLI tooling. Since the first source push status was **Partially Succeeded**, the second source push resulted in **No results found**. There is no option in source push to prevent push if there are errors. This is inconvienient as we had to edit the metadata file so that the push can be done again.