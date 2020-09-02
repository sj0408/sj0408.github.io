---
layout: post
title: "Folium으로 Raster 파일 지도에 올리기"
tags: [Folium, Image, Raster, Reprojection, Mapbox]
comments: true
---
# Folium

1. Folium이란
    - Folium은 leaflet.js를 기반으로 만들어진 Python 라이브러리로 데이터를 시각화하여 leaflet 지도 위에 표출하는 것을 도와준다.
    - OpenStreetMap, Mapbox, Stamen 등 다양한 built-in tileset이 있으며, Mapbox 혹은 Cloudmade API를 이용한 custom tileset도 이용할 수 있다.
    - Folium의 자세한 내용은 [Folium Documentation](https://python-visualization.github.io/folium/)에서 확인할 수 있다.


```python
!pip install folium
!pip install rasterio
```

    Collecting folium
      Downloading folium-0.11.0-py2.py3-none-any.whl (93 kB)
    Requirement already satisfied: jinja2>=2.9 in c:\users\mardi\anaconda3\lib\site-packages (from folium) (2.11.1)
    Requirement already satisfied: requests in c:\users\mardi\anaconda3\lib\site-packages (from folium) (2.22.0)
    Collecting branca>=0.3.0
      Downloading branca-0.4.1-py3-none-any.whl (24 kB)
    Requirement already satisfied: numpy in c:\users\mardi\anaconda3\lib\site-packages (from folium) (1.18.1)
    Requirement already satisfied: MarkupSafe>=0.23 in c:\users\mardi\anaconda3\lib\site-packages (from jinja2>=2.9->folium) (1.1.1)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in c:\users\mardi\anaconda3\lib\site-packages (from requests->folium) (1.25.8)
    Requirement already satisfied: idna<2.9,>=2.5 in c:\users\mardi\anaconda3\lib\site-packages (from requests->folium) (2.8)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in c:\users\mardi\anaconda3\lib\site-packages (from requests->folium) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in c:\users\mardi\anaconda3\lib\site-packages (from requests->folium) (2019.11.28)
    Installing collected packages: branca, folium
    Successfully installed branca-0.4.1 folium-0.11.0
    

    WARNING: You are using pip version 20.1.1; however, version 20.2.2 is available.
    You should consider upgrading via the 'c:\users\mardi\anaconda3\python.exe -m pip install --upgrade pip' command.
    


```python
import folium
```

- location: [위도, 경도]
- zoom_start: 기본 확대값(높을수록 확대), int


```python
folium_map = folium.Map(location=[37.392075,126.958980], zoom_start=15)
folium_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF84MThiNzY2ODBkNWE0MzMzODdlZDMxMzI5ODgzOTA4OSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfODE4Yjc2NjgwZDVhNDMzMzg3ZWQzMTMyOTg4MzkwODkiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzgxOGI3NjY4MGQ1YTQzMzM4N2VkMzEzMjk4ODM5MDg5ID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzgxOGI3NjY4MGQ1YTQzMzM4N2VkMzEzMjk4ODM5MDg5IiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFszNy4zOTIwNzUsIDEyNi45NTg5OF0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxNSwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzIzNjNmZTM5ZTkyMTQ1ZDc5ZmVlOWQ5NmFiNzViOTg2ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF84MThiNzY2ODBkNWE0MzMzODdlZDMxMzI5ODgzOTA4OSk7CiAgICAgICAgCjwvc2NyaXB0Pg== onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



- tiles: 다양한 built-in tileset을 활용할 수 있지만 그닥...


```python
folium_stamen = folium.Map(location=[37.392075,126.958980], zoom_start=15, tiles='Stamen Toner')
folium_stamen
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF85MzdkODBmZDkxMWY0YTQ4ODM1MGViNzdlNDUzZTlkMiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfOTM3ZDgwZmQ5MTFmNGE0ODgzNTBlYjc3ZTQ1M2U5ZDIiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzkzN2Q4MGZkOTExZjRhNDg4MzUwZWI3N2U0NTNlOWQyID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzkzN2Q4MGZkOTExZjRhNDg4MzUwZWI3N2U0NTNlOWQyIiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFszNy4zOTIwNzUsIDEyNi45NTg5OF0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxNSwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzEwZWRlMWRhMmMwNzRhNmJhMjg2OGIzMzRhOGM5NTY0ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly9zdGFtZW4tdGlsZXMte3N9LmEuc3NsLmZhc3RseS5uZXQvdG9uZXIve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiTWFwIHRpbGVzIGJ5IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9zdGFtZW4uY29tXCJcdTAwM2VTdGFtZW4gRGVzaWduXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9jcmVhdGl2ZWNvbW1vbnMub3JnL2xpY2Vuc2VzL2J5LzMuMFwiXHUwMDNlQ0MgQlkgMy4wXHUwMDNjL2FcdTAwM2UuIERhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzkzN2Q4MGZkOTExZjRhNDg4MzUwZWI3N2U0NTNlOWQyKTsKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



2. Folium 위에 Raster 맴핑하기
    - 지도 상에 raster 파일을 맴핑은 간단하다. Reprojection 과정을 거친 후 raster layer를 지도 위에 올리면 끝!
    - Import  
        - 
    ```python
    import folium
    from rasterio.warp import calculate_default_transform, reproject, Resampling
    ```
    
    - Reprojection
        - [이 분의 말](https://stackoverflow.com/questions/57376512/which-projection-is-mapbox-using)에 따르면 mapbox의 맴핑 라이브러리는 EPSG:3857을 사용하지만 마커나 GeoJSON layer와 같은 정보를 지도에 올리기 위해서는 EPSG:4326 좌표계를 사용해야 된다고 함.
    ```python
    input_path = <input_path>
    out_path = <out_path>
    # Destination Coordinate Reference System - mapbox에서 사용하는 좌표계: epsg:4326
    dst_crs = 'epsg:4326' 
    ```
        - [rasterio.warp](https://rasterio.readthedocs.io/en/latest/api/rasterio.warp.html)의 <span style='background :yellow' > calculate_default_transform </span> 모듈은 reprojection에 필요한 output의 transformation matrix과 dimension을 계산해준다.
    ```python
    with rasterio.open(input_path) as src:
            _transform, width, height = calculate_default_transform(src.crs, dst_crs, src.width, src.height, *src.bounds)
    ```        
        - 위에서 얻은 정보들을 이용해 raster의 meta 정보를 업데이트 한다 
    ```python
            kwargs = src.meta.copy()
            kwargs.update({
                'crs': dst_crs,
                'transform': _transform,
                'width': width,
                'height': height
            })
    ```     
        - output될 raster의 정보를 band 별로 입력해준다. 
        - 본 연구에 활용된 raster는 b,g,r와 transparency까지 총 4개의 band로 구성돼 있다.
        
        ```python
                with rasterio.open(out_path, 'w', **kwargs) as dst:
                        for i in range(1, src.count + 1):
                            reproject(
                            source=rasterio.band(src, i),
                            destination=rasterio.band(dst, i),
                            src_transform=src.transform,
                            src_crs=src.crs,
                            dst_transform=_transform,
                            dst_crs=dst_crs)  
        ```
    
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
        - 마지막으로 folium map에 custom mapbox를 이용해 raster layer를 추가하는 과정을 거친다. custom mapbox를 쓰기 위해서는 mapbox api token이 필요하며 tiles 파라미터에 token이 포함된 tile url을 넣어주어야함. token 생성 방법은 [이곳](https://docs.mapbox.com/help/tutorials/get-started-tokens-api/)을 참고.
        
        ```python
        token = 'pk.eyJ1IjoiamF5LWsiLCJhIjoiY2tkNWh4bmI4MDBnNDJ4bWI5ZmNhcDV4ayJ9.VzGXhHv2Bvi3aaL2ajBUBw'
        tileurl = 'https://api.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}@2x.png?access_token=' + str(token)

        m = folium.Map(location=[36.8913652,127.8232119], zoom_start=15, tiles=tileurl, attr='Mapbox')
        m.add_child(folium.raster_layers.ImageOverlay(img, opacity=1, bounds=mask_bounds))        
        ```
        
    







```python

```
