<a href="https://githubsfdeploy.herokuapp.com?owner=SidSays&repo=apex-trigger-handler-helper-framework&ref=main">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png">
</a>

# Simple, Scalable & Maintainable apex framework

The last thing you want to hear is "an apex framework", please bear with me or watch your favorite show instead !

> This is not an original concept neither I claim it to be. Feel free to add/modify to your own needs and relate/refer to any other sources.

## Description
In my 15 years of implementing salesforce for highly complex multi orgs to simple orgs, this framework has helped me most in terms of time saving, ease to educate developers, keeping regression impact minimum yet empowering future scalability and ease of deployments.

> For me, the pros far outweigh the cons.

## Why?
- You lead a team of mixed bag of differntly skilled developers, admins and everyone came from some other technology
- The team changes due to attrition or vendor changes and hence its difficult to follow documentation and standards they left behind
- Everyone has been following some framework which isn't fully understood by every architect and developer
- You need something simple, efficient, scalable where new functionlity is always kept separted to avoid regression impacts
- You know exactly to locate where a problem is arising, easy to identify a small helper class instead of a HUGE MAMMOTH Handler class
- Your org has multiple business units with different release dates and different admins/developers
- Your deployments can be short and efficient because they're deploying specific helper classes and their test classes

## How?
- A switch (Custom Metadata) to turn the trigger for an object ON or OFF, e.g. data migrations
- One trigger per object
- Trigger calls a Handler Class (Which implements an interface)
- Handler class deals with the events (Before, After) and invokes individual Helper classes for each Business Unit
- Handler class also contains a common Helper class for each Business Unit

### Step 1: Create a custom metadata (switch) that enables turning a trigger ON/OFF for every object
- Trigger_Switch__mdt	
- The Is_trigger_On__c custom checkbox field
  
<img width="1164" alt="image" src="https://github.com/SidSays/apex-trigger-handler-helper-framework/assets/25760138/97bad0e2-eb9a-4d8e-9cf0-38def9205a7a">

- Create records for the trigger on every object

<img width="992" alt="image" src="https://github.com/SidSays/apex-trigger-handler-helper-framework/assets/25760138/b3e6a4b5-c8af-4513-8c3c-e12819f8c9a0">


### Step 2: Create a TriggerHandler Interface (i.e. Apex Class)

> NOTE: Only one is needed in every org. (Not per object). This is because we will have multiple Handlers in the org for each SObject. All those Handlers will refer to this interface.

```
public interface TriggerHandler {
    void beforeInsert(List<SObject> newRecords);
    void beforeUpdate(List<SObject> oldRecords, List<SObject> newRecords, Map<ID, Case> oldRecordMap, Map<ID, Case> newRecordMap);
    void beforeDelete(List<SObject> oldRecords, Map<ID, Case> oldRecordMap);
    void afterInsert(List<SObject> newRecords, Map<ID, Case> newRecordMap);
    void afterUpdate(List<SObject> oldRecords, List<SObject> newRecords, Map<ID, Case> oldRecordMap, Map<ID, Case> newRecordMap);
    void afterDelete(List<SObject> oldRecords, Map<ID, Case> oldRecordMap);
    void afterUndelete(List<SObject> newRecords, Map<ID, Case> newRecordMap);
}
```

### Step 2: Create a Trigger Handler Class for each SObject (Implements the interface, example below is for Case)

> NOTE: Observe that Is_trigger_On__c from custom metadata is getting evaluated and only proceeds if true
```
public without sharing class CaseTriggerHandler implements TriggerHandler {
    private boolean triggerIsExecuting;
    private integer triggerSize;
    public CaseTriggerHelper helper;
    public CaseTriggerHandler(boolean triggerIsExecuting, integer triggerSize) {
        this.triggerIsExecuting = triggerIsExecuting;
        this.triggerSize = triggerSize;
        this.helper = new CaseTriggerHelper();
    }
    public void beforeInsert(List<Case> newCases) {
    /* BUSINESS UNIT 1 */
    //commonHelper_Case.method(newCases);
    //specificHelper_Case.method(newCases);
    /* BUSINESS UNIT 2 */
    //commonHelper_Case.method(newCases);
    //specificHelper_Case.method(newCases);
    /* BUSINESS UNIT 3 */
    //commonHelper_Case.method(newCases);
    //specificHelper_Case.method(newCases);
    }
    public void beforeUpdate(List<Case> oldCases, List<Case> newCases, Map<ID,Case> oldCaseMap, Map<ID, Case> newCaseMap) {
    /* BUSINESS UNIT 1 */
    /* BUSINESS UNIT 2 */    
    /* BUSINESS UNIT 3 */ 
    }
    public void beforeDelete(List<Case> oldCases, Map<ID, Case> oldCaseMap) {
        // helper.doTask5();
        // helper.doTask1();
    }
    public void afterInsert(List<Case> newCases, Map<ID, Case> newCaseMap) {
     /* FOPS */
    CaseTrigger_Handler.communityAssignCaseWithActiveRule(newCaseMap); 
    CaseTrigger_Handler.setEmployeeAPIFieldsOnAfterInsert(newCases);
        
     /* IFS */
        
        
     /* OBS */ 
        OBS_Case_Duplicate_Helper.duplicatesOBS(newCases);
    }
    public void afterUpdate(List<Case> oldCases, List<Case> newCases, Map<ID, Case> oldCaseMap, Map<ID, Case> newCaseMap) {
    CaseTrigger_Handler.trackFeedCloseDescription(newCases,oldCaseMap);
    CaseTrigger_Handler.chatterPostForParentCase(newCases);
    }
    public void afterDelete(List<Case> oldCases, Map<ID, Case> oldCaseMap) {
        // helper.doTask3();
        // helper.doTask1();
    }
    public void afterUndelete(List<Case> newCases, Map<ID, Case> newCaseMap) {
        // helper.doTask4();
        // helper.doTask2();
    }
}
```

### Step 1: Trigger that invokes the Handler

> NOTE: Only one is needed in every org. (Not per object)

```
trigger CaseTrigger on Case (before insert, before update, before delete, after insert, after update, after delete, after undelete) {
    if (Triggers_Switch__mdt.getInstance('CaseTrigger')?.Is_trigger_On__c == true) {
        TriggerHandler handler = new CaseTriggerHandler(Trigger.isExecuting, Trigger.size);
        switch on Trigger.operationType {
            when BEFORE_INSERT {
                 handler.beforeInsert(Trigger.new);
            } 
            when BEFORE_UPDATE {
                 handler.beforeUpdate(Trigger.old, Trigger.new, Trigger.oldMap, Trigger.newMap);
            }
            /*when BEFORE_DELETE {
                 handler.beforeDelete(Trigger.old, Trigger.oldMap);
            }*/
            when AFTER_INSERT {
                 handler.afterInsert(Trigger.new, Trigger.newMap);
            }
            when AFTER_UPDATE {
                 handler.afterUpdate(Trigger.old, Trigger.new, Trigger.oldMap, Trigger.newMap);
            }
            /*when AFTER_DELETE {
                 handler.afterDelete(Trigger.old, Trigger.oldMap);
            } 
            when AFTER_UNDELETE {
                 handler.afterUndelete(Trigger.new, Trigger.newMap);
            }*/
        }
    }
}
```

