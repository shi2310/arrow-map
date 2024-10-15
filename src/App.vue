<template>
  <div id="map">
    <div class="btns">
      <p>
        轨迹动画： <button @click="play">播放</button
        ><button @click="pause">{{ pauseRef ? "继续" : "暂停" }}</button>
      </p>
      <p>
        自由绘制： <button @click="drawPolyLine">折线</button
        ><button @click="drawPolygon">多边形</button>
      </p>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onUnmounted } from "vue";
import stop1Png from "@/assets/stop1.png";
import stop2Png from "@/assets/stop2.png";
import MyMap from "arrow-map";
import icon from "@/assets/mShip.png";
import wavePng from "@/assets/wave.png";
import shipImg from "@/assets/ship.png";
import iconPng from "@/assets/icon.png";
import _ from "lodash";
import json from "./source";

// 历史轨迹线点
const historyLines = _.sortBy(
  _.map(json, (o) => ({
    ts: o.ts,
    lat: o.location.coordinates[1],
    lng: o.location.coordinates[0],
  })),
  "ts"
);

const circles = [
  { label: "20海里", color: "#43D9B7", radius: 1852 * 20 },
  { label: "10海里", color: "#FEBB45", radius: 1852 * 10 },
  { label: "5海里", color: "#DD2A2A", radius: 1852 * 5 },
];

const source = [
  {
    guid: "chenhui",
    position: {
      lng: 121.880877501061,
      lat: 31.84957600087311,
    },
    name: "晨晖区域",
  },
  {
    guid: "qy1",
    position: {
      lng: 121.64052,
      lat: 31.94506,
    },
    name: "区域一",
  },
  {
    guid: "BBMS245",
    position: {
      lng: 121.80797186553963,
      lat: 31.66817494666304,
    },
    name: "BBMS245",
  },
  {
    guid: "JG_02_qy3",
    position: {
      lng: 121.74473115763718,
      lat: 31.6688305788027,
    },
    name: "机构_02区域3",
  },
  {
    guid: "JG_02_1_qy1",
    position: {
      lng: 121.75048031290103,
      lat: 31.696363013004408,
    },
    name: "机构_02_1区域1",
  },
  {
    guid: "JG_02_1_qy2",
    position: {
      lng: 121.85281527659772,
      lat: 31.662929698800973,
    },
    name: "机构_02_1区域2",
  },
  {
    guid: "JG_qy3",
    position: {
      lng: 121.75048031290103,
      lat: 31.696363013004408,
    },
    name: "区域3",
  },
];

let _map = null;
let trackLine = null;
onMounted(() => {
  const map = new MyMap("map");

  _.each(circles, (o) => {
    map.drawCircle([31.28, 121.75], {
      radius: o.radius,
      color: o.color,
      weight: 2,
      dashArray: "5,5",
      fillColor: "rgba(0,0,0,.5)",
      fillOpacity: 0.3,
    });
  });

  map.drawShips([
    {
      mmsi: "1120154506",
      label: "神华506",
      lng: 121.705,
      lat: 31.3,
      course: 45,
      updatetime: "2024-5-31",
      width: 60,
      length: 110,
    },
    {
      mmsi: "1120154511",
      label: "神华511",
      lng: 121.822848,
      lat: 31.254086,
      updatetime: "2024-5-31",
      speed: 5,
    },
    {
      mmsi: "1120157485",
      label: "中远海运12",
      lng: 121.792848,
      lat: 31.284086,
      course: 71,
      updatetime: "2024-6-5",
    },
    {
      mmsi: "11201574822",
      label: "腾达",
      lng: 122.0,
      lat: 31.35,
      course: 264,
      updatetime: "2024-5-31",
      width: 20,
      length: 60,
    },
  ]);

  trackLine = map.drawTrajectory(historyLines, {
    color: "red",
    smoothFactor: 2,
  });

  map.drawPointMarkers([
    {
      lat: 31.374086,
      lng: 121.732848,
      text: "2024-7-19 15:32:41",
      icon: stop1Png,
    },
    { lat: 31.36, lng: 121.805, text: "2024-7-20 15:32:41", icon: stop2Png },
  ]);

  map.drawHtmlOverlay(
    [31.28, 121.75],
    `<div class="layer"><div class="name">神华536</div><img width="36" height="36" src="${wavePng}" class="bg" /><div class="triangle"></div></div>`
  );

  map.drawHtmlOverlay(
    [31.28, 121.75],
    `<img src=${shipImg} style="transform: rotate(45deg);width: 5px;height: 16px;"></img>`
  );

  map.drawMarker({ lat: 31.3, lng: 121.75 });

  map
    .createMarkerClusterer(
      (o) => {
        return `<div
          class="marker"
          style='
            transform: translateX(${
              o.reverse ? "calc(-50% + 16px)" : "calc(50% - 16px)"
            });
            flex-direction: ${o.reverse ? "row-reverse" : "row"}
          '
        >
          <img width='25px' src="${iconPng}" />
          <div class="label">
            <div
              style='flexDirection: ${o.reverse ? "row-reverse" : "row"}'>
              <span>${o.key}</span> <span>附加</span>
            </div>
          </div>
    </div>`;
      },
      (info) => {
        // 聚合对象点击回调
      }
    )
    .markerClustererSetData(source);

  _map = map;
});

onUnmounted(() => {
  _map.destory();
});

const play = () => {
  _map.playAnimatedMarker(
    {
      points: trackLine.getSimplifyLatLngs(),
      totalDuration: 10000,
    },
    {
      iconSize: [38, 26],
      iconAnchor: [19, 13],
      iconUrl: icon,
    },
    null, // 每帧动画渲染的回调
    () => {
      // 动画结束
    }
  );
};

const pauseRef = ref(null);
const pause = () => {
  pauseRef.value = !pauseRef.value;
  _map.pauseAnimatedMarker(pauseRef.value);
};

const drawPolyLine = (e) => {
  e.stopPropagation();
  _map.startDrawPolyline();
};

const drawPolygon = (e) => {
  e.stopPropagation();
  _map.startDrawPolygon();
};
</script>

<style lang="less" scoped>
#map {
  width: 100%;
  height: 100%;
  position: relative;

  .btns {
    position: absolute;
    right: 10px;
    top: 10px;
    background-color: rgba(0, 0, 0, 0.5);
    padding: 4px;
    border-radius: 6px;
    z-index: 999;
    color: #fff;
    font-size: medium;

    button {
      margin: 0 4px;
    }
  }

  // 需要穿透到子组件
  :deep(.layer) {
    position: relative;
    white-space: nowrap;
    display: inline-flex;
    justify-content: center;

    @keyframes wave {
      from {
        transform: scale(1);
        opacity: 1;
      }
      50% {
        transform: scale(1.5);
        opacity: 0.5;
      }
      to {
        transform: scale(2);
        opacity: 0;
      }
    }

    .bg {
      animation: wave 3s linear 0s infinite;
      position: relative;
    }

    .triangle {
      position: absolute;
      top: -8px;
      left: calc(50% - 4px);
      width: 0;
      height: 0;
      display: inline-block;
      border-width: 4px;
      border-style: solid;
      border-color: rgba(0, 0, 0, 0.5) transparent transparent transparent;
    }

    .name {
      position: absolute;
      top: calc(-50% - 10px);
      padding: 2px 4px;
      background-color: rgba(0, 0, 0, 0.5);
      border-radius: 4px;
      color: rgb(255, 255, 255);
    }
  }

  :deep(.marker) {
    display: flex;
    align-items: center;
    position: relative;
    background-color: rgba(0, 0, 0, 0.5);
    padding: 4px;
    border-radius: 40px;

    & > img {
      border: 2px solid #dedede;
      z-index: 2;
      border-radius: 50%;
    }

    .label {
      display: inline-flex;
      flex-direction: column;
      justify-content: center;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      color: white;
      z-index: 1;
      padding: 0 4px;

      & > div {
        height: 15px;
        line-height: 15px;
        font-size: 10px;
        vertical-align: middle;
        display: flex;
        justify-content: space-between;
        align-items: center;

        .badge {
          background-color: red;
          color: white;
          padding: 0 4px;
          border-radius: 10px;
        }
      }
    }

    &:hover {
      cursor: pointer;
      background-color: yellow;
      z-index: 999;

      .label {
        color: black;
      }

      &::after {
        border-bottom-color: yellow;
      }
    }
  }
}
</style>
