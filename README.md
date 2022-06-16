# Backup Azure AD Security Groups
Put your hand up if you have ever deleted an Azure AD Security Group by accident… 🙋‍♂️

Unlike Microsoft 365 Groups, AAD Security Groups (Cloud Only) can’t be restored at the click of a button. Maybe you're using them on your Power Platform environment, or perhaps it is linked to your Conditional Access policies. When this happens, you may need to reinstate a group quickly, but can you remember exactly who was in it? I know I wouldn't 😰

Most people don’t even stop to think that Azure AD security groups can’t be restored, because it’s 2022… we’ve had recycle bins for ages, right? Meaning we only realise this after an accidental deletion, by which point it’s too late. Well, I have designed a simple Power Automate flow to backup your group memberships for you 😊

This solution can easily tweaked to suit your needs, I am giving you a simple example which backs up one AAD Security Group to CSV and puts it in to a SharePoint library. The flow auto-deletes files 30 days and older from the SharePoint library, you can tweak this file retention to however best suits you! 

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Pre-requisites: 
- An E1/A1, E3/A3 or E5/A5 license with the feature "Power Automate for Office 365" enabled.
- The account must have access to the SharePoint library you wish to publish to. 
- Be able to read Azure AD security group memberships. Directory Reader role will accomplish this.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Time to create our flow!

1)	I’m going to set the Recurrence for my task to run daily. This is sufficient enough for my particular use. 
2)	Now add a new action step and use the Azure AD connector and we are going to use “Get group members”.
 ![image](https://user-images.githubusercontent.com/72546386/174013460-a9f06234-5842-4233-8156-61a58ed2ca7f.png)

3)	Enter in the Azure AD Object ID for your security group.
 ![image](https://user-images.githubusercontent.com/72546386/174013487-cfcdbd3d-83e0-4b03-a7b7-c5a2a8ee0e96.png)

4)	I have a large group, so I am going to adjust the settings on this action and enable pagination. I will set the threshold to 3000. This will allow the action to fetch more members than it would at the default setting. By default, it will only fetch your first 100 members.
 ![image](https://user-images.githubusercontent.com/72546386/174013505-252dd8b3-77af-4f37-a3cf-63fa9b6b198a.png)

5)	Once you have done this. Add a new Action step. This time it’s going to be a Data Operation action called “Create CSV Table”.
 ![image](https://user-images.githubusercontent.com/72546386/174013530-10829b6f-9fac-4903-9fad-f3ffce6deb83.png)


6)	In the From field, use your dynamic content from the previous ‘Get group members’ operation. I have designed my CSV table like below:
 ![image](https://user-images.githubusercontent.com/72546386/174013554-86364917-55ee-4687-bc91-b7c39eb3f05e.png)

DisplayName field uses DisplayName from our dynamic content.

 ![image](https://user-images.githubusercontent.com/72546386/174013581-2ff0756e-506e-4d73-ab86-3b44f6f03040.png)

UPN uses the UserPrincipalName from our dynamic content.

 ![image](https://user-images.githubusercontent.com/72546386/174013599-0724ad73-0512-4e3f-b7a8-8dfa89fbd2a7.png)

JobTitle uses the JobTitle from our dynamic content.

 ![image](https://user-images.githubusercontent.com/72546386/174013626-363a4425-36c3-4156-97b4-036b57edc4fc.png)

ObjectID uses the ID from our dynamic content.

 ![image](https://user-images.githubusercontent.com/72546386/174013649-84d4a082-a702-4418-998c-78d27f2470d6.png)

7)	Our next Action Step is a SharePoint action of ‘Create file’. I have chosen my SharePoint site address from the drop-down menu and I have browsed to the folder location I wish to deposit my CSV from the folder icon button. The file content is the Output of our ‘Create CSV table’ action in our Dynamic Content.
 ![image](https://user-images.githubusercontent.com/72546386/174013658-98089250-bd51-4d8d-975a-fd35a009ae4d.png)

8)	For the file name, to add a date and timestamp I have used an Expression of utcNow().
 ![image](https://user-images.githubusercontent.com/72546386/174013668-ce8a8aab-4475-4f2a-a4e3-4de2f9900816.png)

9)	Create a new step of another SharePoint action. This time ‘List folder’. Select the same Site address as above. For the File identifier you will need to browse to the same folder location.
 ![image](https://user-images.githubusercontent.com/72546386/174013689-2f1c0eb6-f839-4d98-86eb-a396f38e8c91.png)

10)	 For your next step at a Control action of Apply to each.
 ![image](https://user-images.githubusercontent.com/72546386/174013702-22fc1dd5-fa09-45d5-a53e-b9d79fa1228e.png)

11)	 Select the output from previous steps needs to be the Body from our List folder operation.
 ![image](https://user-images.githubusercontent.com/72546386/174013712-a8d048af-f4a1-4bc7-8a7b-754b525ca386.png)

12)	 Add in a Condition action. Set it up like below:
![image](https://user-images.githubusercontent.com/72546386/174013817-ddf2f8a1-0558-48a9-87b1-2363cca1da24.png)

Add 'LastModified' from our 'List folder' dynamic content. Set the operator to ‘is less than’. Set the evaluator to an expression. The value of the expression should read: 
~~~
addDays(utcNow(),-29)
~~~

![image](https://user-images.githubusercontent.com/72546386/174013834-3fc2d3bc-227d-46e5-9ea2-d90f8ca05f7f.png)

Add another match criteria for 'IsFolder' from the dynamic content of our 'List folder' dynamic content. Set the evaluator to is equal to. Then use an expression of false.

 ![image](https://user-images.githubusercontent.com/72546386/174013869-e23dbe7e-11a0-4731-98f6-2726b13ba05b.png)
 
13)	 For your If Yes use the SharePoint action of Delete file. 
14)	 Select your Site Address like you have done previously.
15)	 In File Identifier use ‘Id’ from your List folder action.

 ![image](https://user-images.githubusercontent.com/72546386/174013895-bb9ebb64-cfd2-41dc-b2cd-762388321958.png)

16)	 Your Apply to each control should look like this:
 ![image](https://user-images.githubusercontent.com/72546386/174013938-fc8a1263-2ad3-40a5-b09f-b93207b91552.png)


And there we have it. Your security groups are now being backed up daily to your SharePoint library as a CSV file. Now, if a group is ever deleted by accident, you can create a new one and use Azure AD Bulk Operations to import your members.
 ![image](https://user-images.githubusercontent.com/72546386/174013956-3d045c68-694e-4e43-bd30-9efcf88c8c3d.png)


Download the import template, copy over the list of UPNs from your backup CSV and you’re good to go! I hope this helps. 😊

-----------------------------------------------------------

**Your flow from start to finish should look similar to this:**
![image](https://user-images.githubusercontent.com/72546386/174015189-ac99e5ec-00d5-4802-9fed-78f045ded8d3.png)

![image](https://user-images.githubusercontent.com/72546386/174015489-1529f571-d6f9-4532-a295-20ca456165a5.png)

![image](https://user-images.githubusercontent.com/72546386/174015547-99ed0c9b-e8e3-4c6f-8e62-5a8dea7a4024.png)



