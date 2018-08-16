# GeoServer + OpenLayer

* GeoWebcCache相当于OpenLayer和GeoServer之间的中介，GeoWebCache会根据你的配置信息，把相应的地图图层切好图，存放在磁盘中，然后在使用OpenLayer加载地图服务的时候，把地图服务的地址指向GeoWebCache，GeoWebCache接收到这些请求后，会根据请求的位置和比例尺在切片目录中找到对应的瓦片，然后返回，省去了动态生成地图的过程，速度大幅度提高，且由于请求的图片资源是事先生成好的，浏览器加载这些图片后，下次再请求同样的图片，就会从浏览器的缓存中拉去，速度进一步提高。

## GeoWebCache配置

> 在WEB-INF目录下找到一系列配置文件，先找到web.xml，然后在web-app根元素下添加：

```xml
<context-param>
    <param-name>GEOWEBCACHE_CACHE_DIR</param-name>
    <param-value>你的GeoWebCache</param-value>
</context-param>
```

>param-value的值就是你要存放GeoWebCache瓦片的位置，配置好这里，重启tomcat，你会发现在你的瓦片目录下生成了一些文件，其中就有GeoWebCache.xml，这个文件是GeoWebCache配置的关键所在，以下是这个文件的配置信息：

```xml
<?xml version="1.0" encoding="utf-8"?>
<gwcConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"     xmlns="http://geowebcache.org/schema/1.3.0" xsi:schemaLocation="http://geowebcache.org/schema/1.3.0
http://geowebcache.org/schema/1.3.0/geowebcache.xsd">
    <version>1.3.0</version>
    <backendTimeout>120</backendTimeout>
    <serviceInformation>
        <title>GeoWebCache</title>
        <description>GeoWebCache is an advanced title cache for WMS server.It supports a large variety of protocols and formats,including WMS-C,WMTS,KML,Google Maps and Virtual Earth.</description>
        <keywords>
            <string>WFS</string>
            <string>WMS</string>
            <string>WMTS</string>
            <string>GEOWEBCACHE</string>
        </keywords>
        <serviceProvider>
            <providerName>John Smith inc.</providerName>
            <providerSite>http://www.example.com/</providerSite>
            <serviceContact>
                <individualName>John Smith</individualName>
                <positionName>Geospatial Expert</positionName>
                <addressType>Work</addressType>
                <addressStreet>1 Bumpy St.</addressStreet>
                <addressCity>Hobart</addressCity>
                <addressAdministrativeArea>TAS</addressAdministrativeArea>
                <addressPostalCode>7005</addressPostalCode>
                <addressCountry>Australia</addressCountry>
                <phoneNumber>+61 3 0000 0000</phoneNumber>
                <faxNumber>+61 3 0000 0001</faxNumber>
                <addressEmail>john.smith@example.com</addressEmail>
            </serviceContact>
        </serviceProvider>
        <fees>NONE</fees>
        <accessConstraints>NONE</accessConstraints>
    </serviceInformation>

    <!--下面定义的是瓦片格网的信息，主要配置投影名称，数据框（坐标范围），比例尺集合，瓦片大小，geowebcache会根据这些信息来分割地图-->
    <gridSets>
        <!-- Grid Set Example, by default EPSG:900913 and EPSG:4326 are defined -->
        <gridSet>
            <!-- 格网信息的名称，这里只是一个标识，可以随便起，下面配置wmsLayer的时候会用到 -->
            <name>EPSG:3395</name>
            <!-- 这里对应就是真正的投影名称了，Geowebcache本身并不认得这些投影名称，因为地图数据是从地图服务器里来的，这些信息最终是要传到地图服务器中去的，所以这里只要和地图服务器中的投影名称一致就可以了 -->
            <srs><number>3395</number></srs>
            <!-- 地图图层的坐标范围，也可以理解为，你需要切图的范围，可以不指定 -->
            <extent>
                <coords>
                    <double>12063355.362599999</double>
                    <double>3248729.1457272936</double>
                    <double>13122908.970199998</double>
                    <double>3908502.2175705903</double>
                </coords>
            </extent>
            <!-- 分辨率集合（也就是定义缩放的级别），一个像素点代表多少地图单位，和比例尺的意思一样，这里定义了11个缩放级别 -->
            <resolutions>
                <double>1000.4406398437495</double>
                <double>517.3601599609374</double>
                <double>258.6800799804687</double>
                <double>129.34003999023435</double>            <double>64.67001999511717</double>
                <double>32.335009997558586</double>
                <double>16.167504998779293</double>
                <double>8.083752499389647</double>
                <double>4.0104690624237058</double>
                <double>2.25261726560592646</double>
                <double>1.12630863280296323</double>
            </resolutions>
            <!-- 每个单位所代表的长度 -->
            <metersPerUnit>1</metersPerUnit>
            <pixelSize>0.0002645833333333333333333333</pixelSize>
            <!-- 瓦片的长宽 -->
            <tileHeight>256</tileHeight>
            <tileWidth>256</tileWidth>
        <gridSet>
    </gridSets>

    <layers>
        <wmsLayer>
        <!-- 地图名称，这个会在OpenLayer调用的时候中用到 -->
            <name>heightway</name>
            <metalInformation>
                <title>heightway</title>
                <description>heightway</description>
            </metalInformation>
            <!-- 图片格式 -->
            <mimeFormats>
                <string>image/jpeg</string>
            </mimeFormats>
            <!-- 使用的瓦片格网，就是上面所配置的格网信息 -->
            <gridSubsets>
                <gridSubset>
                    <gridSetName>EPSG:3395</gridSetName>
                </gridSubset>
            </gridSubsets>
            <!-- wms 服务地址 -->
            <wmsUrl>
                <string>http://localhost:8006/geoserver/cite/wms?service=wms</string>
            </wmsUrl>
            <wmsLayers>cite:heightWay</wmsLayers>
            <!--是否透明-->
            <transparent>false</transparent>
            <!--背景色-->
            <bgColor>#FCFCFC</bgColor>
        </wmsLayer>
    </layers>
</gwcConfiguration>
```

* 配置好上面的信息之后，进入：<http://localhost:8006/geowebcache/demo> 点击"Reload Configuration"重新读取配置信息
* 如果需要你输入密码，密码信息在WEB-INF\users.properties这个文件夹中
* 重新进入：<http://localhost:8006/geowebcache/demo> 如果配置信息没错，所配置的图层信息会全部显示在该页面上:</br>![Image text](https://static.oschina.net/uploads/space/2013/0129/135055_21i7_189876.png)
* 点击"Seed this Layer"，然后你需要输入:</br>![Image Text](https://static.oschina.net/uploads/space/2013/0129/140404_hEbe_189876.png)
* 设置好，点击"submit"就开始切图了

## OpenLayer调用Geowebcache瓦片

```javaScript
var options = {
    resolutions:[1.12630863280296323,2.25261726560592646,4.0104690624237058,8.083752499389647,16.167504998779293,32.335009997558586,64.67001999511717,129.34003999023435,258.6800799804687,517.3601599609374,1000.4406398437495],
    projection: new OpenLayers.Projection("EPSG:3395"),
    units:"meters",
    maxExtent:bounds
};
//初始化地图对象
var map = new OpenLayers.Map("GisMap",option);
//底图
var baseLayer = new OpenLayers.Layer.WMS(
    "baseLayer",
    "http://localhost:8006/geowebcache/service/wms",
    {layers:"heightway",format:'image/jpeg'},
    {tileSize: new OpenLayers.Size(256,256)}
);
map.addLayers([baseLayer]);
```