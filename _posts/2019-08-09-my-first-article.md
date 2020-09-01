---
layout: post
title: "Shape 파일을 이용한 TIFF 이미지 마스킹"
tags: [Image, TIFF, Masking, Shape File]
comments: true
---

This is the first article of mardi's blog.

---

## 1. 변경 

# change


## 2. Code Snippets

### Highlighted Code Blocks

```python

print('hello world!')

import smbus
import time
import sys
import pymysql
from sqlalchemy import create_engine
pymysql.install_as_MySQLdb()
import pandas as pd
from datetime import datetime

bus = smbus.SMBus(1)
address = 0x48

if len(sys.argv) == 2:
	i = int(sys.argv[1])
	input_addr = 0x42 + i
else:
	input_addr = 0x42


while True:
	# time record
	t = datetime.now()
	tim = t.strftime("20%y/%m/%d-%H:%M")
	# value record
	bus.write_byte(address, input_addr)

	aread = pd.DataFrame({'value': [bus.read_byte(address)], 'time': [tim]})
	engine = create_engine('mysql://agdatalab_sensor:"#password"@agdatalab-sensor.cr93cx0zvhvo.ap-northeast-2.rds.amazonaws.com/soil_moisture?charset=utf8', encoding = 'utf-8')
	aread.to_sql(name = 'soil_moisture_data', con=engine,  if_exists = 'append', index = False)

	time.sleep(900)

```
