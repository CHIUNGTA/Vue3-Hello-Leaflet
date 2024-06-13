<template>
  <LMap
    v-model:zoom="zoom"
    v-model:center="center"
    ref="mapElement"
    :max-zoom="19"
    :min-zoom="4"
    :options="leafletInitOptions"
    use-global-leaflet
    zoom-animation
    fade-animation
    no-blocking-animation
    style="height: 100vw; width: 100vw"
    @ready="onMapReady"
  >
    <LTileLayer :url="tile.url" :attribution="tile.attribution"></LTileLayer>
  </LMap>
</template>

<script setup>
import { ref, reactive } from "vue";
import { LMap, LTileLayer } from "@vue-leaflet/vue-leaflet";
import L, { featureGroup } from "leaflet";
import "leaflet-draw";
import "leaflet/dist/leaflet.css";
import "leaflet-draw/dist/leaflet.draw.css";

const mapElement = ref(null);
const map = ref(null);
const center = ref([24, 121]);

const zoom = ref(4);

const tile = reactive({
  url: "https://tile.openstreetmap.org/{z}/{x}/{y}.png",
  attribution:
    '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
});

const leafletInitOptions = {
  preferCanvas: true,
};

const drawLayer = featureGroup();
const initialGeoJSON = {
  type: "FeatureCollection",
  features: [
    {
      type: "Feature",
      properties: {},
      geometry: {
        type: "Polygon",
        coordinates: [
          [
            [121.0, 24.0],
            [121.1, 24.0],
            [121.1, 24.1],
            [121.0, 24.1],
            [121.0, 24.0],
          ],
        ],
      },
    },
  ],
};

const onMapReady = (mapInstance) => {
  map.value = mapInstance;

  // 添加绘制图层到地图
  drawLayer.addTo(map.value);
  L.geoJSON(initialGeoJSON).eachLayer((layer) => {
    console.log("-ssssss", layer);
    drawLayer.addLayer(layer);
  });
  // 添加绘制控件
  const drawControl = new L.Control.Draw({
    edit: {
      featureGroup: drawLayer,
    },
    draw: {
      polygon: true,
      polyline: true,
      rectangle: true,
      circle: true,
      marker: true,
    },
  });
  map.value.addControl(drawControl);

  // 捕获绘制事件
  map.value.on(L.Draw.Event.CREATED, (event) => {
    const layer = event.layer;
    drawLayer.addLayer(layer);
    console.log("create", JSON.stringify(drawLayer.toGeoJSON()));
  });

  // 捕获编辑事件
  map.value.on(L.Draw.Event.EDITED, (event) => {
    console.log("edit", JSON.stringify(drawLayer.toGeoJSON()));
  });

  // 捕获删除事件
  map.value.on(L.Draw.Event.DELETED, (event) => {
    console.log("delete", JSON.stringify(drawLayer.toGeoJSON()));
  });
};
</script>
