<template>
  <div class="home">
    <div style="width: 100%; height: 100%">
      <div class="map" id="map"></div>
    </div>
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import { Tile as TileLayer } from 'ol/layer'
import View from 'ol/View'
import XYZ from 'ol/source/XYZ'

export default {
  name: 'HomeView',
  components: {},
  data () {
    return {
      mapObj: null,
      mapDom: null,
      mapPointList: [],
      pointLayerSource: null,
      pointLayer: null,
    }
  },
  mounted () {
    this.initMap()
  },
  methods: {
    // 清除地图 某些情况 地图容器会存在两个 导致地图无法正常显示 这个问题折腾了我半天。
    // 找了半天官方貌似也没有提供 对应的 api，自己动手了。
    mapClear () {
      if (this.mapDom) {
        this.mapDom.innerHTML = ''
        this.mapDom = null
      }
    },

    // 初始化地图
    initMap () {
      // 先尝试清除
      // this.mapClear()
      // 获取地图容器
      this.mapDom = document.getElementById('map')

      // 初始化地图配置
      this.mapObj = new Map({
        target: this.mapDom, // 地图容器
        view: new View({
          center: [112.475243, 23.077845], // 地图中心点
          zoom: 13, // 缩放
          projection: 'EPSG:4326', // 坐标系
        }),
      })

      // 添加一个使用离线瓦片地图的层
      const offlineMapLayer = new TileLayer({
        source: new XYZ({
          url: 'http://169.254.231.19:8081' + '/{z}/{x}/{y}.png', // 设置本地离线瓦片所在路径,前面的地址是你输入http-server之后的服务地址
          tileLoadFunction:  (imageTile, src)=> {
            console.log(imageTile,src)
            // 使用滤镜 将白色修改为深色
            let img = new Image()
            // img.crossOrigin = ''
            // 设置图片不从缓存取，从缓存取可能会出现跨域，导致加载失败
            img.setAttribute('crossOrigin', 'anonymous')
            img.onload =  ()=> {
              let canvas = document.createElement('canvas')
              let w = img.width
              let h = img.height
              canvas.width = w
              canvas.height = h
              let context = canvas.getContext('2d')
              context.filter = 'grayscale(98%) invert(100%) sepia(20%) hue-rotate(180deg) saturate(1600%) brightness(80%) contrast(90%)'
              context.drawImage(img, 0, 0, w, h, 0, 0, w, h)
              imageTile.getImage().src = canvas.toDataURL('image/png')
            },
            img.onerror = ()=>{
              imageTile.getImage().src = require('@/assets/logo.png')
            }
            img.src = src
          },
        }),
      })
      // 将图层添加到地图
      this.mapObj.addLayer(offlineMapLayer)

      // 加载地理坐标
      // this.addPoint()
    },
  },
  beforeDestroy () {
    this.mapClear()
  },
}
</script>
<style lang="less">
.map {
  width: 1900px;
  height: 1000px;
  // background-color: red;
}
</style>
