<!-- default file list -->
*Files to look at*:

* [Default.aspx](./CS/WebSite/Default.aspx) (VB: [Default.aspx](./VB/WebSite/Default.aspx))
* [MovingItems.js](./CS/WebSite/Scripts/MovingItems.js) (VB: [MovingItems.js](./VB/WebSite/Scripts/MovingItems.js))
<!-- default file list end -->
# Moving items between two list boxes (client-side approach)


<p>This example shows how to move items from the first ASPxListBox to the second ASPxListBox, while maintaining the original order of the items.</p>

><b>NOTE:</b> This code will work with any ASPxListBox selection mode. (Single / Multiple / CheckColumn)

### Follow these steps: 

1. Add 2 ASPxListBox controls and 4 ASPxButton controls to your page.
2. Create an array in which the initial order of elements will be written. Then, create a method to populate the array.

```javascript
var primaryListBoxOptions = [];
function PushOptions() {
    for (var i = 0; i < lbAvailable.GetItemCount(); i++) {
        primaryListBoxOptions.push(lbAvailable.GetItem(i).value);
    }
}
```

3. Handle the OnInit client event at the first ListBox:

```javascript
function OnLBAvailableInit() {
    UpdateButtons();
    PushOptions();
}
```
4. Create a method that will move elements between ListBox controls:

```javascript
function MoveItems(lb1, lb2, items) {
    if (!items) return;
    for (var i = 0; i < items.length; i++) {
        lb2.InsertItem(GetCurrentIndex(items[i].value, lb2), items[i].text, items[i].value);
        lb1.RemoveItem(GetItemIndexByValue(items[i].value, lb1));
    }
}
```
5. Create methods that will determine the future position of an element in the ListBox, depending on its initial position:
```javascript
function GetPrimaryIndex(value) {
    var options = GetPrimaryOptions();
    for (var i = 0; i < options.length; i++)
        if (options[i] == value)
            return i;
}

function GetCurrentIndex(value, lbDestination) {
    var options = GetPrimaryOptions();
    for (var i = (GetPrimaryIndex(value) - 1); i >= 0; i--) {
        var neighborIndex = GetItemIndexByValue(options[i], lbDestination);
        if (neighborIndex != -1)
            return neighborIndex + 1;
    }
    return 0;
}

function GetPrimaryOptions() {
    if (primaryListBoxOptions.length == 0)
        PushOptions();
    return primaryListBoxOptions;
}
```

6. Create methods to move the selected items and all of the items:
```javascript
function MoveSelectedItems(lb1, lb2) {
    MoveItems(lb1, lb2, lb1.GetSelectedItems());
    UpdateButtons();
}

function MoveAllItems(lb1, lb2) {
    MoveItems(lb1, lb2, GetAllItems(lb1));
    UpdateButtons();
}

function GetAllItems(lb) {
    var items = [];
    for (var i = 0; i < lb.GetItemCount(); i++) {
        items.push(lb.GetItem(i));
    }
    return items;
}
```
7. Handle the OnClick client event on all buttons and call the desired methods for moving elements:

```javascript
function AddSelectedItems() {
    MoveSelectedItems(lbAvailable, lbChoosen);
}
function AddAllItems() {
    MoveAllItems(lbAvailable, lbChoosen);
}
function RemoveSelectedItems() {
    MoveSelectedItems(lbChoosen, lbAvailable);
}
function RemoveAllItems() {
    MoveAllItems(lbChoosen, lbAvailable);
}
```


<p><strong>See Also:<strong><br />
</strong><a href="https://www.devexpress.com/Support/Center/p/E3108">Moving items between two ASPxListBoxes (server-side approach)</a></p>

<br/>

