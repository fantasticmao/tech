# 空间数据模型

## OGC 几何图形

开放地理空间联盟（Open-Geospatial-Consortium，OGC）开发了简单功能访问（Simple-Features-Access，SFA）标准，提供了一种地理空间的数据模型。它定义了几何的基本空间类型，以及操作这些几何值以执行空间分析任务的操作。PostGIS 将 OGC 几何模型实现为 PostgreSQL 数据类型的 [geometry](https://postgis.net/docs/manual-3.5/using_postgis_dbmanagement.html#PostGIS_Geometry) 和 [geography](https://postgis.net/docs/manual-3.5/using_postgis_dbmanagement.html#PostGIS_Geography)。

几何是一种抽象的数据类型，几何数据值是其具体的子类型之一。子类型表示各种形状和各种维度的几何形状，包括基本类型的点 Point、线 LineString、线性环 LineRing、多边形 Polygon，以及集合类型的多点 MultiPoint、多线 MultiLineString、多面 MultiPolygon、几何对象几何 GeometryCollection。

几何值与一个 **空间参考系统 Spatial Reference System** 相关联，指示其嵌入的坐标系。空间参考系统由 SRID 编号标识，X 轴和 Y 轴的单位由空间参考系统决定。在平面参考系统中，X 和 Y 坐标通常代表东向和北向，而在大地测量系统中，它们代表经度和纬度。SRID 0 表示无限笛卡尔平面，其轴上没有分配单位。

## WKT 和 WKB

OGC SFA 规范定义了两种供外部使用的几何值表示格式：**Well-Known Text WKT** 和 **Well-Known Binary WKB**。

WKT：

- `POINT(0 0)`
- `POINT Z (0 0 0)`
- `POINT ZM (0 0 0 0)`
- `LINESTRING(0 0,1 1,1 2)`
- `POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))`

WKB：

- WKT: `POINT(1 1)`
- WKB: `0101000000000000000000F03F000000000000F03`
- WKT: `LINESTRING (2 2, 9 9)`
- WKB: `0102000000020000000000000000000040000000000000004000000000000022400000000000002240`

# 几何数据类型

# 地理数据类型

地理坐标是以角度单位（度）表示的球坐标。

由于基础数学更复杂，因此为地理类型定义的函数少于为几何类型定义的函数。随着时间的推移，随着新算法的添加，地理类型的功能将扩展。作为一种解决方法，可以在几何和地理类型之间来回转换。

## 何时使用地理数据类型

地理数据类型允许您将数据存储在经度/纬度坐标中，但有代价：

- 在地理上定义的函数比在几何上定义的函数少；
- 地理数据类型的处理函数，需要更多 CPU 时间来执行。

待选择的数据类型，应由正在构建的应用程序的预期工作区域来确定。例如，数据是跨越全球还是大片大陆区域，还是州、县或直辖市的本地数据。

- 如果数据包含在较小的区域中，可能会发现，就可用的性能和功能而言，选择合适的投影并使用 GEOMETRY 是最佳解决方案；
- 如果数据是整个地球或大陆，将能够构建一个系统，而无需担心地理投影的细节。保存经度/纬度数据并使用地理中定义的函数；
- 如果你不了解投影，并且不想了解它们，并且准备接受地理中可用功能的限制，那么使用地理可能比使用几何更容易。只需将数据加载为经度/纬度，然后从那里开始。

# 空间参考系

SRID: 0，

SRID: 4326，WGS-84，单位是角度

GCJ-02
