title: 根据经纬度计算地图的像素值
date: 2016-08-12 17:14:24
categories:
tags: Programming
description:
---
如果我们想根据经纬度在一张地图的图片上画一个点，就需要知道如何根据经纬度算出对应图片的像素值。本文介绍一下怎么根据在线地图中常用的Web Mercator投射算法来从经纬度计算像素值。

[Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator)投射算法最早是Google地图在2005年使用的，随后就成了在线地图的实际标准，Google地图，必应Bing地图用的都是Web Mercator投射算法。Web Mercator投射算法如下公式所示：

![Web Mercator算法](https://wikimedia.org/api/rest_v1/media/math/render/svg/3e5240c35bde180226b268aa4de4e4cde3b95ed3)

其中λ是经度（longitude），ψ是维度（latitude）。zoom level就是放大的倍数，在Google地图中可以在地址栏看到这个值，比如下面是北京的地址，其中4z就表明现在的zoom level是4。

`https://www.google.com/maps/place/Beijing,+China/@39.9385809,115.835544,4z`

直接套用上面的公式就能算出给定经纬度的地点对应的像素。[Google Maps APIs](https://developers.google.com/maps/documentation/javascript/examples/map-coordinates)上的示例代码也展示了怎么根据经纬度计算像素值，用的是芝加哥的地址。

但是通常我们的图片只包含世界地图的一小块区域，这个时候如果想在这个地图上标出经纬度对应的点，需要算出这个点对应的像素值。然后在算出图片左上角的那个点对应的像素值，一减就是在这个图片上对应的位置了。

下面的代码是根据Google Maps APIs上的示例程序简单修改的计算地图图片中相对坐标的代码。

```javascript
// below function is from: https://developers.google.com/maps/documentation/javascript/examples/map-coordinates
var TILE_SIZE = 256;
function project(lat, lng, zoom) {
    var siny = Math.sin(lat * Math.PI / 180);

    // Truncating to 0.9999 effectively limits latitude to 89.189. This is
    // about a third of a tile past the edge of the world tile.
    siny = Math.min(Math.max(siny, -0.9999), 0.9999);

    return {
        X: TILE_SIZE * (0.5 + lng / 360) * (1 << zoom),
        Y: TILE_SIZE * (0.5 - Math.log((1 + siny) / (1 - siny)) / (4 * Math.PI)) * (1 << zoom)
    };
}

function getPosition(lat, lng) {
    // top left point of the map picture we are using
    var STARTLAT = 40;
    var STARTLNG = 10;

    // zoom level of the map picture
    var zoom = 4;

    var start = project(STARTLAT, STARTLNG, zoom);
    var p = project(lat, lng, zoom);

    return {
        X: p.X - start.X,
        Y: p.Y - start.Y
    };
}
```