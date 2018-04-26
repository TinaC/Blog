Since I did not get an answer in my question: https://stackoverflow.com/questions/49975762/how-is-data-binding-implemented-in-sapui5 
I decided to write it by my own. 

Some comments said UI5 use Handlebars for data binding, and after search, Handlebars only support for [one-time data binding](http://ahadb.com/2017/09/21/binding/). What I am more curious is how two-way data binding implemented in UI5.
~~IMO, the difference between data binding and template is~~: 
In Handlebars Once you compile your template, the view/DOM has nothing to do with the data model.
But two-way data binding connects data to a property or attribute of an element in its local DOM.

And two-way binding means:
> When properties in the model get updated, so does the UI.
> When UI elements get updated, the changes get propagated back to the model.
https://stackoverflow.com/a/13504965/5238583 

In the question of [How to Implement DOM Data Binding in JavaScript
](https://stackoverflow.com/questions/16483560/how-to-implement-dom-data-binding-in-javascript), many techniques are mentioned. UI5 uses these two(what I've found so far): [add change event listener](https://stackoverflow.com/a/16484266/5238583) and [mutators(setter)](https://stackoverflow.com/a/16485030/5238583)

I use this official sample for example: [Data Binding - Step 13 - Element Binding](https://sapui5.hana.ondemand.com/#/sample/sap.ui.core.tutorial.databinding.13/preview)

data binding changes when: 
`oProductDetailPanel.bindElement({ path: sPath, model: "products" });`

Set a break point in [oBinding.setContext() in ManagedObject.prototype.updateBindingContext](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/base/ManagedObject.js#L3972) and [ManagedObject.prototype.updateProperty](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/base/ManagedObject.js#L3197).

1. `Element.prototype.bindElement` equals to [ManagedObject.prototype.bindObject](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/base/ManagedObject.js#L2711)

2. [oBinding.initialize()](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/base/ManagedObject.js#L2784) which means [ClientContextBinding.prototype.initialize](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/model/ClientContextBinding.js#L63) is called in `ManagedObject.prototype._bindObject`

3. [Binding.prototype._fireChange](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/model/Binding.js#L269) is called in the `createBindingContext` callback. Which fire `change` event: `this.fireEvent("change", mArguments)`;

4. And! The change event handler is defined in `ManagedObject.prototype._bindObject` : 

    var fChangeHandler = function(oEvent) {
        that.setElementBindingContext(oBinding.getBoundContext(), sModelName);
    };
    oBinding.attachChange(fChangeHandler);
    oBindingInfo.modelChangeHandler = fChangeHandler;

5. `setElementBindingContext()` calls `ManagedObject.prototype.updateBindingContext` eventually
6.  In `updateBindingContext`, the call stack is `oBinding.setContext(oContext)` -> `JSONPropertyBinding.prototype.checkUpdate`(because the sample use JSON Model here) -> `this._fireChange({reason: ChangeReason.Change})`
7.  For the second change event, the handler is in [ManagedObject.prototype._bindProperty](https://github.com/SAP/openui5/blob/c728dbc7a5393975e0f2e71a5d4627fb4f7bef13/src/sap.ui.core/src/sap/ui/base/ManagedObject.js#L3036) (There are many fModelChangeHandler in bind functions of ManagedObject, For our bindElement sample, we only need this one)

8. In the fModelChangeHandler, `ManagedObject.prototype.updateProperty` is called. That where our setter is used: 

> whenever a property binding is changed.This method gets the external format from the property binding and applies it to the setter.

`this[oPropertyInfo._sMutator](oValue);`. For our sample `oPropertyInfo._sMutator` is `setValue`. excute this, the value in Input (`<Input value="{products>ProductID}"/>`) will be changed. 

