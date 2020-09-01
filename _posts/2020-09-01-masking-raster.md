---
layout: post
title: "Shape 파일을 이용한 Raster 마스킹"
tags: [Image, Raster, Masking, Shape File]
comments: true
---

본 글은 [Earth Lab](https://www.earthdatascience.org/)을 참조하여 작성하였습니다.

---

# Raster

<font size="5"> 1. What is Raster </font>
- Raster data는 grid 상에서 같은 크기의 cell들에 저장된 값들로 지도 상에 하나의 픽셀로 표현되며 각 픽셀은 공간정보를 포함한다.  

<img src="https://www.earthdatascience.org/images/earth-analytics/raster-data/raster-concept.png" width="400">

<br>
<font size="5"> 2. Raster in Python </font>
- Python에서 Raster data를 다룰 수 있는 오픈 소스 패키지는 크게 [Python GDAL](https://pypi.org/project/GDAL/)과 [Rasterio](https://pypi.org/project/rasterio/)가 있다.
- **GDAL**은 C++을 베이스로 만들어져 풍부한 기능을 제공하지만 코드가 다소 복잡한 경향이 있다.
- **Rasterio**는 Pythonic한 인터페이스를 가지고 있으며 코드가 간단하고 가독성이 좋다.

<br>
<font size="5"> 3. Raster and TIFF, GeoTIFF </font>
- TIFF  ([참조](https://www.widen.com/blog/whats-the-difference-between-png-jpeg-gif-and-tiff))  
    - 무손실 압축과 태그를 지원하는 이미지 포맷
    - 대용량의 이미지를 그대로 보존해 편집, 저장하기에 적합
    - 압축률이 낮기 때문에 용량이 작은 파일이나 web에서 사용하기에는 부적합  

<img src="https://embed.widencdn.net/img/widen/j7rruxyvml/exact/file-type-comparison-chart.jpeg?keep=c&crop=yes&u=sar69m" width="600">

- GeoTIFF ([참조](https://www.earthdatascience.org/courses/use-data-open-source-python/intro-raster-data-python/fundamentals-raster-data/intro-to-the-geotiff-file-format/))
    - TIFF와 동일한 확장자명을 가지지만 추가적으로 좌표계 정보를 태그로 포함하고 있다.
    - 태그에 포함된 raster 메타데이터  
        - **Spatial Extent**: What area does this dataset cover?
        - **Coordinate reference system**: What spatial projection / coordinate reference system is used to store the data? Will it line up with other data?
        - **Resolution**: The data appears to be in raster format. This means it is composed of pixels. What area on the ground does each pixel cover - i.e. What is its spatial resolution?
        - **nodata value**
        - How many layers are in the .tif file.

<br>

# Masking Raster with Shape File

- rasterio로 raster file을 읽고 shape file의 좌표정보를 이용해 분석에 필요한 필지를 마스킹한다.

<font size="5"> 1. Import </font>

```python
#  IMPORT
import rasterio
from rasterio.mask import mask

import os
import pandas as pd
import geopandas as gpd
import json
```
<br>
<font size="5"> 2. Shape file과 Raster file을 읽고 두 파일의 좌표계 정보를 일치 </font>

```python
# READ SHAPE FILE
shp_path = '<SHAPE FILE PATH>'
geo = gpd.read_file(shp_path)
      
# READ RASTER DATA
data = rasterio.open('<GEO TIFF FILE PATH>')

# CONVERT SHP CRS >> RASTER CRS
# shape file의 좌표계를 raster data의 좌표계로 변환해 두 파일의 좌표계를 일치시킴
geo = geo.to_crs(crs=data.crs.data)
```
<br>
<font size="5"> 3. 마스킹 좌표 설정 / 마스킹된 Raster의 meta data 생성 </font>

- 마스킹을 위해서는 마스킹될 필지의 좌표정보가 필요하다. geopandas로 읽어들인 shape 파일을 json 형태로 변환, 'features' dict 내의 'geometry' dict에 위치한 좌표 정보를 list 로 저장한다.

```python
json.loads(geo.to_json())
```
> ```python
[{'id': '0',
  'type': 'Feature',
  'properties': {'CHG_CFCD': '01',
   'CHG_CFNM': '유지',
   'FMAP_INNB': '11472433',
   'INTPR_CD': '01',
   'INTPR_NM': '논',
   'INVD_CFCD': '01',
   'INVD_CFNM': '항공영상',
   'ITPINP_DE': '20171130',
   'LGL_EMD_CD': '43760410',
   'LGL_EMD_NM': '불정면',
   'LNM': '363-3답',
   'MAPDMC_NO': '36704053',
   'PNU_LNM_CD': '4376041032103630003',
   'RNHST_CD': 'null',
   'RNHST_NM': 'null',
   'VDPT_YR': '2016'},
  'geometry': {'type': 'Polygon',
   'coordinates': [[[272646.86728058296, 474287.4838322025],
     [272647.0151503971, 474288.9742049827],........]]}},........]
     ```
      
```python
# get coordinates of all areas
coords = [item['geometry'] for item in json.loads(geo.to_json())['features']]
coords
```
> ```python
[{'type': 'Polygon',
  'coordinates': [[[273714.5509720673, 476484.08102420584],
    [273680.551745765, 476427.85928484285],.......]]},
 {'type': 'Polygon',
  'coordinates': [[[273740.2657739733, 476467.3282821396],
    [273723.0888637786, 476438.9142906783],.......]]},
 {'type': 'Polygon',
  'coordinates': [[[273521.8847622286, 476610.9490039577],
     [273545.55808547314, 476595.116112523],.......]]},.......]
     ```
- 마스킹된 raster 파일의 meta data에 입력할 좌표계 정보를 proj4 형태로 획득
- 좌표계 표현 방법인 epsg와 proj4에 대한 설명은 [여기](https://www.earthdatascience.org/courses/use-data-open-source-python/intro-vector-data-python/spatial-data-vector-shapefiles/epsg-proj4-coordinate-reference-system-formats-python/)에 잘 설명돼 있으며 대한민국 epsg코드 및 좌표계에 대한 정보는 [이곳](http://www.gisdeveloper.co.kr/?p=8942)을 참고하자
```python
# GET EPSG CODE
epsg_code = int(data.crs.data['init'][5:])
# EPSG TO PROJ4
epsg2proj =  CRS.from_epsg(epsg_code).to_proj4()
```
<br>
<font size="5"> 4. 마스킹 </font>
- shape 파일을 pandas dataframe으로 변환한 변수 <span style='background :gray' > geo </span>의 행은 각 필지에 대한 정보를 가지고 있으며 <span style='background :gray' > FMAP_INNB </span>  은 각 필지의 고유한 코드이다.
- 대상 필지 코드 리스트 <span style='background :gray' > soybean </span>에 포함된 행에 대해서만 마스킹을 수행한다.
- [rasterio.mask](https://rasterio.readthedocs.io/en/latest/api/rasterio.mask.html)를 이용해 마스킹


```python
for row in geo.iterrows(): # iter rows of geo
    if row[1].FMAP_INNB in soybean: # if FMAP_INNB code is in [soybean]
        try:
            # masking (rasterio.mask)
            out_img, out_transform = mask(dataset=data, shapes=[coords[row[0]]], crop=True)
            # create meta data for masked raster
            out_meta = data.meta.copy()
            out_meta.update({"driver": "GTiff",
                     "height": out_img.shape[1],
                     "width": out_img.shape[2],
                     "transform": out_transform,
                     "crs": epsg2proj}
                             )

            # create file if not exsist
            fileName = f"D://노지항공영상//괴산//{date}_{time}//mask_shp_{img_type}_soybean"
            if not os.path.exists(fileName):
                try:
                    os.makedirs(fileName)
                except OSError as exc: # Guard against race condition
                    if exc.errno != errno.EEXIST:
                        raise

            # output raster
            with rasterio.open(fileName + f"//{img_type}_shp_mask_{row[1]['FMAP_INNB']}.tif", "w", **out_meta) as dest:
                dest.write(out_img)

        except ValueError:
            pass
```
<br>
<img src="./images/masked_1.tif" width="400">
<img src="./images/masked_2.tif" width="400">
<img src="./images/masked_3.tif" width="400">
<br>

- 각각의 필지가 아닌 다수 혹은 전체 필지를 마스킹하고 싶다면 for문을 사용하지 않고 <span style='background :gray' > coords </span>에 마스킹하려는 모든 필지에 대한 좌표 정보를 추가하면 된다.
- 이때 <span style='background :gray' > mask </span>  모듈의 crop 파라미터를 True로 설정할 경우 선택한 필지를 나타낼 수 있는 최소한의 크기로 raster를 잘라내고 False로 설정할 경우 원래 raster 크기 그대로 마스크된 raster를 출력한다. 

```python
# get coordinates of target areas only
coords = [item['geometry'] for item in json.loads(geo.to_json())['features'] if item['properties']['FMAP_INNB'] in soybean]
epsg_code = int(data.crs.data['init'][5:])
epsg2proj =  CRS.from_epsg(epsg_code).to_proj4()

# masking (rasterio.mask)
out_img, out_transform = mask(dataset=data, shapes=coords, crop=False) # overall ver
# create meta data for masked raster
out_meta = data.meta.copy()
out_meta.update({"driver": "GTiff",
         "height": out_img.shape[1],
         "width": out_img.shape[2],
         "transform": out_transform,
         "crs": epsg2proj}
                 )    
```

<br>

# Module

```python
# 개별 필지 mask version
def maskTifShape(date, time, img_type, soybean):
    # READ SHAPE FILE
    shp_path = '<SHAPE FILE PATH>'
    geo = gpd.read_file(shp_path)

    # READ RASTER DATA
    data = rasterio.open('<GEO TIFF FILE PATH>')

    # CONVERT SHP CRS >> RASTER CRS
    # shape file의 좌표계를 raster data의 좌표계로 변환해 두 파일의 좌표계를 일치시킴
    geo = geo.to_crs(crs=data.crs.data)

    # get coordinates of all areas
    coords = [item['geometry'] for item in json.loads(geo.to_json())['features']]
    epsg_code = int(data.crs.data['init'][5:]) # GET EPSG CODE
    epsg2proj =  CRS.from_epsg(epsg_code).to_proj4() # EPSG TO PROJ4

    for row in geo.iterrows(): # iter rows of geo
        if row[1].FMAP_INNB in soybean: # if FMAP_INNB code is in [soybean]
            try:
                # masking (rasterio.mask)
                out_img, out_transform = mask(dataset=data, shapes=[coords[row[0]]], crop=True)
                # create meta data for masked raster
                out_meta = data.meta.copy()
                out_meta.update({"driver": "GTiff",
                         "height": out_img.shape[1],
                         "width": out_img.shape[2],
                         "transform": out_transform,
                         "crs": epsg2proj}
                                 )
                
               # create file if not exsist
                outPath = "<OUTPUT PATH>"

                if not os.path.exists(outPath):
                    try:
                        os.makedirs(outPath)
                    except OSError as exc: # Guard against race condition
                        if exc.errno != errno.EEXIST:
                            raise

                # output raster
                with rasterio.open(outPath + <OUTPUT FILENAME>.tif", "w", **out_meta) as dest:
                    dest.write(out_img)

            except ValueError:
                pass
```


```python
# 전체 필지 mask version
def maskTifShape_soybean(date, time, img_type, soybean):
    # READ SHAPE FILE
    shp_path = '<SHAPE FILE PATH>'
    geo = gpd.read_file(shp_path)

    # READ RASTER DATA
    data = rasterio.open('<GEO TIFF FILE PATH>')

    # CONVERT SHP CRS >> RASTER CRS
    # shape file의 좌표계를 raster data의 좌표계로 변환해 두 파일의 좌표계를 일치시킴
    geo = geo.to_crs(crs=data.crs.data)

    # get coordinates of target areas only
    coords = [item['geometry'] for item in json.loads(geo.to_json())['features'] if item['properties']['FMAP_INNB'] in soybean]
    epsg_code = int(data.crs.data['init'][5:]) # GET EPSG CODE
    epsg2proj =  CRS.from_epsg(epsg_code).to_proj4() # EPSG TO PROJ4
    
    try:
        # masking (rasterio.mask)
        out_img, out_transform = mask(dataset=data, shapes=coords, crop=True) 
        # create meta data for masked raster
        out_meta = data.meta.copy()
        out_meta.update({"driver": "GTiff",
                 "height": out_img.shape[1],
                 "width": out_img.shape[2],
                 "transform": out_transform,
                 "crs": epsg2proj}
                         )    
        
        # create file if not exsist
        outPath = "<OUTPUT PATH>"

        if not os.path.exists(outPath):
            try:
                os.makedirs(outPath)
            except OSError as exc: # Guard against race condition
                if exc.errno != errno.EEXIST:
                    raise
                            
        # output raster
        with rasterio.open(outPath + <OUTPUT FILENAME>.tif", "w", **out_meta) as dest:
                    dest.write(out_img)         

    except ValueError:
        pass
```
