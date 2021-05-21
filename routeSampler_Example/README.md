# Creating Traffic in SUMO with routeSampler 

### To create routes with routeSampler.py from counting data, we need the following files:

+ osm.net.xml : the SUMO net file 
+ myDetectors.xml : the file the traffic counting detector position
+ myFlows.txt : the file with the traffic counting information



#### myFlows.txt:

```textfile
	Detector;Time;qPKW;qLKW;vPKW;vLKW
	Detector_1;0;13;3;50;50
	Detector_1;30;17;9;50;50
```

Hint:

* Time values (second column) in minutes!!!	
* In our case we have 13 PKW and 3 LKW from minute 0 to 30 and 17 PKWs and 9LKWs from minute 30 to  60 at our detector "Detector:1"
* The value "50" in the list at vPKW; vLKW means velocity 	

​	

#### myDetectors.xml

```xml
	<detectors>
		<detectorDefinition id="Detector_1" lane="164086589_3" pos="0.00"/>
	</detectors>
```



​	Hint: 
​		Here we assign the detectors to certain lanes (and lanePositions)
​		The detector positions can be be converted to POIs an then be displayed in the SUMO Map
​			-> See Section "dfrouter" at the end of this file



### Step 1: we create all possible routes

```bash
python3 $SUMO_HOME/tools/randomTrips.py -n osm.net.xml -r sampleRoutes.rou.xml
```

​	-> we get the file: sampleRoutes.rou.xml



### Step 2: we create the edgedata.xml 

```bash
python3 $SUMO_HOME/tools/detector/edgeDataFromFlow.py -d myDetectors.xml -f myFlows.txt -o edgedata.xml -v --begin 0 --end 3600 --interval 30
```



Attention: Unfortunately this ist totally confusing here:
		"--begin 0" and "--end 3600" : the values are seconds!!! 
		"--interval 30" : the values are minutes!!! 



###  Step 3: now we use routeSampler.py



We have to create the route files for PKW and LKW separately: 

##### create routes for PKWs:

```bash
python3 $SUMO_HOME/tools/routeSampler.py -r sampleRoutes.rou.xml --edgedata-files edgedata.xml -o routesPKW.xml -v --edgedata-attribute qPKW --prefix "pkw_" --attributes="type=\"veh_passenger\" departLane=\"best\" departSpeed=\"max\" departPos=\"random\""
```

-> we get routesPKW.xml



##### create routes for LKWs:

```bash
python3 $SUMO_HOME/tools/routeSampler.py -r sampleRoutes.rou.xml --edgedata-files edgedata.xml -o routesLKW.xml -v --edgedata-attribute qLKW --prefix "lkw_" --attributes="type=\"truck_truck\" departLane=\"best\" departSpeed=\"max\" departPos=\"random\""
```

-> we get routesLKW.xml



### Step 4: create an additional file: vehicleTypes.xml

here we define the vehicle types:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://sumo.dlr.de/xsd/routes_file.xsd">
    <vType id="veh_passenger" vClass="passenger"/>  
    <vType id="truck_truck" vClass="truck"/> 
</routes>
```



### Step 5: include the new files to osm.sumocfg

```xml
	<input>
    	<net-file value="osm.net.xml"/>
    	<additional-files value="vehicleTypes.xml, routesPKW.xml, routesLKW"/>
	</input>
```





___________________________________________________________________________________________

### Step 6 (optionally) 

#### we use dfrouter here to convert detector positions into POIs which can be displayed in the SUMO Map

```bash
dfrouter -c dfRouterConfigFile.conf
```

 using dfRouterConfigFile.conf :

```xml
<configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://sumo.dlr.de/xsd/dfrouterConfiguration.xsd">
<input>
    <net-file value="../osm.net.xml" />
    <detector-files value="../detectors/detectors.xml" />
    <measure-files value="../detectors/flows.detectors.csv" />
</input>

<output>
    <routes-output value="../routes.rou.xml" />
    <detector-output value="../out.detectors.xml" />
    <detectors-poi-output value="../poi.detectors.xml" />
    <emitters-output value="../vehicles.xml" />
    <vtype value="true" />
    <vtype-output value="../vtype.xml" />
	<routes-for-all value="true" />
</output>

<processing>
	<guess-empty-flows value="true" />
    <disallowed-edges value="" />
    <time-step value="1800" />
    <strict-sources value="true" />
    <scale value="0.3" />
	<max-search-depth value="300" />
	<highway-mode value="false" />
	<respect-concurrent-inflows value="true" />
</processing>
</configuration>
```












