# UpdateSetBestPractice
Update set practice
{{Technical Best Practices}}
[[Category:Development Best Practices]]

= Overview = 

An [[System Update Sets|update set]] is a group of customizations that can be moved from one instance to another. This feature allows administrators to group a series of changes into a named set and then move them as a unit to other instances. Update sets allow customizations to be developed in a development instance, moved to a test instance, and then applied to a production instance. 

For information about planning the update process and avoiding common pitfalls, see [[Getting Started with Update Sets]]. To use update sets even more effectively, follow the best practices described on this page.  

= Update Set Management = 

Update sets capture configuration information but not task or process data. For example, update sets track service catalog item definitions and related configuration data like variables and variable choices. However, if you test the service catalog by placing orders, the orders (requests, items, catalog tasks) are not tracked by update sets.

== Follow Update Source Requirements ==

To avoid transfer errors create only one update source record per instance source. Each update source must have a unique URL. See [[Transferring_Update_Sets#Retrieving_Update_Sets|Transferring Update Sets]] for more information.

For example, this is a valid list of update sources:

{| class="standard"
|-
! Name
! Type
! URL
|-
| Feature development
| Development
| https://feature_dev.service-now.com/
|-
| Patch development
| Development
| https://patch_dev.service-now.com/
|}

{{Warning|Do not create multiple update source records with the same URL.}}

== Define Naming Standards ==

A consistent naming system for update sets makes it easier to coordinate changes from multiple developers or when applying changes from one instance to another one. Some examples of information to use in update set names include:

*Release names
*Sprint names
*Version numbers
*Record numbers
*Dates
*Developer's initials

You may also want to use the following suggestions for naming update sets:

*If an update set is being generated to fix a problem, consider including the problem ticket in the name. For example, ''PRB10005 - Duplicate Email Issues Fix''.
*If more than one update set is needed to address a problem, include a sequence number in the naming convention. This allows you to apply update sets in the order they were created. For example, ''PRB10005.01 - Duplicate Email Issues Fix'' would be applied before ''PRB10005.02 - Duplicate Email Issues Fix''.

== Provide Helpful Descriptions ==

Provide descriptions for update sets and [[Commenting Your Work|comment your work]]. Consider the following when updating the '''Description''' field:
*Function of the update set. For example, ''Basic SSO user authentication configuration''.
*Contents of the update set. For example, ''Includes SSO configuration and validation script action''.
*Post-installation instructions. For example, ''Navigate to System Properties > Single Sign On and configure three redirection properties''.
*Release, defect, or enhancement numbers, if applicable. For example, ''Defect: DFCT0010035 - Enable single sign-on''.

== Organize Update Sets  ==

For major releases, create update sets based on process, stream, or application. For example, assume that hypothetical Release 1 includes the Incident and Knowledge applications and is implemented in a number of sprints, or sub-releases. It is beneficial to create separate update sets for each area (incident and knowledge), organized by sprint. For example, R1_Incident_Sprint1, R1_Knowledge_Sprint2, and so on. This allows changes to be promoted and tested within the same release or sprint with minimal conflict. 

You can handle cross-process or process agnostic information, such as a new field in the User [sys_user] table, by creating a series of core or platform update sets that are separate from process-specific update sets. For example, R1_Core_Sprint1_121231_CT.

When addressing bug fixes or defects, the method you use may depend upon how often your organization releases fixes.  If you release fixes only sporadically, you can create an update set for each fix and promote each one as needed. If your organization releases fixes on a regular schedule, you can create an update set for each fix in a development instance and later merge the update sets for the next scheduled release to production.

== Merge Update Sets ==

You may want to [[Using Update Sets#Merging Update Sets|merge update sets]] to simplify migration. For example, rather than transferring 30 update sets from a development instance to a test instance, then previewing and committing each update set individually, you can consolidate the update sets into one merged update set. This reduces the number of times you have to preview and commit update sets. 

{{Note|Do not manually merge updates into an update set. Always use the '''Merge Update Sets''' module to compare duplicate files between update sets and select the newest version.}}

=== Clean Up Previously Merged Update Sets ===

Once you have merged one or more update sets to a single update set, consider removing the original update sets or changing the state to "Ignore" to prevent them from being retrieved by the target instance. The merge process moves all the updates from the source update sets to the target, leaving little or no content in your source update set(s). If you leave the original update sets in a state of "Complete" on your development instance for example, it may be confusing to administrators on the test system if they retrieve update sets that have previously been merged and have no valuable content.

= Update Set Validation  = 

Inspecting your update set before committing it can help prevent errors as changes are migrated to other instances. Before changing the state of an update set to '''Complete''', consider the following guidelines.

{{Warning|Never reopen completed update sets. After a completed update set is transferred to another instance, any future changes to the source update set are ignored.}}

== Verify Dictionary Updates Do Not Result in Data Loss ==
Update sets cannot [[System Dictionary#Altering System Dictionary Records|remove tables]] or [[System Dictionary#Modifying Dictionary Entries|change a column's data type]] if the change results in data loss. The target instance skips the update set and adds a log message if an incoming change would result in data loss. Perform these tasks manually on the target instance if they are crucial to the customization.

== Remove Debugging Code ==

Debugging statements are generally not needed in production instances. For example, the following statements are mainly used for debugging purposes and are not usually necessary in production:
*<tt>gs.log</tt>
*<tt>gs.logWarning</tt>
*<tt>gs.logError</tt> 

To check for unnecessary debugging code:

#Navigate to '''System Update Sets > Local Update Sets''' 
#Open an update set record.
#Create a filter on the '''Customer Updates''' related list with these following conditions:
#: Payload contains gs.log
#: OR 
#: Payload contains gs.logWarning
#: OR
#: Payload contains gs.logError 
#Review the records listed to ensure log statements are necessary.

== Ensure Updates Are Instance Independent ==

There may be data or settings in an update set that are instance-specific. For example, the SAML configuration on the ServiceNow development instance points to a different single sign-on identity provider than the production, test, and QA instances do. In such cases, [[Documenting_a_Migration_Procedure|post-migration instructions]] are required to ensure that target instances behave as expected. If the instance-specific data is part of the update set, it must be changed manually after the update set is transferred, previewed, and committed on the target instance; otherwise, the data may cause unexpected behaviors on the target instance.

== Remove Unnecessary Records ==

If you are creating unit tests, ensure the unit test scripts are not included in update sets. There is no need to perform unit tests outside a development instance.
