Source code involves: 
https://github.com/SAP/openui5/blob/master/src/sap.m/src/sap/m/routing/Router.js 
https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/UIComponent.js https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/routing/Router.js

Here is a discussion about Router and Target: https://stackoverflow.com/questions/46336762/what-is-the-difference-between-sap-ui-core-routing-router-navto-and-sap-m-rout

The reason I dig into the source code is I want to understand why `View` can change when I call `Router.navTo()`.

Take this offical demo as an example: 
https://sapui5.hana.ondemand.com/#/sample/sap.ui.core.sample.RoutingFullscreen/preview

From the source code of `navTo`, it seems has nothing to do with View: 
https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/routing/Router.js#L514 
```js
// third party hasher.setHash() is called in sap.ui.core.routing.HashChanger
HashChanger.prototype.setHash = function(sHash) {
	this.fireEvent("hashSet", { sHash : sHash });
	hasher.setHash(sHash);
};
```

So I think the event might be registered in `Component.init()`

# What `UIComponent.prototype.init` do for Route? 

## manifest.json is defined in Demo

## Component.js: 
call `UIComponent.prototype.init`

```js
init : function () {
  UIComponent.prototype.init.apply(this, arguments);
  this.getRouter().initialize();
}
```

## sap.ui.core.UIComponent.js

In function `UIConfig.prototype.init`, router constructor is called: 
https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/UIComponent.js#L257

`var oRoutingManifestEntry = this._getManifestEntry("/sap.ui5/routing", true) || {}`
returns config in manifest.json. (metadata.routing.routes)

`vRoutes = oRoutingManifestEntry.routes;`
vRoutes is used for storing `routes` config

[function getConstructorFunctionFor ](https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/UIComponent.js#L307) using `jQuery.sap.getObject` to get constructor of `sap.m.routing.Router`

And then the constructor is called:
`this._oRouter = new fnRouterConstructor(vRoutes, oRoutingConfig, this, oRoutingManifestEntry.targets);`

The four arguments stands for: 

* vRoutes: metadata.routing.routes
* oRoutingConfig: metadata.routing.config
* this: UIComponent itself，for view creation
* oRoutingManifestEntry.targets: metadata.routing.targets

The constructor call make the process jump to `sap.m.routing.Router` :

```js
var MobileRouter = Router.extend("sap.m.routing.Router",{
	constructor : function() {
		this._oTargetHandler = new TargetHandler();

		Router.prototype.constructor.apply(this, arguments);
	}
}
```

As we know, sap.m.routing.Router(Mobile Router) extend `sap.ui.core.routing.Router`:
The four arguments corresponding to the previous four.

`this._oRouter = crossroads.create();`
a new independent Router instance is created by the third party routing library [crossroads](
https://millermedeiros.github.io/crossroads.js/#crossroads-create), 

```js
this._oViews = new Views({
  component : oOwner,
  async : this._oConfig._async
});
```
Views that are using in the routes are created, which can be using in `Router.getView()` and `Router._createTargets` etc.:

```js
getViews : function () {
	return this._oViews;
},

//_createTargets is called in constructor if metadata.routing.targets exists
_createTargets : function (oConfig, oTargetsConfig) {
	return new Targets({
		views: this._oViews,
		config: oConfig,
		targets: oTargetsConfig
	});
},
```

if target config exist, creates targets(call `Targets` constructor) and stores them in `this._oTargets`, `this._oTargets` will be used in `Router.initialize()`

Call Stack: 
[sap.m.rounting.Targets.constructor](https://github.com/SAP/openui5/blob/master/src/sap.m/src/sap/m/routing/Target.js#L39) => [sap.ui.core.routing.Targets.constructor](https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/routing/Targets.js#L293) => [sap.ui.core.routing.Targets._createTarget()](https://github.com/SAP/openui5/blob/master/src/sap.ui.core/src/sap/ui/core/routing/Targets.js#L523)

Finally, I think I found the view display register function: 
```js
// _createTarget()
oTarget.attachDisplay(function (oEvent) {
	var oParameters = oEvent.getParameters();

        //break point here
	this.fireDisplay({
		name : sName,
		view : oParameters.view,
		control : oParameters.control,
		config : oParameters.config,
		data: oParameters.data
	});
}, this);
```

Set a breakpoint in `this.fireDisplay`, and you can see the call stack.

The `hasher.setHash()` called in `navTo` fires `hashChanged` event and `_routeMatched` event, and then the `_display` event is fired. And then View is displayed.

