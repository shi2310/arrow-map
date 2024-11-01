<template>
  <div id="map">
    <div class="btns">
      <p>
        轨迹动画： <button @click="play">播放</button
        ><button @click="pause">{{ pauseRef ? "继续" : "暂停" }}</button>
      </p>
      <p>工具：<button @click="drawMeasure">测量</button></p>
      <p>
        自由绘制： <button @click="drawPolyLine">折线</button
        ><button @click="drawPolygon">多边形</button>
      </p>
      <div class="list" v-if="state.polylines.length">
        <p>折线：</p>
        <div v-for="(o, i) in state.polylines">
          <span class="dot"
            ><i>{{ i + 1 }}</i>
          </span>
          <span class="txt" :title="o.latLngs.toString()"
            >主键：{{ o.id }}</span
          >
          <span class="dot" @click="o.toggle">{{
            o.show ? "隐藏" : "显示"
          }}</span
          ><span class="dot" @click="o.remove">删除</span>
        </div>
      </div>
      <div class="list" v-if="state.polygons.length">
        <p>多边形：</p>
        <div v-for="(o, i) in state.polygons">
          <span class="dot"
            ><i>{{ i + 1 }}</i>
          </span>
          <span class="txt" :title="o.latLngs.toString()"
            >主键：{{ o.id }}</span
          >
          <span class="dot" @click="o.toggle">{{
            o.show ? "隐藏" : "显示"
          }}</span
          ><span class="dot" @click="o.remove">删除</span>
        </div>
      </div>
      <p>航道： <button @click="drawRouteArea">航道</button></p>
      <div class="list" v-if="state.routeAreas.length">
        <div v-for="(o, i) in state.routeAreas">
          <span class="dot"
            ><i>{{ i + 1 }}</i>
          </span>
          <span class="txt" :title="o.latLngs.toString()"
            >主键：{{ o.id }} 名称:{{ o.name }}</span
          >
          <span class="dot" @click="o.remove()">删除</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onUnmounted, reactive } from "vue";
import stop1Png from "@/assets/stop1.png";
import stop2Png from "@/assets/stop2.png";
import MyMap from "arrow-map";
import icon from "@/assets/mShip.png";
import wavePng from "@/assets/wave.png";
import shipImg from "@/assets/ship.png";
import iconPng from "@/assets/icon.png";
import bgPng from "@/assets/bg.png";
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

const state = reactive({ polygons: [], polylines: [], routeAreas: [] });

let _map = null;
let trackLine = null;
onMounted(() => {
  const map = new MyMap("map");

  map.groundOverlay(
    [
      [31.721139190517842, 121.67430248603551],
      [31.732439796482474, 121.69906802998445],
    ],
    {
      type: "image",
      url: bgPng,
      opacity: 1,
    }
  );

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

  map
    .drawShips([
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
    ])
    .setSelectedItem("1120157485");

  trackLine = map.drawTrajectory(historyLines, {
    color: "red",
    smoothFactor: 4,
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
        // 错位布局时html模板
        return `<div
          class="marker"
          style='
            transform: translateX(${
              o.reverse ? "calc(-50% + 14px)" : "calc(50% - 14px)"
            });
            flex-direction: ${o.reverse ? "row-reverse" : "row"}
          '
        >
          <img width='24px' src="${iconPng}" />
          <div class="label" style='flexDirection: ${
            o.reverse ? "row-reverse" : "row"
          }'>
            <div>${o.key}</div>
          </div>
        </div>`;
      },
      16,
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
  _map.startDrawPolyline(({ id, latLngs, show, toggle, remove }) => {
    // 数据变化
    const index = _.findIndex(state.polylines, { id });
    if (index >= 0) {
      if (latLngs) {
        state.polylines[index] = { id, latLngs, show, toggle, remove };
      } else {
        // 表示移除
        state.polylines.splice(index, 1);
      }
    } else {
      state.polylines.push({ id, latLngs, show, toggle, remove });
    }
  });
};

const drawPolygon = (e) => {
  e.stopPropagation();
  _map.startDrawPolygon(({ id, latLngs, show, toggle, remove }) => {
    // 数据变化
    const index = _.findIndex(state.polygons, { id });
    if (index >= 0) {
      if (latLngs) {
        state.polygons[index] = { id, latLngs, show, toggle, remove };
      } else {
        // 表示移除
        state.polygons.splice(index, 1);
      }
    } else {
      state.polygons.push({ id, latLngs, show, toggle, remove });
    }
  });
};

// 绘制测量标尺
const drawMeasure = (e) => {
  e.stopPropagation();
  _map.startDrawMeasure();
};

const drawRouteArea = (e) => {
  e.stopPropagation();
  _map.startDrawAreaByLine(({ id, latLngs, remove, angle, index, groupId }) => {
    // 数据变化
    const name = `${groupId}_${angle >= 0 ? "右" : "左"}_${index}`;
    const _index = _.findIndex(state.routeAreas, { id });
    if (_index >= 0) {
      if (latLngs) {
        state.routeAreas[_index] = { id, latLngs, remove, name };
      } else {
        // 表示移除
        state.routeAreas.splice(_index, 1);
      }
    } else {
      state.routeAreas.push({ id, latLngs, remove, name });
    }
  });
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

    .list {
      max-height: 500px;
      overflow-y: auto;

      & > div {
        display: flex;
        align-items: center;
        margin: 4px 0;
        border: 1px solid #f8f8f8;
        border-radius: 4px;

        span {
          padding: 4px;
        }

        .dot {
          text-align: center;
          padding: 4px;
          display: inline-block;
          cursor: pointer;

          i {
            display: inline-block;
            background-color: blue;
            color: #fff;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            font-size: 0.8em;
          }
        }

        .txt {
          flex: 1;
          border-right: 1px solid #f8f8f8;
          border-left: 1px solid #f8f8f8;
        }
      }
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
    padding: 2px;
    border-radius: 40px;

    & > img {
      border: 2px solid #dedede;
      z-index: 2;
      border-radius: 50%;
    }

    .label {
      display: flex;
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
