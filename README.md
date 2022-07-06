# the-maximum-river-discharge-channel-of-basins
#the maximum river discharge channel of basins for gis from raster data
from osgeo import gdal, ogr
from osgeo import gdal
import numpy as np
driver = ogr.GetDriverByName('ESRI Shapefile')
shp = "E:/SRTM GL1/出水口/new_layer.shp"
ds = driver.Open(shp, 0)
layer = ds.GetLayer(0)
layer.GetFeatureCount()
xValues = []
yValues = []
station_list = []
layer.ResetReading()
for i in range(layer.GetFeatureCount()):
    feature = layer.GetFeature(i)
    geometry = feature.GetGeometryRef()
    x = geometry.GetX()
    y = geometry.GetY()
    xValues.append(x)
    yValues.append(y)
    station = feature.GetField("id")
    station_list.append(station)

dr = gdal.Open("E:/SRTM GL1/Topo/10e+05/flow_FD.tif")
dr_2 = gdal.Open("E:/SRTM GL1/Topo/10e+05/flow_A.tif")
im_width = dr.RasterXSize
im_height = dr.RasterYSize
im_bands = dr.RasterCount
im_proj = dr.GetProjection()
band=dr.GetRasterBand(1)
band_2=dr_2.GetRasterBand(1)
im_datas=band.ReadAsArray(0,0,im_width,im_height)
im_datas_2=band_2.ReadAsArray(0,0,im_width,im_height)
transform = dr.GetGeoTransform()
x_origin = transform[0]
y_origin = transform[3]
pixel_width = transform[1]
pixel_height = transform[5]
driver = gdal.GetDriverByName("GTiff")
dr_3 = driver.Create("E:/SRTM GL1/python生成文件/fdem_new.tif",im_width,im_height,1,gdal.GDT_Float64)
dr_3.SetGeoTransform([x_origin,pixel_width,0,y_origin,0,pixel_height])
dr_3.SetProjection(im_proj)
band_3=dr_3.GetRasterBand(1)
im_datas_3=band_3.ReadAsArray(0,0,im_width,im_height)
data_3= np.array([[0,0,0], [0,0,0],[0,0,0]])
for i in range(len(station_list)):
    x = xValues[i]
    y = yValues[i]
    x_offset = int((x - x_origin) / pixel_width)
    y_offset = int((y - y_origin) / pixel_height)
    im_datas_3[y_offset,x_offset]=im_datas_2[y_offset,x_offset]
    while 1:
        data = im_datas[y_offset-1:y_offset+2,x_offset-1:x_offset+2]
        data_2 = im_datas_2[y_offset-1:y_offset+2,x_offset-1:x_offset+2]
        data_3[0,0]=((data[0,0]==2)*data_2[0,0])
        data_3[0,1]=((data[0,1]==4)*data_2[0,1])
        data_3[0,2]=((data[0,2]==8)*data_2[0,2])
        data_3[1,0]=((data[1,0]==1)*data_2[1,0])
        data_3[1,1]=0
        data_3[1,2]=((data[1,2]==16)*data_2[1,2])
        data_3[2,0]=((data[2,0]==128)*data_2[2,0])
        data_3[2,1]=((data[2,1]==64)*data_2[2,1])
        data_3[2,2]=((data[2,2]==32)*data_2[2,2])
        i=np.argmax(data_3)
        a=int(i/3)
        b=(i+1)-3*a-1
        if np.max(data_3) == 0:
            break
        im_datas_3[a-1+y_offset,b-1+x_offset]=im_datas_2[a-1+y_offset,b-1+x_offset]
        x_offset=b-1+x_offset
        y_offset=a-1+y_offset

dr_3.GetRasterBand(1).WriteArray(im_datas_3)
del dr_3
