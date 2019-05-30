# Automation - Update Sales/Service Cloud from Marketing Cloud

[Marketing Cloud connect](https://help.salesforce.com/articleView?id=mc_co_marketing_cloud_connect.htm&type=5) does not support bidirectional synchronisation, but we can use [AMPscript Salesforce Functions](https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/updatesinglesalesforceobject.htm) to run into a script activity in the *Automation Studio*. 

### LOGIC

1. Populate a data extension with an SQL query or an automated filter. 
   This will contain your rows to update. Of course, you can also do the whole filtering logic in AMPscript.

2. Create a HTML Content Block with your AMPscript

3. Call it in a Script Activity

      ```html
   <script runat="server">
      Platform.Load("Core", "1.1.1");
          var ampscriptCode = Platform.Function.ContentBlockByID("ContentBlockByID");
          var textBlock = TreatAsContent(ampscriptCode);
   </script>
      ```

4. Add the Script Activity into an automation and set the frequency schedule.



### AMPSCRIPT BLOCK SAMPLE

```java
%%[
set @currentSystemTime = NOW()
set @today = SystemDateToLocalDate(@currentSystemTime)
/* Let's assume that the automations runs daily, here we fetch only the records of the day */
set @lookDe = lookuprows("YourDe","DateField",@today)
set @nbLookDe = rowcount(@lookDe)
/* we need to use a for loop to repete the actions for every rows matching the lookup rows criterias */
for @i = 1 to @nbLookDe DO
    set @lineLookDe  = row(@lookDe,@i)
    set @AccountId = field(@lineLookDe,"Id")
    set @PersonContactId = field(@lineLookDe,"PersonContactId")
    set @opportId = field(@lineLookDe,"OpportId")
    set @redirectionLink = CloudPagesURL(123,'subkey',@PersonContactId,'opport',@opportId)
    /* Updates the fields that you want to update here */
    set @updateSfField1 = UpdateSingleSalesforceObject("Account",@offerAccountId,"mcEncryptedLink__c",@redirectionLink)
    set @updateSfField2 = UpdateSingleSalesforceObject("Account",@offerAccountId,"Status","Accepted")
next @i
]%%
```

> :bulb: **Tip** : if you only want to retrieve in the loop all the rows from the DE using the LookupRows function, then you can create a field in your data extension. This field will have a default value (like `default_value`) for all rows. Example : 
>
> ```java
> set @lookDeOffer = lookuprows("YourDE","AmpscriptCol","default_value")
> ```


