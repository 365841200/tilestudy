# 自定义图层

## 如何定义图层

* GWC的主要配置文件是geowebcache.xml，默认情况下，它与缓存储存在同一个路径下面，如果不存在该文件，GWC会拷贝一份初始配置文件到缓存路径下。除非有特别指明，所有的配置改变都是通过编辑geowebcache.xml实现的。所有的配置选项和参数都是基于XML Schema。可通过连接查询了解GWC的schema细节：

> GeoWebCache schema <http://geowebcache.org/schema/--version--/geowebcache.xsd></br>
> GeoWebCache schema docs <http://geowebcache.org/schema/docs/--version--/></br>
> 将--version--替换为当前版本号

***

## 自定义投影

* GWC默认仅支持EPSG:4326(Longitude/latitude)和EPSG:900913(spherical web mercator)，其他投影需要手动定义配置。添加自定义投影需要修改geowebcache.xml配置文件，添加投影定义然后便可将图层与投影结合起来。

### 定义自定义投影

> GWC无法识别投影，它仅知道自己的瓦片网格，这些信息会在向WMS传递信息时使用，假定WMS是可以理解自定义投影的请求。

创建自定义投影时，需要以下信息：

| 字段 | 标签 | 描述 |
|:-:|:-:|:-:|
|Name| \<name> | 投影名称 |
|SRS number|\<srs>\<number>|空间参照系统的数字定义（例：EPSG:2263）|
|SRS extent|\<extent>\<coords>|投影的完整范围|
|List of valid resolutions|\<resolutions>|分辨率列表，通常按一级两倍进行减小，不是必须的|
|Meters per unit|\<metersPerUnit>|投影上每米的恒定值|
|Tile height and width|\<tileHeight>,\<tileWidth>|通常为256*256，其他值也是支持的|

> 当定义新的SRS/gridset,必须使用完整的投影范围，即使数据在投影范围中只有一小部分

#### 定义SRS Extent

* 如果不确定空间参照系统的范围，可以使用网上资源(<http://spatialreference.org>)来确定投影边界。例如：在EPSG:2263的参数页中，投影边界为：

>909126.0155 , 110626.2880 , 1610215.3590 , 424498.0529

#### 定义 Meters per unit

* 虽然许多空间参照系的单位定义为米，但还是可以定义为其他单位。设置后可以防止格网变化和错误输出。下面为一张单位换算表格：

|单位|每米|
|:-:|:-:|
|度|111319.49079327358|
|英尺|0.3048|
|英寸|0.0254|
|米|1|

*下面为自定义投影例子：

```xml
<gridSets>
  ...
  <gridSet>
    <name>EPSG:2263</name>
    <srs><number>2263</number></srs>
    <extent>
      <coords>
        <double>909126.0155</double>
        <double>110626.2880</double>
        <double>1610215.3590</double>
        <double>424498.0529</double>
      </coords>
    </extent>
    <resolutions>
      <double>466.2771277605631</double>
      <double>233.13856388028155</double>
      <double>116.569281940140775</double>
      <double>58.2846409700703875</double>
      <double>29.14232048503519375</double>
      <double>14.571160242517596875</double>
      <double>7.2855801212587984375</double>
    </resolutions>
    <metersPerUnit>0.3048</metersPerUnit>
    <tileHeight>256</tileHeight>
    <tileWidth>256</tileWidth>
  </gridSet>
  ...
</gridSets>
```

***

## 应用自定义投影

* 当GWC加载了投影后，下一步就是将投影应用到图层上，这也是在geowebcache.xml文件中进行配置的。

必要信息如下：
|字段|标签|描述|
|:-:|:-:|:-:|
|Layer Name|\<name>|发布图层名|
|Name of projection|\<gridSetName>|投影名称|
|Layer extent|\<extent>\<corrds>|图层的范围|
|WMS URL|\<wmsUrl>|WMS路径|

例子：

```xml
<layers>
  ...
  <wmsLayer>
    <name>my:layer</name>
    <gridSubsets>
      <gridSubset>
        <gridSetName>EPSG:2263</gridSetName>
        <extent>
          <coords>
            <double>937558.37821372</double>
            <double>89539.946131199</double>
            <double>1090288.6738155</double>
            <double>327948.21243638</double>
          </coords>
        </extent>
      </gridSubset>
    </gridSubsets>
    <wmsUrl><string>http://myserver/geoserver/wms</string></wmsUrl>
  </wmsLayer>
  ...
</layers>
```

***对geowebcache.xml进行修改后，需要reload  the configuration***