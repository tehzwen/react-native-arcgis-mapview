<h1>ArcGIS React Native Library</h1>

This App is a proof of concept for now showing that ArcGIS can be used in react native to draw maps, layers and interact with the maps and layers to display data relevant to the points on the layer.

The App utilizes react-native-arcgis-mapview written by davidgalindo on Github. The library had a ton of very useful methods/props that were perfect for the project.

It took a lot of work to get the library to even run with the current version of React Native. Below are some of the steps that I took to get it work with my project. My version of the library can be found 
here https://github.com/tehzwen/react-native-arcgis-mapview. It will be the latest update that is being used in the app. It currently does not support iOS as that has yet to be added.

<h2>Getting the library setup</h2>

<h3>Android Instructions </h3>
- Use npm or yarn to install davidgalindo's library ie. ```yarn add react-native-arcgis-mapview``` or ```npm install react-native-arcgis-mapview --save```.
- Clone the repository https://github.com/tehzwen/react-native-arcgis-mapview and add it to your node modules folder after cloning it which will override some of the files from the existing library.
- Follow davidgalindo's instructions on https://github.com/davidgalindo/react-native-arcgis-mapview under android including adding the extra fields to your build.gradle in ```android\```:
    1. Under allProjects add the following ```        maven {
            url 'https://esri.bintray.com/arcgis'
        }```.
        Do not try to add this to any existing maven field there. Instead add it below as its own field.
    2. In the same fiel I needed to change my ```minSdkVersion = 19``` which I believe was previously 16.

- Next you will need to edit the build.gradle file inside of ```android\app```:
    1. First ensure that under dependencies you have a line that reads ```implementation project(':react-native-arcgis-mapview')```. If it is not present, add it.
    2. The next thing I had to do (the app would run in the emulator just fine but not on my actual mobile device without this) was to add ```        ndk {
            abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
        }``` under the field ```defaultConfig```. 
    3. Make sure that under ```compileOptions``` you have the following ```    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }```

- In the file ```AndroidManifest.xml``` found in ```android\app\src\main``` you will need to add ```<uses-permission android:name="android.permission.INTERNET" />```
- Inside the folder ```android\app\src\main\java\com\yourprojectname``` there will be a ```MainApplication.java``` file.  This file requires the following lines: 
    1. ```import com.davidgalindo.rnarcgismapview.RNArcGISMapViewPackage;``` at the top with your other imports.
    2. Under the 
        ```@Override
        protected List<ReactPackage> getPackages() {
                return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new RNArcGISMapViewPackage(),
            );
        } 
        ```
    you need to add the ```new RNArcGISMapViewPackage(),``` as seen in the above example of my file.

- Lastly you should check your ```settings.gradle``` file under ```android/``` and ensure that the line

    ```
    include ':react-native-arcgis-mapview'
    project(':react-native-arcgis-mapview').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-arcgis-mapview/android')
    ```

Once those steps have been completed you should be able to import the library into your App.js file and create a simple MapView. Below is a simple example:

```javascript
import React, { Component } from 'react';
import { View } from 'react-native';
import ArcGISMapView from 'react-native-arcgis-mapview';

export default class MapScreen extends Component {
    constructor(props) {
        super(props);

        this._mapView = "none";
    }

    render() {
        return (
            <View>
                <ArcGISMapView ref={mapView => this._mapView = mapView}
                    style={{ height: 300, width: 300 }}
                    initialMapCenter={[{ latitude: 34.05564888958834, longitude: -118.51107463698345 }]}
                    onSingleTap={(event) => this.getTapInformation(event)}
                    onMapDidLoad={() => console.log("Map loaded")}
                    basemapUrl={"url provided by arcGIS website of map"}
                />
            </View>
        );
    }
}
```

<h2>Connecting Android Functions/Methods to React Native</h2>
Following the patterns found in davidgalindo's library I will give a brief example on how to create a method in android and then how to access said method in React Native. For the sake of this example we will use our ArcGISMapView library.

Our function will do the following:
    - take a string argument called url which represents a URL for an arcGIS feature layer.
    - create the feature layer from the url
    - display the layer on the Map.

In the `C:\Users\zshaw\Documents\rnprototype\node_modules\react-native-arcgis-mapview\android\src\main\java\com\davidgalindo\rnarcgismapview\` folder you will find a file called RNAGSMapView.java. This file is where we will write the logic for what our function will do on the android side (native side) of the app when called by React Native. 

Start by writing the function as follows 
```java
    public void addMapLayer(String url) {
        ServiceFeatureTable serviceFeatureTable = new ServiceFeatureTable(url);
        FeatureLayer featureLayer = new FeatureLayer(serviceFeatureTable);
        ArcGISMap map = mapView.getMap();
        map.getOperationalLayers().add(featureLayer);
    }
```

- The first line is creating a ServiceFeatureTable from the url we passed as an argument
- Next the FeatureLayer itself is created from that ServiceFeatureTable which can then be applied to the ArcGISMap.
- mapView holds a reference to the current MapView and by calling getMap() we retreive the basemap attached to our view. 
- The final call is made to get the current layers on the map and add the feature layer we created. 

Now that we have written our function we need to open the RNArcGISMapViewManager.java file, this file contains the definitions for how we will pass the methods to React Native.

At the top of the file you'll notice that there are several constants. We will make one for our new method `private final int ADD_MAP_LAYER = 10;`.

Now look for the method getCommandsMap. Here we will create a command map that holds references to all of our methods using the constants and string name of our method. We will add our new method definition like this:

```java
    @Override
    public Map<String, Integer> getCommandsMap() {
        Map<String, Integer> map = MapBuilder.of(
                "showCalloutViaManager",SHOW_CALLOUT,
                "centerMapViaManager", CENTER_MAP,
                "addGraphicsOverlayViaManager", ADD_GRAPHICS_OVERLAY,
                "removeGraphicsOverlayViaManager",REMOVE_GRAPHICS_OVERLAY,
                "addPointsToOverlayViaManager",ADD_POINTS_TO_OVERLAY,
                "removePointsFromOverlayViaManager", REMOVE_POINTS_FROM_OVERLAY,
                "updatePointsInGraphicsOverlayViaManager", UPDATE_POINTS_IN_GRAPHICS_OVERLAY
        );
        map.put("routeGraphicsOverlayViaManager",ROUTE_GRAPHICS_OVERLAY);
        map.put("setRouteIsVisible", SET_ROUTE_IS_VISIBLE);
        map.put("addMapLayer", ADD_MAP_LAYER); //this is our new method to add.
        map.put("dispose", DISPOSE);
        return map;
    }


```

Now look for the receiveCommand method. We will add a case for our method inside of it like this:
```java
    @Override
    public void receiveCommand(RNAGSMapView mapView, int command, ReadableArray args) {
        Assertions.assertNotNull(mapView);
        Assertions.assertNotNull(args);
        switch (command) {
            case SHOW_CALLOUT: mapView.showCallout(args.getMap(0));return;
            case CENTER_MAP: mapView.centerMap(args.getArray(0));return;
            case ADD_GRAPHICS_OVERLAY: mapView.addGraphicsOverlay(args.getMap(0));return;
            case REMOVE_GRAPHICS_OVERLAY: mapView.removeGraphicsOverlay(args.getString(0));return;
            case ADD_POINTS_TO_OVERLAY: mapView.addPointsToOverlay(args.getMap(0));return;
            case REMOVE_POINTS_FROM_OVERLAY: mapView.removePointsFromOverlay(args.getMap(0));return;
            case UPDATE_POINTS_IN_GRAPHICS_OVERLAY: mapView.updatePointsInGraphicsOverlay(args.getMap(0));return;
            case ROUTE_GRAPHICS_OVERLAY: mapView.routeGraphicsOverlay(args.getMap(0));return;
            case SET_ROUTE_IS_VISIBLE: mapView.setRouteIsVisible(args.getBoolean(0));return;
            case ADD_MAP_LAYER: mapView.addMapLayer(args.getString(0));return; //this is our method here
            case DISPOSE: mapView.onHostDestroy();
        }
    }

```

You'll notice that we pass it args.getString(0) which tells our receiveCommand that it is expected to get arguments that that the first argument should be a string (our url that we talked about above).


The last step we need to do is to add the definition of this method to our actual javascript library file AGSMapView.js found in the root directory of the library.

Inside this file you will see several functions declared, we will do the same thing but with our newly written function. 

```javascript
  addMapLayer = (args) => {
    UIManager.dispatchViewManagerCommand(
      findNodeHandle(this.agsMapRef),
      UIManager.getViewManagerConfig('RNArcGISMapView').Commands.addMapLayer,
      [args]
    );
  }
```

This code tells the library how and where to access the function in android so we can use it in React Native.

Now that this is done we can refer back to the first example I posted of the MapView in React Native:

```javascript
        return (
            <View>
                <ArcGISMapView ref={mapView => this._mapView = mapView}
                    style={{ height: 300, width: 300 }}
                    initialMapCenter={[{ latitude: 34.05564888958834, longitude: -118.51107463698345 }]}
                    onSingleTap={(event) => this.getTapInformation(event)}
                    onMapDidLoad={() => console.log("Map loaded")}
                    basemapUrl={"url provided by arcGIS website of map"}
                />
                <Button onPress={() => this._mapView.addMapLayer("example layer url")}>
                    <Text>Add a layer</Text>
                </Button>
            </View>
        );

```

We have now added a button to our screen that will allow us to call the function we just wrote by using the reference to the mapview. We can now call void functions in native android from React Native. Next we will find out how we can retrieve values from methods in Android.

<h2>Getting Values from Android to React Native</h2>

There are situations where we might be interested in getting return values from native android whether its the value of something we are interested in or some data that can only be retreived on the native side we can use callbacks and ReactMethods to transfer this data.

Again we need to go into our RNAGSMapView.java file and write the method we want to return some value.

```java
public String getTestValue() {
    return "hey there";
}
```

This simple function will return the string `hey there` when called. We can obviously do more complicated methods and return values from them, the important thing to keep in mind if we need to return objects is to use something like gson to encode it before passing it.

Next let's add this method to our module file `RNArcGISMapViewModule.java` like so:

```java
@ReactMethod
public void getTestValue(final int viewId, final Callback callback) {
        UIManagerModule uiManagerModule = getReactApplicationContext().getNativeModule(UIManagerModule.class);
        uiManagerModule.addUIBlock(nativeViewHierarchyManager -> {
            View view = nativeViewHierarchyManager.resolveView(viewId);
            if (view instanceof RNAGSMapView) {
                String result = ((RNAGSMapView) view).getTestValue();
                callback.invoke(result);
            }
        });
    }
```

This method is a ReactMethod written in our module to communicate what we want android to do when called. It finds the correct View (in this case our mapview) and ensures that it is the type we want, then it gets the value from the method we just wrote by calling it, and lastly it will invoke our callback with the string passed to it.

Now we will add the function definition to our `AGSMapView.js` file:

```javascript
  getTestValue = (callback) => {
    if (Platform.OS === 'ios') {
      UIManager.dispatchViewManagerCommand(
        findNodeHandle(this.agsMapRef),
        UIManager.getViewManagerConfig('RNArcGISMapView').Commands.getTestValueViaManager,
        [callback]
      );
    } else {
      NativeModules.RNArcGISMapViewManager.getTestValue(findNodeHandle(this.agsMapRef), callback);
    }
  };
```

Now our react native library has a reference to our method and knows to use callbacks to get the values. The last step is to use our newly made function in our app. I'll use the same example we had above and simply add a button to call this function.

Start by writing the button:
```javascript
        return (
            <View>
                <ArcGISMapView ref={mapView => this._mapView = mapView}
                    style={{ height: 300, width: 300 }}
                    initialMapCenter={[{ latitude: 34.05564888958834, longitude: -118.51107463698345 }]}
                    onSingleTap={(event) => this.getTapInformation(event)}
                    onMapDidLoad={() => console.log("Map loaded")}
                    basemapUrl={"url provided by arcGIS website of map"}
                />
                <Button onPress={() => this._mapView.addMapLayer("example layer url")}>
                    <Text>Add a layer</Text>
                </Button>
                
                <Button onPress={this.testFunction}>
                    <Text>PRESS ME</Text>
                </Button>
            </View>
        );
```

Now that we added that button we need to define `this.testFunction` so our App knows what to do when it is pressed:

```javascript
    testFunction() {
       this._mapView.getTestValue((value) => {
           console.log(value);
       })
    }
```

This function will use the ref to our MapView and call the function we just wrote. The only issue is that we need to make our function bound to the correct context otherwise it won't have access to the MapView ref. Inside the constructor lets add this:

```javascript
this.testFunction = this.testFunction.bind(this);
```

Perfect, now when we open our app and run `react-native log-android` we will see that on button press we get the value `hey there` in the terminal.







