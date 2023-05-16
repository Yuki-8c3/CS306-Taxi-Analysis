<template>

  <div class="home">
    <div id="map" ref="map"></div> <!-- 地图 -->
    <div id="info"></div>  <!--  坐标提示框 -->
  </div>
</template>

<script>
// import url('https://api.mapbox.com/mapbox-gl-js/v0.49.0/mapbox-gl.css');
import mapboxgl from "mapbox-gl";
import { MapboxLayer } from "@deck.gl/mapbox";
import { TripsLayer } from "@deck.gl/geo-layers";

export default {
  name: "Home",

  data() {
    return {
      map: "", //地图
      //主题
      theme: {
        buildingColor: [74, 80, 87],
        trailColor0: [253, 128, 93],
        trailColor1: [23, 184, 190],
        material: {
          ambient: 0.1, //环境
          diffuse: 0.6, //扩散
          shininess: 32, //亮度
          specularColor: [60, 64, 70],
        },
      },
      time: 0, //时间
      loopLength: 1800, // 线圈长度
      animationSpeed: 1, //动画速度
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    // 初始化底层地图
    initMap() {
      this.map = new mapboxgl.Map({
        accessToken: "自己在mapBox上申请的access token",
        container: this.$refs.map,
        style: "https://basemaps.cartocdn.com/gl/dark-matter-nolabels-gl-style/style.json",
        // center: [114.098549, 22.573233], //深圳随便一个地方
        center: [-74, 40.72], //曼哈顿（样例的）
        zoom: 13,
        pitch: 45,
        bearing: 0,
        antialias: true, // 平滑曲线
      });
      // 添加图层
      this.addLayers();

      // 点击显示当前位置坐标
      this.map.on("click", (e) => {
        document.getElementById("info").innerHTML ='当前位置：'+JSON.stringify(e.lngLat.wrap());
        document.getElementById("info").style.display="table"
      });
    },
    // 添加图层
    addLayers() {
      // 线路层
      let myDeckLayer = new MapboxLayer({
        id: "trips",
        type: TripsLayer,
        data: "https://raw.githubusercontent.com/visgl/deck.gl-data/master/examples/trips/trips-v7.json",
        // data: "/deskGL-trips-layer.json",
        getPath: (d) => d.path,
        getTimestamps: (d) => d.timestamps, //返回一个时间戳数组，对应于返回的路径的每个导航点，表示访问该点的时间
        getColor: (d) =>
            // d.is_passenger === 0 ? this.theme.trailColor0 : this.theme.trailColor1,
            d.vendor === 0 ? this.theme.trailColor0 : this.theme.trailColor1,
        opacity: 0.3,
        widthMinPixels: 5, // 轨迹的宽度
        capRounded: true,
        // fadeTrail:false  //路径是否淡出
        trailLength: 120, //路径完全淡出需要多长时间
        currentTime: 0, //帧的当前时间，即动画的播放头;此值应与 timestamps 中的时间戳单位相同
        shadowEnabled: false,
      });

      this.map.on("load", () => {
        // 添加myDeckLayer图层
        this.map.addLayer(myDeckLayer);
        // 每50ms更新一下时间，形成动画
        setInterval(() => {
          myDeckLayer.setProps({
            currentTime: this.time,
          });
        }, 50);
        window.requestAnimationFrame(this.animate);
      });
    },
    // 动画
    animate() {
      this.time = (this.time + this.animationSpeed) % this.loopLength;
      window.requestAnimationFrame(this.animate);
    },
  },
};
</script>

<style scoped>
/*// 导入mapbox样式*/

#map {
  width: 1400px;
  height: 800px;
  background: #e5e9ec;
}
#info {
  display: none;
  position: absolute;
  top: 90px;
  margin: 0px auto;
  word-wrap: anywhere;
  white-space: pre-wrap;
  padding: 10px;
  border: none;
  border-radius: 3px;
  font-size: 12px;
  text-align: center;
  color: #222;
  background: #fff;
}
</style>