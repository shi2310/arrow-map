<template>
  <div id="map">
    <div class="btns">
      轨迹动画： <button @click="play">播放</button
      ><button @click="pause">{{ pauseRef ? "继续" : "暂停" }}</button>
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

// 历史轨迹线点
const historyLines = [
  { lat: 31.254086, lng: 121.742848, ts: "2024-7-19 15:32:41" },
  { lat: 31.254086, lng: 121.743048, ts: "2024-7-19 16:00:41" },
  { lat: 31.35, lng: 121.805, ts: "2024-7-20 16:32:41" },
  { lat: 31.37, lng: 121.835, ts: "2024-7-21 18:32:41" },
  { lat: 31.4, lng: 121.9, ts: "2024-7-22 15:32:41" },
];

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
      lng: 120.64052,
      lat: 39.34506,
    },
    name: "区域一",
  },
  {
    guid: "api_test",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "测试",
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
    guid: "111111",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "111111111",
  },
  {
    guid: "BBMS125",
    position: {
      lng: 121.78497524448419,
      lat: 31.685547682978168,
    },
    name: "BBMS125",
  },
  {
    guid: "BBMS87",
    position: {
      lng: 118.07869,
      lat: 24.597569,
    },
    name: "BBMS87",
  },
  {
    guid: "td_test",
    position: {
      lng: 121.77290201843009,
      lat: 31.69243031299051,
    },
    name: "td_test",
  },
  {
    guid: "DNH1",
    position: {
      lng: 121.76312845448153,
      lat: 31.68325336344183,
    },
    name: "大南湖二矿",
  },
  {
    guid: "JG_test1",
    position: {
      lng: 121.7642782855343,
      lat: 31.683908888080893,
    },
    name: "机构区域一",
  },
  {
    guid: "JG_01_qy1",
    position: {
      lng: 121.75852913027045,
      lat: 31.678664537578474,
    },
    name: "机构_01区域1",
  },
  {
    guid: "JG_01_qy3",
    position: {
      lng: 121.77462676500924,
      lat: 31.688497444667075,
    },
    name: "机构_01区域3",
  },
  {
    guid: "JG_02_qy1",
    position: {
      lng: 121.78842473764252,
      lat: 31.66817494666304,
    },
    name: "机构_02区域1",
  },
  {
    guid: "JG_02_qy2",
    position: {
      lng: 121.76772777869262,
      lat: 31.672764288504148,
    },
    name: "机构_02区域2",
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
    guid: "JG_02_1_qy3",
    position: {
      lng: 121.77002744079816,
      lat: 31.676697829952634,
    },
    name: "机构_02_1区域3",
  },
  {
    guid: "test10",
    position: {
      lng: 119.059944,
      lat: 32.22505,
    },
    name: "服务器10区域",
  },
  {
    guid: "test123",
    position: {
      lng: 109.51873,
      lat: 31.043587,
    },
    name: "嵌入式123区域",
  },
  {
    guid: "test28",
    position: {
      lng: 121.67401654789171,
      lat: 31.717989854042994,
    },
    name: "区域test28",
  },
  {
    guid: "hhh",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "hhh",
  },
  {
    guid: "HY001",
    position: {
      lng: 121.72920843842476,
      lat: 31.724870053556437,
    },
    name: "海运001",
  },
  {
    guid: "000",
    position: {
      lng: 114.40218455431486,
      lat: 30.472466374059625,
    },
    name: "srt_puser船",
  },
  {
    guid: "test",
    position: {
      lng: 121.7844003289578,
      lat: 31.755333308391904,
    },
    name: "test",
  },
  {
    guid: "jgquyu1",
    position: {
      lng: 97.458680011496,
      lat: 39.70536526512196,
    },
    name: "子jg区域1",
  },
  {
    guid: "nanhu",
    position: {
      lng: 121.4952178191857,
      lat: 31.902593106904582,
    },
    name: "南湖区域_测云台",
  },
  {
    guid: "tututu",
    position: {
      lng: 117.74374000000002,
      lat: 30.8934217,
    },
    name: "fsdfas",
  },
  {
    guid: "ttttttttttt",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "ttttttttttt",
  },
  {
    guid: "aa33",
    position: {
      lng: 117.63583413324218,
      lat: 38.98713489013469,
    },
    name: "船33",
  },
  {
    guid: "quyu0906",
    position: {
      lng: 114.44003726638697,
      lat: 30.485958472600327,
    },
    name: "区域0906",
  },
  {
    guid: "test02",
    position: {
      lng: 121.4693466204982,
      lat: 31.867272359051615,
    },
    name: "test",
  },
  {
    guid: "12312312",
    position: {
      lng: 121.86546341817822,
      lat: 31.779893078200757,
    },
    name: "测试",
  },
  {
    guid: "test_zxj",
    position: {
      lng: 116.98928,
      lat: -1.3487934,
    },
    name: "周小军测试",
  },
  {
    guid: "xpctest",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "许鹏程区域test",
  },
  {
    guid: "423423",
    position: {
      lng: 121.57973040156443,
      lat: 31.768105210254358,
    },
    name: "423423",
  },
  {
    guid: "53425432",
    position: {
      lng: 121.60272702261986,
      lat: 31.772034668111242,
    },
    name: "53453",
  },
  {
    guid: "xiaomingquyu",
    position: {
      lng: 121.7073616484221,
      lat: 31.81230189189942,
    },
    name: "小明区域",
  },
  {
    guid: "yuhuaquyu",
    position: {
      lng: 121.6464206026252,
      lat: 31.789715142651417,
    },
    name: "余化区域",
  },
  {
    guid: "xiaomingquyu1",
    position: {
      lng: 121.7763515115884,
      lat: 31.649159506897366,
    },
    name: "小明区域1",
  },
  {
    guid: "yuhuaquyu1",
    position: {
      lng: 121.75565455263852,
      lat: 31.694396684036857,
    },
    name: "余化区域1",
  },
  {
    guid: "yuhuaquyu2",
    position: {
      lng: 121.69701316894715,
      lat: 31.74649017993255,
    },
    name: "余化区域2",
  },
  {
    guid: "186server",
    position: {
      lng: 121.43715135102066,
      lat: 31.720938523887444,
    },
    name: "186server",
  },
  {
    guid: "82server",
    position: {
      lng: 119.06094,
      lat: 32.21927,
    },
    name: "82server",
  },
  {
    guid: "85server",
    position: {
      lng: 121.698163,
      lat: 31.730767,
    },
    name: "85server",
  },
  {
    guid: "JG_car1",
    position: {
      lng: 121.5849046413019,
      lat: 31.711436798653537,
    },
    name: "JG_car1",
  },
  {
    guid: "JG_qy1",
    position: {
      lng: 121.77577659606202,
      lat: 31.678664537578474,
    },
    name: "区域1",
  },
  {
    guid: "JG_qy2",
    position: {
      lng: 121.7435813265844,
      lat: 31.696363013004408,
    },
    name: "区域2",
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
onMounted(() => {
  const map = new MyMap("map");

  _.each(circles, (o) => {
    map.drawCircleRange([31.28, 121.75], {
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

  map.drawTrajectory(historyLines);

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

  map.drawMarker({ lat: 31.3, lng: 121.75 }, undefined);

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
      points: historyLines,
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
