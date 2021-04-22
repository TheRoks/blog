---
title: "Modify SPListItem fields with vars from the Session in the itemAdding or itemAdded events"
date: "2009-01-30"
categories: 
  - "code"
  - "sharepoint"
---

Some time ago I tried to alter a field of a SPListItem right after the moment that the item has been created. A var was stored in the Session so it could be used on the newly created SPListItem (itemAdded).

NOT!

I figured out that the HttpContext isn't available when during the ItemAdded event. When using the itemAdding event, the HttpContext is available, but the item that will be created, isnt. The SPFeatureReceiverProperty.SPListItem is null, because the item is not yet created.  After reading the MSDN documentation I figured out that the SPFeatureReceiverProperties Afterproperties can be used to store a value, which will be applied right after the item is created.

below is an example:

using System;

public class ItemEventReceiver

{

public ItemEventReceiver()

{

hContext = HttpContext.Current;

}

/// <summary>

/// overrides default ItemAdded

/// </summary>

/// <param name="properties"></param>

public override void ItemAdded(SPItemEventProperties properties)

{

Debug.WriteLine("ItemAdded");

CheckHContextAndSPListItem(properties.ListItem);

base.ItemAdded(properties);
}

/// <summary>

/// overrides the standard ItemAdding

/// </summary>

/// <param name="properties"></param>

public override void ItemAdding(SPItemEventProperties properties)

{

string internalFieldName = null;

string FIELD\_NAME = "Foo";

SPFieldLookupValue lookupField = null;

Debug.WriteLine("ItemAdding");

CheckHContextAndSPListItem(properties.ListItem);

using (SPWeb web = properties.OpenWeb())

{

internalFieldName = web.Lists\[properties.ListId\].Fields\[FIELD\_NAME\].InternalName;
}

// example for adding a value to a SPListItem property

string tempValue = "foo";

properties.AfterProperties\[internalFieldName\] = tempValue;

}

/// <summary>

/// checks the values of hContext and SPListItem

/// </summary>

/// <param name="SPListItem">SPListItem out of the SPItemEventProperties</param>

private static void CheckHContextAndSPListItem(SPListItemlistItem)

{

if (HttpContext.Current == null)

Debug.WriteLine("hContext is null");

else

Debug.WriteLine("hContext is available");

if (listItem == null)

Debug.WriteLine("listItem is null");

else

Debug.WriteLine("listItem is available");

}

}
