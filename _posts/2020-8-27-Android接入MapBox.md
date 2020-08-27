---
title: Android接入MapBox
date: 2020-8-27 12:00:00
categories:
- Android/开发
tags: Android
--- 

作者：Angki  
转载请注明

---

#### MapBox接入
###### MapBox的[官方文档地址](https://docs.mapbox.com/android/maps/overview/)，[插件地址](https://docs.mapbox.com/android/plugins/overview/)。
###### 大概的接入步骤（按照文档来就行）：
- 注册MapBox账号，创建私密访问令牌（为啥要创建呢，后面会说）。
- 模块级的build.gradle文件添加：
    
    ```
    dependencies {
        implementation 'com.mapbox.mapboxsdk:mapbox-android-sdk:9.3.0'
    }
    ```
- 添加私密令牌：
    
    ```
    在gradle.properties文件中添加
    MAPBOX_DOWNLOADS_TOKEN = 私密令牌
    ```

- 项目级的build.gradle文件添加：
    
    ```
    maven {
            url 'https://api.mapbox.com/downloads/v2/releases/maven'
            authentication {
                basic(BasicAuthentication)
            }
            credentials {
                username = 'mapbox'
                // Use the secret token you stored in gradle.properties as the password
                password = project.properties['MAPBOX_DOWNLOADS_TOKEN'] ?: ""
            }
    }
    ```
- 添加公共令牌：
    
    ```
    打开R.strings.xml文件
    添加 <string name="mapbox_access_token">公共令牌</string>
    ```
- 在清单文件中配置权限：
    ```
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    ```
- 初始化Mapbox，建议在Application中：
    
    ```
    Mapbox.getInstance(this, resources.getString(R.string.mapbox_access_token))
    ```
---
#### MapBox的使用
- 布局文件中添加MapBox

    ```
        <com.mapbox.mapboxsdk.maps.MapView
            android:id="@+id/Activity_Main_Map"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            mapbox:mapbox_cameraZoom="12"/>
        
        mapbox:mapbox_cameraZoom表示初始化的缩放等级
    ```

- 添加生命周期方法
    ```
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Activity_Main_Map.onCreate(savedInstanceState)
    }
    
    override fun onStart() {
        super.onStart()
        Activity_Main_Map.onStart()
    }

    override fun onResume() {
        super.onResume()
        Activity_Main_Map.onResume()
    }

    override fun onPause() {
        super.onPause()
        Activity_Main_Map.onPause()
    }

    override fun onStop() {
        super.onStop()
        Activity_Main_Map.onStop()
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        Activity_Main_Map.onSaveInstanceState(outState)
    }

    override fun onLowMemory() {
        super.onLowMemory()
        Activity_Main_Map.onLowMemory()
    }

    override fun onDestroy() {
        super.onDestroy()
        Activity_Main_Map.onDestroy()
    }
    ```

- 设置地图相关设置，在Activity_Main_Map.onCreate(savedInstanceState)后加入：
    ```
        Activity_Main_Map.getMapAsync{ mapboxMap ->
            
            mapboxMap.setStyle(Style.OUTDOORS) { style ->
                //个性化设置
                val uiSettings = mapboxMap.uiSettings
                //设置罗盘
                uiSettings.setCompassMargins(0, AutoSizeUtils.mm2px(this, 120f), AutoSizeUtils.mm2px(this, 40f), 0)
                //设置本地化(需要添加本地化插件： implementation 'com.mapbox.mapboxsdk:mapbox-android-plugin-localization-v9:0.12.0')
                val localizationPlugin = LocalizationPlugin(Activity_Main_Map, mapboxMap, style)
                //将地图与设备语言匹配
                localizationPlugin.setMapLanguage(MapLocale.SIMPLIFIED_CHINESE)
                // 设置位置
                // 自定义位置的显示
                val customLocationComponentOptions = LocationComponentOptions.builder(this)
                    .trackingGesturesManagement(true)
                    .accuracyColor(ContextCompat.getColor(this, R.color.mapboxGreen))
                    .build()
    
                val locationComponentActivationOptions = LocationComponentActivationOptions
                    .builder(this, style)
                    .locationComponentOptions(customLocationComponentOptions)
                    .useDefaultLocationEngine(true)
                    .build()
    
                this.mLocationComponent = mapboxMap.locationComponent
    
                // Get an instance of the LocationComponent and then adjust its settings
                // 获取LocationComponent的实例，然后调整其设置
                this.mLocationComponent.apply {
                    // Activate the LocationComponent with options
                    // 使用选项激活LocationComponent
                    activateLocationComponent(locationComponentActivationOptions)
    
                    // Enable to make the LocationComponent visible
                    // 启用以使LocationComponent可见
                    isLocationComponentEnabled = true
    
                    // Set the LocationComponent's camera mode
                    // 设置LocationComponent的摄像头模式
                    cameraMode = CameraMode.TRACKING_COMPASS
    
                    // Set the LocationComponent's render mode
                    // 设置LocationComponent的渲染模式
                    renderMode = RenderMode.COMPASS
            }
        }
    ```
---
#### 一些其他操作
- 将地图源替换为天地图的源
    
    ```
    1.先创建天地图账号，获取token
    2.在assets文件中创建一个json文件：
        {
          "version": 8,
          "sources": {
            //矢量底图源
            "URL_VECTOR_2000": {
              "tiles": ["http://t0.tianditu.gov.cn/vec_w/wmts?tk=你的token&SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&FORMAT=tiles"],
              "type": "raster",
              "tileSize": 256
            },
            //矢量注记源
            "URL_VECTOR_ANNOTATION_CHINESE_2000": {
              "tiles": ["http://t0.tianditu.gov.cn/cva_w/DataServer?T=cva_w&X={x}&Y={y}&L={z}&tk=你的token"],
              "type": "raster",
              "tileSize": 256
            }
          },
          "layers": [
            {
              "id": "URL_VECTOR_2000",
              "type": "raster",
              "source": "URL_VECTOR_2000",
              "maxzoom": 22
            },
            {
              "id": "URL_VECTOR_ANNOTATION_CHINESE_2000",
              "type": "raster",
              "source": "URL_VECTOR_ANNOTATION_CHINESE_2000",
              "maxzoom": 22
            }
          ]
        }
    3.设置MapBox的Style：
        mMapBoxMap.setStyle(Style.Builder().fromUri("asset://Json文件"))
    ```
- 创建一个Marker（标记点）

    ```
        //所有的标注，建筑等等都是添加到Layer里面
        mMapBoxMap.getStyle {
            if (it.getLayer("Home_Layer") != null) {
                return@getStyle
            }
            //添加icon
            it.addImage("marker_icon", BitmapFactory.decodeResource(resources, R.drawable.red_marker))
            //设置数据（GeoJson数据）
            it.addSource(getMarkerSource())
            //添加图层
            it.addLayer(SymbolLayer("Home_Layer", "Home")
                //设置属性
                .withProperties(
                    PropertyFactory.iconImage("marker_icon"),
                    //如果为true，则即使其他符号与图标碰撞也可以看到。（机翻，大概就这意思）
                    PropertyFactory.iconIgnorePlacement(true),
                    //如果为true，则即使该图标与其他先前绘制的符号发生冲突也将是可见的。（机翻，大概就这意思）
                    PropertyFactory.iconAllowOverlap(true)
                ))
        }
        
        /**
         * 指定坐标数据
         */
        fun getMarkerSource(): GeoJsonSource {
            val lat = 25.02365687211753
            val lng = 102.73601658370522
            //构建了一个点的数据
            return GeoJsonSource("Home", Feature.fromGeometry(Point.fromLngLat(lng, lat)))
        }
    ```

- 设置点击事件（比如给Marker设置一个点击后移动到该Marker位置的事件）
    ```
        //设置点击事件
        mMapBoxMap.addOnMapClickListener(this)
        
        override fun onMapClick(point: LatLng): Boolean {
            return handleClickIcon(mMapBoxMap.projection.toScreenLocation(point))
        }
        
        /**
         * 处理点击事件
         * 大概逻辑就是，当点击地图某个点时，检索有没有符合的Feature，有的话处理返回false
         */
        private fun handleClickIcon(screenPoint: PointF): Boolean {
            val features: List<Feature> = mMapBoxMap.queryRenderedFeatures(
                screenPoint,
                "Home_Layer"
            )
            return if (features.isNotEmpty()) {
                features[0].geometry()?.let {
                    if (it.type() == "Point") {
                        val point = it as Point
                        movePosition(LatLng(point.latitude(), point.longitude()), 18.0, 180.0, 30.0)
                    }
                }
                true
            } else {
                false
            }
        }
    ```
---
#### 一些问题
- 地图加载不出来咋办？

    ```
        //使用这个方法，地图加载失败的提示
        Activity_Main_Map.addOnDidFailLoadingMapListener {
            LogUtils.eTag("Angki", it)
        }
    ```
- 经纬度不对？
    
    ```
    MapBox默认使用的WGS84坐标系，而国内使用的是GCJ-02。所以在把地图源替换为天地图源后坐标会偏移。可以使用MapBox提供的中国插件，把MapView替换为ChinaMapView。就是这个插件需要特殊的令牌，估计是要购买。
    ```

- 总结下？

    ```
    MapBox官方文档齐全，但是就不要看国内的那个官方文档，太老了。然后地图的一些显示和效果也都有例子。建议把官方demo跑一下，就能知道可以做些什么。
    ```
