<!-- default file list -->
*Files to look at*:

* [Default.aspx](./CS/WebSite/Default.aspx) (VB: [Default.aspx](./VB/WebSite/Default.aspx))
* [Default.aspx.cs](./CS/WebSite/Default.aspx.cs) (VB: [Default.aspx.vb](./VB/WebSite/Default.aspx.vb))
* [BatchEditController.js](./CS/WebSite/BatchEditController.js) (VB: [BatchEditController.js](./VB/WebSite/BatchEditController.js))
<!-- default file list end -->
# ASPxGridView  - Batch Edit - How to use EditItemTemplate with DataItemTemplate.

This example shows how to use a DataItemTemplate and EditItemTemplate together using ASPxRatingControl as a template.

### Follow these steps: 

1. Add an ASPxGridView control to your page and configure it as follows:
```aspx
<dx:ASPxGridView ID="ASPxGridView1" runat="server" KeyFieldName="ID"
	ClientInstanceName="grid" Width="350" DataSourceID="ObjectDataSource1">	
	...
	<SettingsEditing Mode="Batch">
		<BatchEditSettings EditMode="Cell" AllowRegularDataItemTemplate="true" StartEditAction="FocusedCellClick" />
	</SettingsEditing>
	<ClientSideEvents BatchEditStartEditing="OnBatchEditStartEditing"
		BatchEditEndEditing="OnBatchEditEndEditing"
		FocusedCellChanging="OnFocusedCellChanging" Init="OnGridInit" />
</dx:ASPxGridView>
```
>Note the values of the [AllowRegularDataItemTemplate][1] and [StartEditAction][2] properties.

2. Add controls to the DataItemTemplate and EditItemTemplate tags. Notice that in the DataItemTemplate ClientInstanceName property of the control is dynamically assigned to each control. And for the EditItemTemplate control, this property is specified statically.

```aspx
 <DataItemTemplate>
	<dx:ASPxRatingControl ID="ratingControl" runat="server"
		ClientInstanceName='<%# "DataItemRateControl"+ Container.VisibleIndex %>' 
			ItemCount="5" Value='<%# Convert.ToInt32(Eval("RatingValue")) %>'>
			<ClientSideEvents ItemClick="OnItemMouseClick_DataItem"
				 ItemMouseOver="OnItemMouseOver_DataItem" ItemMouseOut="OnItemMouseOut_DataItem" />
	</dx:ASPxRatingControl>
</DataItemTemplate>
<EditItemTemplate>
	<dx:ASPxRatingControl ID="ratingControl" runat="server"
		ClientInstanceName="EditItemRateControl" ItemCount="5">
		<ClientSideEvents ItemClick="OnItemMouseClick_EditItem" Init="OnRateControlInit_EditItem" />
	</dx:ASPxRatingControl>
</EditItemTemplate>
```

3. Handle the client-side [BatchEditStartEditing][3] and [OnBatchEditEndEditing][4] events to set the ASPxRatingControl value inside the EditItemTemplate when editing starts, and also to save the selected value in the grid when editing ends.
```javascript
function OnBatchEditStartEditing(s, e) {
    EditItemRateControl.SetValue(e.rowValues[s.GetColumnByField("RatingValue").index].value);
}
function OnBatchEditEndEditing(s, e) {
    var templateColumn = s.GetColumnByField("RatingValue");
    if (!e.rowValues.hasOwnProperty(templateColumn.index))
        return;
    var cellInfo = e.rowValues[templateColumn.index];
    cellInfo.value = EditItemRateControl.GetValue();
    SetRateControlValueByRowIndex_DataItem(e.visibleIndex, cellInfo.value);
}
```
4.Handle the client-side ItemMouseClick event for controls inside the EditItemTemplate and DataItemTemplate separatly.
If you select the ASPxRatingControl value in the EditItemTemplate, the editing will be completed using the [batchEditApi.EndEdit][5] method. And if you select the ASPxRatingControl value in the DataItemTemplate, you should set the cell value using the [batchEditApi.SetCellValue][6] method.

```javascript
function OnItemMouseClick_EditItem(s, e) {
    grid.batchEditApi.EndEdit();
}
function OnItemMouseClick_DataItem(s, e) {
    grid.batchEditApi.SetCellValue(currentFocusedCell.itemVisibleIndex, currentFocusedCell.column.index, s.GetValue());
}
```
5. To add navigation to the grid, you need to handle the client-side Init event of the grid to subscribe to its keydown event.
```javascript
function OnGridInit(s, e) {
    ASPxClientUtils.AttachEventToElement(s.GetMainElement(), "keydown", function (evt) {
        return OnKeyDown(evt, s);
    });
}
function OnGridViewKeyDown(evt, grid) {
    if (typeof (event) != "undefined" && event != null)
        evt = event;
    if (!grid.InCallback() && NeedProcessDocumentKeyDown(evt)) {
        if (evt.shiftKey && evt.keyCode == 9 /*Shift + tab */) {
            setTimeout(function () {
                grid.batchEditApi.MoveFocusBackward();
            }, 0);
        } else if (evt.keyCode == 9 /*Tab key*/) {
            setTimeout(function () {
                grid.batchEditApi.MoveFocusForward();
            }, 0);
        }  
    }
}
```
6. Also, to add navigation to the ASPxRatingControl, you need to handle the client-side Init event of the ASPxRatingControl to subscribe to its keydown event.
```javascript
function OnRateControlInit_EditItem(s, e) {
    ASPxClientUtils.AttachEventToElement(s.GetMainElement(), "keydown", function (evt) {
        return OnRatingControlKeyDown(evt, s);
    });
}
function OnRatingControlKeyDown(evt, ratingControl) {
    if (typeof (event) != "undefined" && event != null)
        evt = event;
    if (!ratingControl.InCallback() && NeedProcessDocumentKeyDown(evt)) {
        if (evt.keyCode == 32 /*Space bar*/) {
            setTimeout(function () {
                MoveFocusToNextStar();
            }, 0);
        }
    }
}
```


[1]: https://documentation.devexpress.com/AspNet/DevExpress.Web.GridBatchEditSettings.AllowRegularDataItemTemplate.property
[2]: https://documentation.devexpress.com/AspNet/DevExpress.Web.GridBatchEditSettings.StartEditAction.property
[3]: https://docs.devexpress.com/AspNet/js-ASPxClientGridView.BatchEditStartEditing
[4]: https://docs.devexpress.com/AspNet/js-ASPxClientGridView.BatchEditEndEditing
[5]: https://docs.devexpress.com/AspNet/js-ASPxClientGridViewBatchEditApi.EndEdit
[6]: https://docs.devexpress.com/AspNet/js-ASPxClientGridViewBatchEditApi.SetCellValue(visibleIndex-columnFieldNameOrId-value)
