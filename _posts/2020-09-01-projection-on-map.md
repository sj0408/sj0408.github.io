---
layout: post
title: "Folium으로 Raster 파일 지도에 올리기"
tags: [Folium, Image, Raster, Reprojection, Mapbox]
comments: true
---
# Folium
<br>

1. Folium이란
    - Folium은 leaflet.js를 기반으로 만들어진 Python 라이브러리로 데이터를 시각화하여 leaflet 지도 위에 표출하는 것을 도와준다.
    - OpenStreetMap, Mapbox, Stamen 등 다양한 built-in tileset이 있으며, Mapbox 혹은 Cloudmade API를 이용한 custom tileset도 이용할 수 있다.
    - Folium의 자세한 내용은 [Folium Documentation](https://python-visualization.github.io/folium/)에서 확인할 수 있다.


```python
# install
!pip install folium
!pip install rasterio
```


```python
# import
import folium
```
<br>

- location: [위도, 경도]
- zoom_start: 기본 확대값(높을수록 확대), int

```python
folium_map = folium.Map(location=[37.392075,126.958980], zoom_start=15)
folium_map
```
<iframe src="/images/folium_map.html" width="700" height="500" frameborder="0" style="border:0" allowfullscreen></iframe>
<br>

- tiles: 다양한 built-in tileset을 활용할 수 있지만 그닥 예뻐보이진 않는다...
```python
folium_stamen = folium.Map(location=[37.392075,126.958980], zoom_start=15, tiles='Stamen Toner')
folium_stamen
```
<iframe src="/images/folium_stamen.html" width="700" height="500" frameborder="0" style="border:0" allowfullscreen></iframe>
<br>


2. Folium 위에 Raster 맵핑하기
    - 지도 상에 raster 파일을 맵핑은 간단하다. Reprojection 과정을 거친 후 raster layer를 지도 위에 올리면 끝!
<br>
    - Import  
        - 
    ```python
    from rasterio.warp import calculate_default_transform, reproject, Resampling
    ```
<br>

    
    - Reprojection
        - [이 분의 말](https://stackoverflow.com/questions/57376512/which-projection-is-mapbox-using)에 따르면 mapbox의 맵핑 라이브러리는 EPSG:3857을 사용하지만 마커나 GeoJSON layer와 같은 정보를 지도에 올리기 위해서는 EPSG:4326 좌표계를 사용해야 된다고 함.
    ```python
    input_path = <input_path>
    out_path = <out_path>
    # Destination Coordinate Reference System - mapbox에서 사용하는 좌표계: epsg:4326
    dst_crs = 'epsg:4326' 
    ```
<br>

        - [rasterio.warp](https://rasterio.readthedocs.io/en/latest/api/rasterio.warp.html)의 <span style='background :yellow' > calculate_default_transform </span> 모듈은 reprojection에 필요한 output의 transformation matrix과 dimension을 계산해준다.
    ```python
    # calculate default transform for reprojection
    with rasterio.open(input_path) as src:
            _transform, width, height = calculate_default_transform(src.crs, dst_crs, src.width, src.height, *src.bounds)
    ```      
<br>

        - 위에서 얻은 정보들을 이용해 raster의 meta 정보를 업데이트 한다 
    ```python
            kwargs = src.meta.copy() 
            kwargs.update({     # meta 정보 업데이트
                'crs': dst_crs,
                'transform': _transform,
                'width': width,
                'height': height
            })
    ```     
<br>

        - output될 raster의 정보를 band 별로 입력해준다. 본 연구에 활용된 raster는 b,g,r와 transparency까지 총 4개의 band로 구성돼 있다.
        
        ```python
                with rasterio.open(out_path, 'w', **kwargs) as dst:
                        for i in range(1, src.count + 1): # band별 정보 입력
                            reproject(
                            source=rasterio.band(src, i),
                            destination=rasterio.band(dst, i),
                            src_transform=src.transform,
                            src_crs=src.crs,
                            dst_transform=_transform,
                            dst_crs=dst_crs)  
        ```
<br>

    
    - Mapping            
        - reprojection 된 raster를 읽어 bound(좌상, 우하 좌표)와 raster 벡터를 얻는다. 이때 raster 벡터의 형상을  (bands, rows, columns)에서 (rows, columns, bands)로 변환한다.
        ```python
        with rasterio.open(out_path) as src:
            boundary = src.bounds
            mask_bounds = [[boundary[1],boundary[0]],[boundary[3],boundary[2]]]

            img = src.read()
            img_n = np.zeros((img.shape[1],img.shape[2],img.shape[0]))
            for i in range(4):
                img_n[:,:,i] = img[i,:,:]
        ```
<br>

        - 마지막으로 folium map에 custom mapbox를 이용해 raster layer를 추가하는 과정을 거친다. custom mapbox를 쓰기 위해서는 mapbox api token이 필요하며 tiles 파라미터에 token이 포함된 tile url을 넣어주어야함. token 생성 방법은 [이곳](https://docs.mapbox.com/help/tutorials/get-started-tokens-api/)을 참고.
        
        ```python
        token = 'pk.eyJ1IjoiamF5LWsiLCJhIjoiY2tkNWh4bmI4MDBnNDJ4bWI5ZmNhcDV4ayJ9.VzGXhHv2Bvi3aaL2ajBUBw'
        tileurl = 'https://api.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}@2x.png?access_token=' + str(token)

        m = folium.Map(location=[36.8913652,127.8232119], zoom_start=15, tiles=tileurl, attr='Mapbox')
        m.add_child(folium.raster_layers.ImageOverlay(img, opacity=1, bounds=mask_bounds))        
        ```
