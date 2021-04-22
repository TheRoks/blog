---
title: "More about SharePoint eventhandling"
date: "2009-01-30"
categories: 
  - "code"
  - "eventhandlers"
  - "sharepoint"
---

As I had problems with the way events worked, I wanted to know more about the way they worked. below is bit of information about events that is good to know:  
There are two different kinds of events: Synchronous and Asynchronous events.

- Synchronous events
    - End with -ing (ItemAdding, ItemUpdating)
    - Block the code as long the events arent finished
    - Can (thus) make use of Session vars (HttpContext.Current is available), because they run in the same session
    - Can be used to prevent the event being finished. For example, with the ItemUpdating event you can verify the content being updated. Is the update not valid, the SPItemEventProperties.Cancel can be set to true and the SPItemEventProperties.ErrorMessage can be filled with an error message. This error message _will not_ be displayed, but is written to the logfile. An error page will appear.
    - dont contain the SPListItem in it's SPItemEventProperties so when you want to make changes to any of it's field you need to make use of the SPItemEventProperties.AfterProperties (see message below). In the case of itemUpdating, the "old" SPListItem will be used
- Asynchronous events
    - End with -ed (ItemAdded, ItemUpdated)
    - Dont block the code.
    - Can't (thus) make use of the Session (HttpContext.Current is null, because code is run in another context)
    - Can't be reverted; When these events are fired, the item is already added, updated or deleted.
    - contains the SPListItem. Changes can be made if you want to. This fires the (for example) ItemUpdating and ItemUpdated event. This can be turned off by using this.DisableEventFiring();
    - When creating a new list, events like FieldAdded and FieldAdding are _not_ fired!
