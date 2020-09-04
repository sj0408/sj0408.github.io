---
layout: post
title: "Raster 파일 지도에 올리기(2)"
tags: [gdal, Image, Raster, TWM]
comments: true
---
gdal을 이용해 zoom별 tile을 생성해서 지도에 올려보자!
---
<br>  
<br>  

[지난 포스트](https://sj0408.github.io/projection-on-map/)에서는 folium 라이브러리를 이용해 reprojection한 raster 파일을 지도 위에 표출해보았다. 하지만 folium에서는 raster의 bound를 설정하기 때문에 polygon 형태로 마스킹된 필지의 경우 필지 이외의 부분들이 검은색으로 나타나는 문제가 생겼다. 배경을 투명하게 만들기 위한 시도도 해봤지만 실패... 그래서 대부분의 지도 서비스에서 하듯이 raster 파일을 zoom별 tile로 잘라 지도 상에 표출해보기로 하였다.
<br>  

1. Tiled Web Map(TWM)
<br>  
    - 이렇게 tile을 이용해 표출되는 지도를 [Tiled Web Map(TWM)](https://en.wikipedia.org/wiki/Tiled_web_map)이라고 하는데 zoom 별로 하나의 큰 지도 파일을 사용하던 [Web Map Service(WMS)](https://en.wikipedia.org/wiki/Web_Map_Service)에 비해 zoom이 된 지역에 해당되는 몇장의 tile을 사용하면 되기 때문에 매우 가볍다는 장점이 있다. 때문에 GoogleMap, OpenStreetMap 등 대부분의 지도 API에서 TWM 방식을 사용한다고 한다.
    - [Maptiler](https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection/)라는 지도 API 서비스 사이트에서 z, x, y로 이루어진 tile 좌표계의 작동하는 원리와 지리좌표계(위도, 경도), 투영좌표계(원점으로부터의 거리), pixel 좌표계와의 관계에 대해서 잘 설명하고 있다. 또한 좌표계 간 좌표 변환 코드도 제공하고 있어 다른 좌표계의 데이터를 가지고 있는 경우 활용할 수 있다.
    - tile 좌표계의 z, x, y에 대해서 간단히 말하자면 먼저 z는 확대 정도를 의미하고 0일 때 전세계를 하나의 tile로 표현하고 일반적으로 최대로 22까지 사용한다. x와 y는 쉽게 말해 전체 tile에서 해당 tile이 위치한 열과 행이라고 볼 수 있다. 다만, 아래에서 더 설명하겠지만 행에 해당하는 y값의 경우 Google 형식과 TMS 형식에 따라 차이가 있어 어떤 형식을 사용할 것인지에 따라 맞춰 줄 필요가 있다.
<br>  
<br>  
2. gdal2tiles
<br>  
    - 설명을 읽으면 이래저래 복잡해 보이지만 놀랍게도 gdal 라이브러리를 사용하면 raster를 넣기만 하면 알아서 다 해주는 기적을 경험할 수 있다. gdal에 관해서는 [shape 파일 마스킹 포스트](https://sj0408.github.io/masking-raster/)에서 짧게 언급한 적이 있기 때문에 설명은 생략하겠다. 
    - gdal2tiles[[gdal documenation](https://gdal.org/programs/gdal2tiles.html)], [[PYPI gdal documentation](https://pypi.org/project/gdal2tiles/)] 모듈을 이용하면 몇 줄의 코드로 raster file을 tile로 자를 수 있을뿐더러 동시에 GoogleMap, Leaflet, Openlayer 세 가지 지도 API 기반의 html 파일을 자동으로 생성해주어 매우 유용하다. 
<br>  

    ```python
    # install and import
    !pip install gdal2tiles
    import gdal2tiles

    # path
    in_path = <INPUT PATH>
    out_path = <OUTPUT PATH>

    # gdal to tiles
    zoomMax = 21  # set max zoom
    options = {'zoom': (14, zoomMax), 'resume': True}  # zoom -> 0~22 
    gdal2tiles.generate_tiles(in_path, out_path, **options)
    ```  
<br>   

    위 코드를 실행하면 다음과 같은 구조로 tile이 생성된다.
    ```python
    output path
        |__ 14
            |__ x
                |__ y
                    |__ xxx.tif
                    |__ xxx.tif
        |__ 15
        |__ 16
        .
        .
        |__ 21
        |__ googlemap.html
        |__ leaflet.html
        |__ openlayers.html
    ```  
<br>  
<br>  

이제 leaflet이나 openlayer html 파일을 실행하면...! 지도 위에 raster file이 잘 올라가 있는 것을 확인할 수 있다. 참고로 googlemap은 별도로 발급 받은 api key를 option으로 전달해야 사용 가능하다.
<br>  

[Openlayers html 실행 화면]

<!DOCTYPE html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            <meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no' />
            <title>ndvi.tif</title>

            <!-- Leaflet -->
            <link rel="stylesheet" href="http://cdn.leafletjs.com/leaflet-0.7.5/leaflet.css" />
            <script src="http://cdn.leafletjs.com/leaflet-0.7.5/leaflet.js"></script>

            <style>
                body { margin:0; padding:0; }
                body, table, tr, td, th, div, h1, h2, input { font-family: "Calibri", "Trebuchet MS", "Ubuntu", Serif; font-size: 11pt; }
                #map { position:absolute; top:0; bottom:0; width:100%; } /* full size */
                .ctl {
                    padding: 2px 10px 2px 10px;
                    background: white;
                    background: rgba(255,255,255,0.9);
                    box-shadow: 0 0 15px rgba(0,0,0,0.2);
                    border-radius: 5px;
                    text-align: right;
                }
                .title {
                    font-size: 18pt;
                    font-weight: bold;
                }
                .src {
                    font-size: 10pt;
                }

            </style>

        </head>
        <body>

        <div id="map"></div>

        <script>
        /* **** Leaflet **** */

        // Base layers
        //  .. OpenStreetMap
        var osm = L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'});

        //  .. CartoDB Positron
        var cartodb = L.tileLayer('http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png', {attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, &copy; <a href="http://cartodb.com/attributions">CartoDB</a>'});

        //  .. OSM Toner
        var toner = L.tileLayer('http://{s}.tile.stamen.com/toner/{z}/{x}/{y}.png', {attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStreetMap</a>, under <a href="http://www.openstreetmap.org/copyright">ODbL</a>.'});

        //  .. White background
        var white = L.tileLayer("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAAEAAQMAAABmvDolAAAAA1BMVEX///+nxBvIAAAAH0lEQVQYGe3BAQ0AAADCIPunfg43YAAAAAAAAAAA5wIhAAAB9aK9BAAAAABJRU5ErkJggg==");

        // Overlay layers (TMS)
        var lyr = L.tileLayer('./{z}/{x}/{y}.png', {tms: true, opacity: 0.7, attribution: ""});

        // Map
        var map = L.map('map', {
            center: [36.888860899062195, 127.82339311507181],
            zoom: 16,
            minZoom: 14,
            maxZoom: 16,
            layers: [osm]
        });

        var basemaps = {"OpenStreetMap": osm, "CartoDB Positron": cartodb, "Stamen Toner": toner, "Without background": white}
        var overlaymaps = {"Layer": lyr}

        // Title
        var title = L.control();
        title.onAdd = function(map) {
            this._div = L.DomUtil.create('div', 'ctl title');
            this.update();
            return this._div;
        };
        title.update = function(props) {
            this._div.innerHTML = "ndvi.tif";
        };
        title.addTo(map);

        // Note
        var src = 'Generated by <a href="http://www.klokan.cz/projects/gdal2tiles/">GDAL2Tiles</a>, Copyright &copy; 2008 <a href="http://www.klokan.cz/">Klokan Petr Pridal</a>,  <a href="http://www.gdal.org/">GDAL</a> &amp; <a href="http://www.osgeo.org/">OSGeo</a> <a href="http://code.google.com/soc/">GSoC</a>';
        var title = L.control({position: 'bottomleft'});
        title.onAdd = function(map) {
            this._div = L.DomUtil.create('div', 'ctl src');
            this.update();
            return this._div;
        };
        title.update = function(props) {
            this._div.innerHTML = src;
        };
        title.addTo(map);


        // Add base layers
        L.control.layers(basemaps, overlaymaps, {collapsed: false}).addTo(map);

        // Fit to overlay bounds (SW and NE points with (lat, lon))
        map.fitBounds([[36.88125967695782, 127.83317252619364], [36.896462121166564, 127.81361370395]]);

        </script>

        </body>
        </html>

    
<br>  
test용으로 zoom은 14-17까지만 tile을 만들었다. zoom은 최대 22까지 가능하지만 많은 이미지를 필요로하고 zoom 단계별 이미지들이 나타내는 범위는 다르지만 크기는 256X256으로 고정돼(조정 가능) 있기 때문에 용량을 꽤 많이 차지하게 된다...(정확히 말하자면 zoom을 한 단계 올릴 때마다 용량은 약 4배 증가)
<br>  
<br>  
3. y 좌표 변환
<br>  
위에서 언급했듯이 tile 좌표계에는 Google형식과 TMS형식이 있는데 gdal2tiles의 default는 Google형식에 맞춰져 있다.따라서 Google 형식을 따르도록 돼 있는 Google Map에서는(혹시 Google Map API를 사용하게 된다면) tile을 확인할 수 없다. tile을 표출하기 위해서는 y값에 해당하는 파일명을 Google형식에 맞춰 변환해야한다. 다행히도 변환 코드는 [Maptiler](https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection/)의 <span style='background :yellow' > GlobalMercator </span> 모듈에 있다. 

    ```python
    def GoogleTile(tx, ty, zoom):
            "Converts TMS tile coordinates to Google Tile coordinates"
            # coordinate origin is moved from bottom-left to top-left corner of the extent
            return tx, (2**zoom - 1) - ty
    ```

    ```python
    out_path = <OUTPUT PATH>

    # TMS tile >> Google tile convert
    zoomMax = 21
    for zoom in range(14,zoomMax+1): # z 값
        for col in os.listdir(f"{out_path}//{zoom}"): # x 값
            for tmsRow in os.listdir(f"{out_path}//{zoom}//{col}"): # y 값
                if tmsRow.endswith(".png"):   
                    tmsRow = tmsRow.split('.')[0]
                    googleRow = GoogleTile(int(col), int(tmsRow), int(zoom))[1]

                    # change file names
                    old_file = f"{out_path}//{zoom}//{col}//{tmsRow}.png"
                    new_file = f"{out_path}//{zoom}//{col}//{googleRow}.png"

                    os.rename(old_file, new_file)
    ```