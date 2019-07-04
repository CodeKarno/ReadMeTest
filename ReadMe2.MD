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
	OnBatchUpdate="ASPxGridView1_BatchUpdate" ClientInstanceName="grid"
	Width="350" DataSourceID="ObjectDataSource1">
	<Columns>
		...
		<dx:GridViewDataColumn FieldName="RatingValue" VisibleIndex="2">
		<DataItemTemplate>...</DataItemTemplate>
		<EditItemTemplate>...</EditItemTemplate>
		</dx:GridViewDataColumn>
		...
	</Columns>
	<SettingsEditing Mode="Batch">
		<BatchEditSettings EditMode="Cell" AllowRegularDataItemTemplate="true" StartEditAction="FocusedCellClick" />
	</SettingsEditing>
	<ClientSideEvents BatchEditStartEditing="OnBatchEditStartEditing"
		BatchEditEndEditing="OnBatchEditEndEditing"
		FocusedCellChanging="OnFocusedCellChanging" Init="OnGridInit" />
</dx:ASPxGridView>
```
>Note the values of the [AllowRegularDataItemTemplate][1] and [StartEditAction][2] properties.

2. Add controls to the DataItemTemplate and EditDataTemplate tags. Notice that in the DataItemTemplate ClientInstanceName property of the control is dynamically assigned to each control. And for the EditDataTemplate control, this property is specified statically.

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
4.Handle the client-side ItemMouseClick event for the EditItemTemplate DataItemTemplate separately.
If you select the ASPxRatingControl value in the EditItemTemplate, the editing will be completed using the [batchEditApi.EndEdit][5] method. And if you select the ASPxRatingControl value in the DataItemTemplate, you must set the cell value using the [batchEditApi.SetCellValue][6] method.

```javascript
function OnItemMouseClick_EditItem(s, e) {
    grid.batchEditApi.EndEdit();
}
function OnItemMouseClick_DataItem(s, e) {
    grid.batchEditApi.SetCellValue(currentFocusedCell.itemVisibleIndex, currentFocusedCell.column.index, s.GetValue());
}
```

5. Also, you need to handle the server-side BatchUpdate event of the grid  and update the value ​​in the data source.
```csharp
 protected void ASPxGridView1_BatchUpdate(object sender, DevExpress.Web.Data.ASPxDataBatchUpdateEventArgs e) {
        foreach (var args in e.UpdateValues)
            UpdateItem(args.Keys, args.NewValues);
    }
    protected void UpdateItem(System.Collections.Specialized.OrderedDictionary keys, OrderedDictionary newValues) {
        var id = Convert.ToInt32(keys["ID"]);
        Product product = new Product { ID = id, RatingValue = Convert.ToInt32(newValues["RatingValue"]) };
        DataModel.UpdateProducts(product);
    }
```

6. To add navigation to the grid, you need to handle the client-side Init events of the grid and ASPxRatingControl (EditItemTemplate) to subscribe to the keydown events of these controls.
```javascript
function OnGridInit(s, e) {
    ASPxClientUtils.AttachEventToElement(s.GetMainElement(), "keydown", function (evt) {
        return OnKeyDown(evt, s);
    });
}
function OnRateControlInit_EditItem(s, e) {
    ASPxClientUtils.AttachEventToElement(s.GetMainElement(), "keydown", function (evt) {
        return OnRatingControlKeyDown(evt, s);
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
