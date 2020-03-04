# Getting Started with PI Vision Extensibility

This document creates a simple PI Vision extensibility symbol. The symbol accepts a single tag or attribute and displays its value, label, timestamp, and units. The interesting thing about this symbol is that it changes its opacity based on the current value. The higher the value, the higher the opacity. In this way, a Process Engineer will only see data that is becoming an issue. Think of it as a multi-stated Value symbol that changes opacity as opposed to color. The goal of this document is to provide the user with a fully functional extensibility symbol framework that can be easily modified to create new symbols which meet the user&#39;s business requirements.

## Create the directory folders
### Ext Folder

Create the <code>ext</code> folder under:

```
PIVISION\_INSTALLATION\_FOLDER\Scripts\app\editor\symbols\_ext_
```

### Icons Folder

Create the <code>icons</code> folder under:
```
PIVISION\_INSTALLATION\_FOLDER\Scripts\app\editor\symbols\_ext\icons_
```

## Create the symbol files
### Presentation layer

The Presentation layer controls what the user sees in PI Vision. The Presentation layer for a custom symbol is basic HTML with AngularJS for data and configuration binding.

Create the presentation HTML file for the symbol in the ext folder and name it sym-_symbolPrimer_-template.html. Note that the name is important. PI Vision automatically looks for customer symbols based on this naming convention.

Add the following content to the file:

```html
<div class="SymbolPrimerLayout" ng-style="{opacity: config.Opacity, background: config.BackgroundColor, color: MultistateColor || config.TextColor}">
    <div ng-show="config.ShowLabel">{{label}}</div>
	<div>{{value}} <span ng-show="config.ShowUnits"> {{Units}}</span> </div>     
	<div ng-show="config.ShowTime">{{time}}</div>
</div>
```

### Configuration layer

The Configuration layer, much like the presentation layer, is basic HTML with AngularJS for data binding and provides a way to configure the symbol&#39;s appearance.

Create the configuration HTML file for the symbol in the _ext_ folder and name it sym-_symbolPrimer_-config.html.

Add the following content to the file:

```html
<div class="ConfigLayout">
    <label class="ConfigCheckBox">
        <input type="checkbox" name="ShowLabel" ng-model="config.ShowLabel">Show Label<br />
    </label>
    <label class="ConfigCheckBox">
        <input type="checkbox" name="ShowTime" ng-model="config.ShowTime">Show Timestamp<br />
    </label>
    <label class="ConfigCheckBox">
        <input type="checkbox" name="ShowUnits" ng-model="config.ShowUnits">Show Units<br />
    </label>
    <label class="ConfigText">
        <input class="ConfigTextBox" type="text" name="MinValue" ng-model="config.MinValue">Minimum Value<br />
    </label>
    <label class="ConfigText">
        <input class="ConfigTextBox" type="text" name="MaxValue" ng-model="config.MaxValue">Maximum Value<br />
	</label>
</div>
```

## Custom Style Sheet

The Custom Style Sheet (CSS) provides styling for the Presentation and Configuration layers.

Create the CSS file for the symbol in the _ext_ folder and name it sym-_symbolPrimer_.css.

Add the following content to the file:

```CSS
.ConfigLayout {
    padding: 15px;
    font-size: 14px
}
.ConfigCheckBox {
    display: flex;
    align-items: center;
    margin-top: 8px;
    margin-left: 15px;
    font-size: 14px;
}

.ConfigText {
    display: flex;
    align-items: center;
    margin-top: 8px;
    margin-left: 15px;
    font-size: 14px;
}

.ConfigTextBox {
    width: 50px;
}
```

### Symbol Icon

You can provide an icon to be added to the PI Vision symbol selector which appears in the Assets pane. A default symbol icon is used if this is not available. Use your preferred image editor to create a 512 x 512 pixel image with a transparent background. Save the image as sym-_symbolPrimer_.png. Copy the saved image into the

```
PIVISION\_INSTALLATION\_FOLDER\Scripts\app\editor\symbols\_ext\Icons_ folder.
```

### Implementation Layer

The JavaScript implementation file has four parts: definition, registration, initialization, and event handlers.

Create a file to contain the symbol&#39;s implementation in the _ext_ folder and name it sym-_symbolPrimer_.js.

Add the following to the file, note that the file contents will be explained below:

```javascript
(function(PV) {
    'use strict';

    function symbolPrimer() {}
    PV.deriveVisualizationFromBase(symbolPrimer);

    var definition = {
        typeName: 'symbolPrimer',
        datasourceBehavior: PV.Extensibility.Enums.DatasourceBehaviors.Single,
        iconUrl: 'scripts/app/editor/symbols/ext/icons/sym-symbolPrimer.png',
        supportsCollections: true,
        visObjectType: symbolPrimer,
        getDefaultConfig: function() {
            return {
                DataShape: 'Value',
                Height: 70,
                Width: 250,
                Opacity: "1.0",
                BackgroundColor: 'grey',
                TextColor: 'white',
                ShowLabel: true,
                ShowTime: false,
                ShowUnits: false,
                MinValue: 0,
                MaxValue: 100
            };
        },
        configTitle: 'Format Symbol Primer',
        StateVariables: ['MultistateColor']
    };

    symbolPrimer.prototype.init = function(scope, elem) {
        this.onDataUpdate = dataUpdate;
        this.onResize = symbolResize;
        this.onConfigChange = configChange;


        function dataUpdate(data) {
            if (data) {
                scope.value = data.Value;
                scope.config.Opacity = (data.Value - scope.config.MinValue) / (scope.config.MaxValue - scope.config.MinValue);
                scope.time = data.Time;
                if (data.Label) {
                    scope.label = data.Label;
                }
                if (data.Units) {
                    scope.Units = data.Units;
                }
            }
        }

        function symbolResize(width, height) {
            var SymbolContainer = elem.find('.SymbolPrimerLayout')[0];
            if (SymbolContainer) {
                SymbolContainer.style.width = width + 'px';
                SymbolContainer.style.height = height + 'px';
            }
        }

        function configChange(newConfig, oldConfig) {
            if (newConfig && oldConfig && !angular.equals(newConfig, oldConfig)) {
                if (!isNumeric(newConfig.MinValue) || !isNumeric(newConfig.MaxValue) || parseFloat(newConfig.MinValue) >= parseFloat(newConfig.MaxValue)) {
                    newConfig.MinValue = oldConfig.MinValue;
                    newConfig.MaxValue = oldConfig.MaxValue;
                }
            }
        }

        function isNumeric(n) {
            return n === '' || n === '-' || !isNaN(parseFloat(n)) && isFinite(n);
        }
    };

    PV.symbolCatalog.register(definition);
})(window.PIVisualization);
```

### A look at the code

#### Symbol wrapper

The Implementation layer is wrapped in an immediately invoked function expression (IIFE). This function excepts a global PI Visualization object, which is passed in as a parameter.

```javascript
(function(PV) {
    'use strict';

    function symbolPrimer() {}
    PV.deriveVisualizationFromBase(symbolPrimer);
})(window.PIVisualization);
```

#### Definition object

The definition property, shown below, is a JSON object (key-value pairs) that sets defaults for the symbol.

<ul>
<li><code>typeName</code>: This is the name of the symbol and will appear as a tooltip when the mouse is hovered over the symbol&#39;s icon. </li>
<li><code>datasourceBehavior</code>: Setting this property to Single allows you to drag and drop a single data item onto a display to create the symbol.</li>
<li><code>getDefaultConfig</code>: This property initializes the symbols default configuration. Updates to this property made by the configuration layer will be saved to the backend database. When a saved display is reopened it will be initialized with the saved configuration. Changes to this property should only be made by the configuration layer.</li>
<li><code>DataShape</code>: This parameter tells the application server the information that this symbol needs to represent the data.</li>
<ul>
  <li><code>Value</code>: Single value at a specific time</li>
  <li><code>Gauge</code>: Includes the ratio of a value between a minimum and a maximum</li>
  <li><code>Trend</code>: Multiple data source shape</li>
  <li><code>Table</code>:  Multiple data source shape, allows you to specify columns and sorting</li>
  </ul>
<li><code>configTitle</code>: This is the text for the configuration menu option that appears in the symbol&#39;s context (right-click) menu.</li>
<li><code>StateVariables</code>:  Setting this to [&#39;MultistateColor&#39;] enables multi-state source configuration.</li>
<li><code>supportsCollectons</code>: 
Indicates whether the symbol can be included as part of a collection symbol.</li>
<li><code>visObjectType</code>: The name of the function that was extended from PV.deriveVisualizationFromBase.</li>
</ul>
```javascript
var definition = {
    typeName: 'symbolPrimer',
    datasourceBehavior: PV.Extensibility.Enums.DatasourceBehaviors.Single,
    supportsCollections: true,
    visObjectType: symbolPrimer,
    getDefaultConfig: function() {
        return {
            DataShape: 'Value',
            Height: 70,
            Width: 250,
            Opacity: "1.0",
            BackgroundColor: 'grey',
            TextColor: 'white',
            ShowLabel: true,
            ShowTime: false,
            ShowUnits: false,
            MinValue: 0,
            MaxValue: 100
        };
    },
    configTitle: 'Format Symbol Primer',
    StateVariables: ['MultistateColor']
};
```

#### Initialization function

The Initialization function is used to set callback functions for the symbol which drive the symbol's behavior. The function accepts scope and element parameters.
<ul>
	<li><code>scope</code>: Provides access to PI Vision variables available to the symbol.</li>
	<li><code>codeelem</code>: Provides access to the HTML DOM element of the symbol's presentation layer. </li>
</ul>

```javascript
symbolPrimer.prototype.init = function(scope, elem) {

};
```

#### Callback functions

The symbol defined in this document defines callback functions to handle data update, resize, and configuration change events.

```javascript
this.onDataUpdate = dataUpdate;
this.onResize = symbolResize;
this.onConfigChange = configChange;
```

#### `onDataUpdate` callback

This function is called by the PI Vision infrastructure any time a data update occurs. The properties on the object returned are determined by the DataShape specified in the getDefaultConfig function.

<ul>
	<li>Infrequently changed metadata values such as label, units, and path are contained in the first callback invocation and intermittently thereafter. The code checks to see if the label and units were provided prior to using them.</li>
	<li>The primer symbol sets the opacity for the presentation layer based on the data value and the configured minimum/maximum. Note that the PI Web API can be used to obtain the min and max data item values.</li>
</ul>

```javascript
function dataUpdate(data) {
    if (data) {
        scope.value = data.Value;
        scope.config.Opacity = (data.Value - scope.config.MinValue) / (scope.config.MaxValue - scope.config.MinValue);
        scope.time = data.Time;
        if (data.Label) {
            scope.label = data.Label;
        }
        if (data.Units) {
            scope.Units = data.Units;
        }
    }
}
```

#### `onResize` callback

This function is called by the PI Vision infrastructure when the symbol is resized.

<ul>
	<li>The HTML DOM element, elem, passed into the Initialization function by PI Vision, for the symbol is used to find the symbol&#39;s container and then the container is resized.</li>
</ul>

```javascript
function symbolResize(width, height) {
    var SymbolContainer = elem.find('.SymbolPrimerLayout')[0];
    if (SymbolContainer) {
        SymbolContainer.style.width = width + 'px';
        SymbolContainer.style.height = height + 'px';
    }
}
```

#### `onConfigChange` callback

This function is called by the PI Vision infrastructure anytime the configuration of a symbol is updated by the user in the Configuration layer.
<ul>
	<li>The primer symbol uses this callback to validate the minimum and maximum values provided by the user in the Configuration pane.</li>
</ul>

```javascript
function configChange(newConfig, oldConfig) {
    if (newConfig && oldConfig && !angular.equals(newConfig, oldConfig)) {
        if (!isNumeric(newConfig.MinValue) || !isNumeric(newConfig.MaxValue) || parseFloat(newConfig.MinValue) >= parseFloat(newConfig.MaxValue)) {
            newConfig.MinValue = oldConfig.MinValue;
            newConfig.MaxValue = oldConfig.MaxValue;
        }
    }
}

function isNumeric(n) {
    return n === '' || n === '-' || !isNaN(parseFloat(n)) && isFinite(n);
}
```

### The symbol in PI Vision

We created an ![Symbol Primer](./images/image001.png) icon for the symbol primer. This icon is displayed by PI Vision in the Assets tool panel.

To use the symbol:

* Click the ![Symbol Primer](./images/image001.png) icon
* Navigate to an attribute with a numeric value
* Drag the attribute onto the display panel
* To display the configuration panel:
  * Right-click on the symbol to display the popup context menu
  * Click the &quot;Format Symbol Primer&quot; menu option

 ![PI Vision 3 Display](./images/image003.png)

## Addendum

### PI Web API HTTP request example

To make a HTTP request from a custom symbol, you will need to use an AngularJS $http provider. The following example uses the $http provider to get the available AF servers. Depending on your CORS (Cross-Origin Resource Sharing) configuration you may need to start Chrome with the --disable-web-security flag (debug testing only).
<ul>
	<li>Inject a &#39;$http&#39; provider string into symbol definition</li>
</ul>

```javascript
var definition = {
    inject: ['$http'],
};
```
<ul>
	<li>Pass the $http provider as a parameter to the Initialization function</li>
</ul>
```
symbolPrimer.prototype.init = function (scope, elem, $http) {

    };
```

<ul>
	<li>Make a <code>GET HTTP</code> request for the list of all available asset servers and output results to the console.</li>
</ul>
```javascript
symbolPrimer.prototype.init = function(scope, elem, $http) {
    var baseUrl = PV.ClientSettings.PIWebAPIUrl;
    $http.get(baseUrl + '/assetservers').then(function(response) {
        console.log(response);
    });
};
```