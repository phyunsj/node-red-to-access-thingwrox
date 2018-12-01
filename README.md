# Node-RED as an Edge Device to connect ThingWrox 

<p align="center">
<img src="https://github.com/phyunsj/node-red-as-edge-device-to-thingwrox/blob/master/images/thingworx-foundation.png" width="300px"/>
</p>


 >  :computer: Product Information : [ThingWrox ThingWorx Industrial IoT](https://www.ptc.com/en/products/iot/thingworx-platform?cl1=IoT_General_Google_CLC-cpc-ThingWorxBrandCampaign-33029&cmsrc=Google&cid=7012A000001UvT4QAK&elqCampaignId=9998&gclid=EAIaIQobChMIztXeu_T83gIVk4TICh2AiQL3EAAYASAAEgJvXPD_BwE)


## Using REST API to modify ThingWrox Properties

**URL Pattern**

Based on the terms provided above, the following URL pattern is used when communicating with ThingWorx:

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

TBD

## Flow

TBD

## Related Posts

- [ThingWrox Foundation](https://developer.thingworx.com/resources/downloads/foundation-trial-edition)
- [ThingWorx REST API](http://support.ptc.com/cs/help/thingworx_hc/thingworx_6.0_hc/index.jspx?id=thingworx10&action=show)
- [Drag and Drop Edge Device Development with ThingWorx](https://community.ptc.com/t5/IoT-Tech-Tips/Drag-and-Drop-Edge-Device-Development-with-ThingWorx-Node-js-and/td-p/535127)
