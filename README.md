# Node-RED as an Edge Device to connect ThingWrox 

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/thingworx-foundation.png" width="300px"/>
</p>


 >  :computer: Product Information : [ThingWrox ThingWorx Industrial IoT](https://www.ptc.com/en/products/iot/thingworx-platform?cl1=IoT_General_Google_CLC-cpc-ThingWorxBrandCampaign-33029&cmsrc=Google&cid=7012A000001UvT4QAK&elqCampaignId=9998&gclid=EAIaIQobChMIztXeu_T83gIVk4TICh2AiQL3EAAYASAAEgJvXPD_BwE)


## Using REST API to modify ThingWrox Properties

**URL Pattern**

The following URL pattern is used when communicating with ThingWorx:

```<http/https:><host:port>/Thingworx/<entity collection>/<entity>/<characteristic collection>/<characteristic>?<query parameters>```
 
**HTTP Header**

```
newMsg.headers['Accept'] = "application/json";
newMsg.headers['content-type'] = "application/json";
newMsg.headers['content-length'] = newMsg.payload.length;
newMsg.headers["appKey"] = '*********-4958-45db-9789-************';
```

**HTTP URL**

 ```http://<host>:<port>/Thingworx/Things/Building12/Properties/*```
 
 
## In Action

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/node-red-thingwrox-in-action.gif" width="800px"/>
</p>

## Flow

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/node-red-thingwrox-flow.png" width="800px"/>
</p>

If the connection is down, a single queue will store all events upto `totalAllowedQueueSize(20)`.
Once the connection is re-established, the oldest event will be sent first.

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/node-red-thingwrox-flow-error.png" width="800px"/>
</p>

```
[{"id":"4911b967.24ef58","type":"inject","z":"d70af7bd.4d2698","name":"Generate Temp, Humidity 5s","topic":"","payload":"","payloadType":"date","repeat":"5","crontab":"","once":false,"onceDelay":0.1,"x":175.10000610351562,"y":94,"wires":[["d8d8507f.5db5"]]},{"id":"214817f8.7eb3e8","type":"http request","z":"d70af7bd.4d2698","name":"REST PUT -> ThingWrox ","method":"PUT","ret":"txt","url":"http://192.168.1.10:8080/Thingworx/Things/Building12/Properties/*","tls":"","x":749.10009765625,"y":266,"wires":[["5994701.9200d9","ff815965.15afd8"]]},{"id":"2ed759d1.f63c76","type":"debug","z":"d70af7bd.4d2698","name":"HTTP PUT Messasge ","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","x":934.1000366210938,"y":138,"wires":[]},{"id":"5994701.9200d9","type":"function","z":"d70af7bd.4d2698","name":"Detect HTTP Status","func":"var newMsg = {};\n\nif ( msg.statusCode !== 200 ) {\n    // 'ECONNREFUSED') \n    // 405 : method not allowed\n    console.log(\"Failed !!! statusCode = \", msg.statusCode);\n    newMsg.payload = { op: \"connection\", connected : 0 , statusCode :  msg.statusCode };\n} else \n    newMsg.payload = { op: \"connection\", connected : 1 , statusCode :  msg.statusCode };\nreturn newMsg;","outputs":1,"noerr":0,"x":483,"y":330,"wires":[["cdc0336b.69b3e"]]},{"id":"c76fa51a.555828","type":"function","z":"d70af7bd.4d2698","name":"Build HTTP Header w/ appKey","func":"\n// payload\nvar newMsg = {};\nnewMsg.payload = msg.payload;\n\nnewMsg.headers = {};\nnewMsg.headers['Accept'] = \"application/json\";\nnewMsg.headers['content-type'] = \"application/json\";\nnewMsg.headers['content-length'] = newMsg.payload.length;\nnewMsg.headers[\"appKey\"] = '*********-4958-45db-9789-**********';\n\nreturn newMsg;\n","outputs":1,"noerr":0,"x":678,"y":188,"wires":[["214817f8.7eb3e8","2ed759d1.f63c76"]]},{"id":"cdc0336b.69b3e","type":"function","z":"d70af7bd.4d2698","name":"Event Queue (max 20)","func":"context.queue = context.queue || [];\nvar http_connected = context.connected || 0 ;\n\nvar totalAllowedQueueSize = 20;\nvar blocked = false;\nvar newMsg = null;\n\nif (context.queue.length >= totalAllowedQueueSize) blocked = true;\n\nswitch( msg.payload.op ) {\ncase \"connection\" :\n    context.connected = msg.payload.connected;\n    http_connected = msg.payload.connected;\n    break;\ncase \"reset\" :\n    context.queue = [];\n    blocked = false;\n    context.connected = 1;\n    break;\ncase \"trigger\" :\n    // dequeue\n    if ( context.queue.length !== 0 && http_connected ) newMsg = { payload : context.queue.shift() } ;\n    else newMsg = { payload : { op : \"trigger\" } };\n    break;\ndefault :\n    // enqueue\n    if ( !blocked ) context.queue.push(msg.payload);\n    break;\n}\n\nif (http_connected) {\n    if( !blocked )\n        node.status({fill:\"green\",shape:\"dot\",text: '('+context.queue.length+') stored, connected' });\n    else \n        node.status({fill:\"red\",shape:\"ring\",text: 'blocked,('+context.queue.length+') stored, connected' });\n} else {\n    if( !blocked ) \n        node.status({fill:\"red\",shape:\"dot\",text: '('+context.queue.length+') stored, disconnected' });\n    else\n        node.status({fill:\"red\",shape:\"ring\",text: 'blocked,('+context.queue.length+') stored, disconnected' });\n}\n\nreturn newMsg;\n","outputs":1,"noerr":0,"x":416,"y":190,"wires":[["c76fa51a.555828"]]},{"id":"4745042.308b8fc","type":"inject","z":"d70af7bd.4d2698","name":"Trigger Event every 3s","topic":"","payload":" { \"op\" : \"trigger\" }","payloadType":"json","repeat":"3","crontab":"","once":false,"onceDelay":0.1,"x":157,"y":243,"wires":[["cdc0336b.69b3e"]]},{"id":"d8d8507f.5db5","type":"function","z":"d70af7bd.4d2698","name":"Read Temp, Humidity Values","func":"var tempF = Math.floor(Math.random() * 40 + 60);\nvar tempC = Math.floor( (tempF-32)/1.8 ) ;\nvar humidity = Math.floor(Math.random() * 80 + 20);\nmsg.payload = {\n    op : \"store\",\n    timestamp : Math.floor(Date.now() / 1000),\n    Temperature: tempF,\n    Humidity: humidity,\n    Location : \"B12\"\n    };\nreturn msg;\n","outputs":1,"noerr":0,"x":452,"y":94,"wires":[["cdc0336b.69b3e"]]},{"id":"9683d41c.1a5358","type":"inject","z":"d70af7bd.4d2698","name":"Reset","topic":"","payload":"{\"op\" : \"reset\"}","payloadType":"json","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":98,"y":161,"wires":[["cdc0336b.69b3e"]]},{"id":"bdac5bea.6d4128","type":"comment","z":"d70af7bd.4d2698","name":"Events","info":"","x":87,"y":49,"wires":[]},{"id":"ff815965.15afd8","type":"debug","z":"d70af7bd.4d2698","name":"HTTP Response Message","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":930,"y":339,"wires":[]},{"id":"4ca174e.870b18c","type":"comment","z":"d70af7bd.4d2698","name":"Periodic Reporting","info":"","x":122,"y":204,"wires":[]},{"id":"f581d59d.b605f8","type":"comment","z":"d70af7bd.4d2698","name":"Simpel Queue (or Time-series dataase)","info":"","x":379,"y":146,"wires":[]}]
```

## Alert

When an alert condition ( `#Temperature <= 65` ) is met, ThingWorx fires off an alert. Alerts are written to the alert history file and can be viewed through the **Alert Summary** and **Alert History** Mashups. 

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/thingwrox-alert-history.png" width="800px"/>
</p>

## Alert Subscription

Create a **Subscription** to the Alert so that you are automatically notified when an alert is triggered.  

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/thingwrox-alert-postjson.png" width="800px"/>
</p>

Send a message from ThingWrox to an Edge Device (Node-RED) using HTTP REST API when an alert is triggered.

```
var headers = { 
    "content-type": "application/json",
    "Accept":"*/*"            
};

var content = "{ \"op\" : \"inc_temperature\" }";

var params = { 
    headers: headers,
    url: "http://192.168.1.8:1880/alerts" /* STRING */,
    content: content /* STRING */,
};
var result = Resources["ContentLoaderFunctions"].PostJSON(params);
```

Node-RED executes an appropriate action based on the payload recevied by `http in (POST /alerts)` node. 


## Related Posts

- [ThingWrox Foundation](https://developer.thingworx.com/resources/downloads/foundation-trial-edition)
- [ThingWorx REST API](http://support.ptc.com/cs/help/thingworx_hc/thingworx_6.0_hc/index.jspx?id=thingworx10&action=show)
- [Fundamentals of IoT Development with ThingWorx](https://www.udemy.com/thingworx-fundamentals/)
- [Create an Application Key for device authentication](https://developer.thingworx.com/en/resources/guides/create-application-key2)
- [Drag and Drop Edge Device Development with ThingWorx](https://community.ptc.com/t5/IoT-Tech-Tips/Drag-and-Drop-Edge-Device-Development-with-ThingWorx-Node-js-and/td-p/535127) ( :scream: I found  [`node-red-node-thingrest`](https://github.com/obiwan314/node-red-node-thingrest) while I was experimenting function & http nodes with thingwrox foundation server. No dependancy is required with my example.)
