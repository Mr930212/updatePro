<template>
  <div class="home-left" :class="{ 'hidden-sider': !showLeft }">
    <div class="left-btn" @click="showLeft = !showLeft">
      <ArrowButton type="left"></ArrowButton>
    </div>
    <div class="sider-content">
      <div class="sider-content-body">
        <div class="top-head">
          <span>文物点统计</span>
          <img src="../../../assets/chillerImg.png" alt="" />
        </div>
        <div class="plot-content unconfirmed-list">
          <div class="yqlist">
            <ul class="clearfix">
              <li v-for="item in circleData" :key="item.address">
                <div class="yq" id="yq">{{ item.value }}</div>
                <span>{{ item.address }}</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="top-head">
          <span>文物馆统计</span>
          <img src="../../../assets/chillerImg.png" alt="" />
        </div>
        <div class="alarm-content" id="piechart"></div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, onUnmounted } from "vue";
import ArrowButton from "./ArrowButton.vue";
import { Pie, G2 } from "@antv/g2plot";
const G = G2.getEngine("canvas");
const showLeft = ref(true);
const piePlot = ref(null);
const circleData = ref([
  { address: "湘东区", value: 76 },
  { address: "安源区", value: 45 },
  { address: "上栗县", value: 32 },
  { address: "莲花县", value: 21 },
  { address: "芦溪县", value: 10 },
]);
const pieData = ref([
  { address: "湘东区", value: 27 },
  { address: "安源区", value: 25 },
  { address: "上栗县", value: 18 },
  { address: "莲花县", value: 15 },
  { address: "芦溪县", value: 10 },
]);
const countBy = (arr, key) =>
  arr.reduce((acc, obj) => {
    const value = obj[key];
    acc[value] = (acc[value] || 0) + 1;
    return acc;
  }, {});
onMounted(() => {
  const Data = JSON.parse(localStorage.getItem("cultural")).filter(
    (item) => item.type == 2
  );
  const occurrences = countBy(Data, "address_name");
  circleData.value = Object.entries(occurrences).map(([address, value]) => ({
    address,
    value,
  }));
  alarmPie();
});

function alarmPie() {
  const Data = JSON.parse(localStorage.getItem("cultural")).filter(
    (item) => item.type == 1
  );
  const occurrences = countBy(Data, "address_name");
  pieData.value = Object.entries(occurrences).map(([address, value]) => ({
    address,
    value,
  }));
  piePlot.value = new Pie("piechart", {
    padding: [30, 0, 30, 0],
    data: pieData.value,
    autoFit: true,
    angleField: "value",
    colorField: "address",
    legend: false,
    label: {
      type: "spider",
      labelHeight: 28,
      formatter: (data, mappingData) => {
        const group = new G.Group({});
        group.addShape({
          type: "circle",
          attrs: {
            x: 0,
            y: 0,
            width: 20,
            height: 30,
            r: 5,
            fill: mappingData.color,
          },
        });
        group.addShape({
          type: "text",
          attrs: {
            x: 10,
            y: 8,
            text: `${data.address}`,
            fill: "#c3e5ff",
          },
        });
        group.addShape({
          type: "text",
          attrs: {
            x: 10,
            y: 25,
            text: `${data.value}`,
            fill: "#c3e5ff",
            fontWeight: 500,
          },
        });
        return group;
      },
    },
    tooltip: {
      showTitle: true,
      showMarkers: false,
      domStyles: {
        "g2-tooltip": {
          fontSize: 12,
          fontWeight: 400,
          background: "rgba(0,0,0,0.6)",
          color: "#e0f7fa",
          "user-select": "none",
        }, // 设置 tooltip 的 css 样式
        "g2-tooltip-title": {
          width: "100%",
          height: "24px",
          lineHeight: "24px",
          textAlign: "left",
        }, // 设置 tooltip 的 css 样式
        "g2-tooltip-value": {
          fontWeight: "bold",
          marginLeft: -50,
        },
      },
    },
    interactions: [{ type: "element-selected" }, { type: "element-active" }],
  });
  // @ts-ignore
  piePlot.value.render();
}
onBeforeUnmount(() => {
  piePlot.value?.destroy();
});
</script>
<style scoped lang="less">
@import "../../../assets/style.css";
.home-left {
  width: 400px;
  height:100%;
  z-index: 100;
  position: absolute;
  animation: toolBarShowLeft 0s;
  left: 0px;
  top: 90px;
  .sider-content {
    width: 100%;
    border-radius: 5px;
    color: #fff;
    height: 100%;
    overflow: auto;
    overflow-x: hidden;
    padding-bottom: 70px;
    background: linear-gradient(
      -90deg,
      rgba(8, 17, 40, 0.3) 0%,
      rgba(7, 22, 59, 0.6) 100%
    );
    &-body {
      padding: 0 16px 10px 0px;
      height:100%;
    }
    .top-head {
      width: 100%;
      padding-left: 10px;
      padding-top: 20px;
      font-size: 20px;
      color: #fff;
      font-style: oblique;
      text-shadow: 0 0 10px #128df9, 0 0 20px #128df9, 0 0 30px #128df9,
        0 0 40px #128df9;
      img {
        margin-top: -10px;
      }
    }
    .plot-content {
      width: 100%;
      margin: 20px auto 10px auto;
    }
    .unconfirmed-list {
      position: relative;
      width: 100%;
      height: 250px;
      .clearfix {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        column-gap: 15px;
      }
      .yqlist li {
        width: 30%;
        float: left;
        padding: 10px 0;
        text-align: center;
        margin: 0 auto;
      }
      .yq {
        width: 90px;
        height: 90px;
        margin: 0 auto 5px auto;
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;

        font-size: 20px;
        font-family: electronicFont;
        color: #fff32b;
      }
      .yqlist li span {
        opacity: 0.6;
        font-size: 14px;
      }

      .yq:before {
        position: absolute;
        width: 100%;
        height: 100%;
        content: "";
        background: url(../../../assets/img1.png) center center;
        border-radius: 100px;
        background-size: 100% 100%;
        opacity: 1;
        left: 0;
        top: 0;
        animation: myfirst2 15s infinite linear;
      }

      .yq:after {
        position: absolute;
        width: 86%;
        background: url(../../../assets/img2.png) center center;
        border-radius: 100px;
        background-size: 100% 100%;
        opacity: 1;
        height: 86%;
        content: "";
        left: 7%;
        top: 7%;
        animation: myfirst 15s infinite linear;
      }

      @keyframes myfirst {
        to {
          transform: rotate(-360deg);
        }
      }
      @keyframes myfirst2 {
        to {
          transform: rotate(360deg);
        }
      }
    }
    .alarm-content {
      height: 300px;
      margin-top: 40px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .energy-content {
      height: 150px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .destroy-content {
      position: relative;

      .bubble-circle {
        position: absolute;
        top: 10px;
        left: 15px;
        width: 40px;
        height: 40px;
        text-align: center;
        opacity: 0.9;
        box-shadow: 0 0 20px 4px #347ebb inset;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
      }
      p {
        margin-left: 60px;
      }
    }
    .destroy-content p:nth-of-type(2) {
      font-size: 16px;
      font-weight: bold;
      span {
        font-size: 14px;
        font-weight: normal;
        color: #ccc;
      }
    }
    .security-content {
      p {
        font-size: 16px;
        font-weight: bold;
        text-align: center;
        span {
          font-weight: 14px;
          font-weight: normal;
          color: #ccc;
        }
      }
      p:first-of-type {
        font-size: 14px;
        color: #ccc;
        font-weight: normal;
      }
    }
    .last-content {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100px;
      column-gap: 50px;
    }
  }
}
// 收缩动画
.left-btn {
  position: absolute;
  top: calc(50% - 70px);
  right: -10px;
  animation: leftSideBtnShow 0s;
  animation-fill-mode: forwards;
}
@keyframes leftSideBtnShow {
  from {
    transform: translateX(-12px) rotate(180deg);
  }
  to {
    transform: translateX(0px) rotate(0deg);
  }
}
.hidden-sider {
  animation: toolBarHiddenLeft 0s ease-in;
  animation-fill-mode: forwards;
  height: 100%;

  .left-btn {
    right: -30px;
    animation: leftSideBtnHidden 0s ease-in-out;
    animation-fill-mode: forwards;
  }
  @keyframes leftSideBtnHidden {
    from {
      transform: translateX(0px) rotate(0deg);
    }
    to {
      transform: translateX(-12px) rotate(180deg);
    }
  }
  @keyframes toolBarHiddenLeft {
    from {
      width: 400px;
    }
    to {
      width: 0px;
    }
  }
}
@keyframes toolBarShowLeft {
  from {
    width: 0px;
  }
  to {
    width: 400px;
  }
}
</style>
