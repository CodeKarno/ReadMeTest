<!-- default file list -->
*Files to look at*:

* [Default.aspx](./CS/WebSite/Default.aspx) (VB: [Default.aspx](./VB/WebSite/Default.aspx))
* [Default.aspx.cs](./CS/WebSite/Default.aspx.cs) (VB: [Default.aspx.cs](./VB/WebSite/Default.aspx.cs))
<!-- default file list end -->
# BootstrapGridView  - Batch Edit - How to initialize ComboBox column on a row basis.


This example shows how to get a value from other cell from a row on the client side and bind the combo box in the same row on the server side based on this value.
This causes problems, because if you use the built-in ComboBox column, when you rebind it to a filtered data source, this data source applies to all rows, not just one.
To fix that, use BootstrapComboBox control in EditItemTemplate and initialize it on a row basis. A similar approach is demonstrated in [ASPxGridView - Batch editing - How to use the EditItemTemplate](https://www.devexpress.com/Support/Center/Question/Details/T618940/aspxgridview-batch-editing-how-to-use-edititemtemplate) example 

><b>NOTE:</b> The following code is written for BootstrapGridView, but you can also use it with an ASPxGridView, if you make the appropriate changes. The general approach does not change.

Since your grid will use column settings for the **View mode** and BootstrapComboBox settings for the **Edit mode**, they should be configured **separately**.

### Follow these steps: 

1. Add a BootstrapGridView control to your page and add the required columns to it.
2. To bind the column, handle its [ItemRequestedByValue][1] and [ItemRequestedByFilterCondition][2] events and filter its data source without using a value from another field. This binding is for BootstrapGridView's **View mode**:

```csharp
protected void OnItemRequestedByValue_View(object source, ListEditItemRequestedByValueEventArgs e) {
    if (e.Value == null) return;
    int subTypeID = (int)e.Value;
    BootstrapComboBox editor = source as BootstrapComboBox;
    editor.DataSource = ItemTypeFactory.GetItemSubTypes().Where(s => s.SubTypeID == subTypeID);
    editor.DataBind();
}
protected void OnItemsRequestedByFilterCondition_View(object source, ListEditItemsRequestedByFilterConditionEventArgs e) {
    BootstrapComboBox editor = source as BootstrapComboBox;
    editor.DataSource = ItemTypeFactory.GetItemSubTypes().Where(s => s.SubTypeDescription.Contains(e.Filter)
    || s.SubTypeName.Contains(e.Filter)
    || s.SubTypeID.ToString().Contains(e.Filter));
    editor.DataBind();
}
```

3. Also, configure the BootstrapGridViewComboBox column as follows:

```aspx
<dx:BootstrapGridViewComboBoxColumn FieldName="YourFieldName" ...>
    <PropertiesComboBox ValueField="YourFieldName"	OnItemRequestedByValue="OnItemRequestedByValue_View"
    OnItemsRequestedByFilterCondition="OnItemsRequestedByFilterCondition_View" TextFormatString="{0}|{1}...">
        <Fields>
            ...                            
        </Fields>
```
4. Add BootstrapComboBox control to the column's EditItemTemplate. This control will be used for the Edit mode. It has almost the same configuration with minor differences: it should have other data binding handlers, you need to set its ClientInstanceName for client-side access:

```aspx
<EditItemTemplate>
    <dx:BootstrapComboBox ID="bsCombobox" runat="server" EnableCallbackMode="true" ClientInstanceName="bsCombobox"
    OnItemRequestedByValue="OnItemRequestedByValue" OnItemsRequestedByFilterCondition="OnItemsRequestedByFilterCondition"
    ValueField="SubTypeID" TextFormatString="{0}|{1}...">
        <Fields>
            ...
        </Fields>
```
><b>NOTE:</b> Be sure to set the EnableCallbackMode parameter to true.

5. In EditItemTemplate, both [OnItemsRequestedByFilterCondition][2] and [OnItemRequestedByValue][1] handlers should use your other field value:
```csharp
protected void OnItemRequestedByValue(object source, ListEditItemRequestedByValueEventArgs e) {
    ...
    int typeID = GetCurrentItemTypeID();
    List<SubType> subTypes;
    if (typeID > -1)
        subTypes = ItemTypeFactory.GetItemSubTypes().Where(s => s.TypeID == typeID && s.SubTypeID.ToString() == id).ToList();
    ...
    editor.DataSource = subTypes;
    editor.DataBind();
}

protected void OnItemsRequestedByFilterCondition(object source, ListEditItemsRequestedByFilterConditionEventArgs e) {
    BootstrapComboBox editor = source as BootstrapComboBox;
    int typeID = GetCurrentItemTypeID();
    List<SubType> subTypes;
    if (typeID > -1)
        subTypes = ItemTypeFactory.GetItemSubTypes().Where(s => s.TypeID == typeID && (s.SubTypeDescription.Contains(e.Filter) ...
    editor.DataSource = subTypes;
    editor.DataBind();
}
```

6. Finally, the control should be initialized on the client side in the way demonstrated at [ASPxGridView - Batch Editing - A simple implementation of an EditItemTemplate](https://www.devexpress.com/Support/Center/p/T115096.aspx). The BootstrapComboBox value should be initialized with the cell value in the BatchEditStartEditing handler and the cell value and text should be set back in the BatchEditEndEditing handler:
```javascript
function OnBatchEditStartEdit(s, e) {
    var currentTypeID = grid.batchEditApi.GetCellValue(e.visibleIndex, 'TypeID');
    var cellInfo = e.rowValues[ColIndexByName('SubTypeID')];
    this.setTimeout(function () {
      bsCombobox.SetValue(cellInfo.value);
      bsCombobox.SetText(cellInfo.text);
    }, 0);
    RefreshData(currentTypeID);
}
function OnBatchEditEndEdit(s, e) {
    var cellInfo = e.rowValues[ColIndexByName('SubTypeID')];
    cellInfo.value = bsCombobox.GetValue();
    cellInfo.text = bsCombobox.GetText();
    bsCombobox.SetValue(null);
}
```
7. Also, to transfer the value of the field to the server, you need the ASPxHiddenField control and the RefreshData method:

```javascript
function RefreshData(currentTypeID) {
		hf1.Set('currentTypeID', currentTypeID);
		bsCombobox.PerformCallback();
	}
```
[1]: https://documentation.devexpress.com/AspNet/DevExpress.Web.ASPxComboBox.ItemRequestedByValue.event
[2]: https://documentation.devexpress.com/AspNet/DevExpress.Web.ASPxComboBox.ItemsRequestedByFilterCondition.event

