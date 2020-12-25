<template>
  <div>
    <div class="m-map">
      <div id="js-container" class="map">正在加载数据 ...</div>
    </div>
    <div class="input-card" style="width: 24rem;">
      <div class="input-item">
        <input type="radio" name="func" value="rectangle" /><span
          class="input-text"
          >画矩形生成下载代码</span
        >
      </div>
      <div class="input-item">
        <label for="zoom">地图级别：</label><input type="number" id="zoom" style="width: 200px" value="15" min="1" max="18"/>
        <input id="clear" type="button" class="btn" value="清除矩形" />
        <input id="close" type="button" class="btn" value="关闭绘图" />
      </div>
    </div>
  </div>
</template>

<script>
import remoteLoad from "@/utils/remoteLoad.js";
import { MapKey, MapCityName } from "@/config/config";
import App from "../App.vue";
export default {
  components: { App },
  props: ["lat", "lng"],
  data() {
    return {
      AMap: null,
    };
  },
  methods: {
    // 实例化地图
    initMap() {
      let AMap = (this.AMap = window.AMap);
      let mapConfig = {
        zoom: 8,
        cityName: MapCityName,
      };
      if (this.lat && this.lng) {
        mapConfig.center = [this.lng, this.lat];
      }
      let map = new AMap.Map("js-container", mapConfig);
      var satelliteLayer = new AMap.TileLayer.Satellite();
      map.add([satelliteLayer]);
      var mouseTool = new AMap.MouseTool(map);
      var overlays = [];
      mouseTool.on("draw", function(e) {
        map.remove(overlays);
        overlays = [];
        overlays.push(e.obj);
      });

      function draw(type) {
        switch (type) {
          case "rectangle": {
            mouseTool.rectangle({
              fillColor: "#00b0ff",
              strokeColor: "#80d8ff",
            });
            break;
          }
        }
      }
      var radios = document.getElementsByName("func");
      for (var i = 0; i < radios.length; i += 1) {
        radios[i].onchange = function(e) {
          draw(e.target.value);
        };
      }
      draw("marker");

      document.getElementById("clear").onclick = function() {
        map.remove(overlays);
        overlays = [];
      };
      document.getElementById("close").onclick = function() {
        mouseTool.close(true); //关闭，并清除覆盖物
        for (var i = 0; i < radios.length; i += 1) {
          radios[i].checked = false;
        }
      };
    },
  },
  async created() {
    // 已载入高德地图API，则直接初始化地图
    if (window.AMap && window.AMapUI) {
      this.initMap();
      // 未载入高德地图API，则先载入API再初始化
    } else {
      await remoteLoad(`http://webapi.amap.com/maps?v=1.4&key=${MapKey}`);
      await remoteLoad("http://webapi.amap.com/ui/1.0/main.js");
      this.initMap();
    }
  },
};
</script>

<style lang="css">
#js-container {
  height: 100%;
}
.m-map {
  position: absolute;
  width: 100%;
  height: 100%;
}
.input-card {
  position: absolute;
  background-color: white;
  height: 100px;
}
.input-item {
  height: 2.2rem;
}
.btn {
  width: 6rem;
  margin: 0 1rem 0 2rem;
}
.input-text {
  width: 4rem;
  margin-right: 1rem;
}
</style>
