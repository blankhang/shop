<template>
    <div class="loading" v-show="data.isLoading">
        <Loading :progress="data.progress"/>
    </div>
    <div class="product" v-show="!data.isLoading" id="product">
        <div
            class="desc"
            :class="{active: data.descIndex === i}"
            v-if="data.products[data.pIndex]"
            v-for="(desc, i) in data.products[data.pIndex].desc"
        >
            <h1 class="title">{{desc.title}}</h1>
            <p class="content">{{desc.content}}</p>
        </div>
        <div class="prod-list" :class="{hidden: store.state.isFullScreen}">
            <h1><SketchOutlined/>产品推荐</h1>
            <div class="products">
                <div class="prod-item"
                    v-for="(prod, pi) in data.products"
                    @click="changeModel(prod, pi)"
                    :class="{active: pi === data.pIndex}"
                >
                    <div class="prod-title">{{prod.title}}</div>
                    <div class="img">
                        <img :src="prod.imgsrc" :alt="prod.title"/>
                    </div>
                    <a-button type="primary" block @click.stop="addBuycart(prod)"><ShoppingCartOutlined/>加入购物车</a-button>
                </div>
            </div>
        </div>
        <div class="scene-list" :class="{hidden: store.state.isFullScreen}">
            <h3><RadarChartOutlined/>切换使用场景</h3>
            <div class="scenes">
                <div class="scene-item" v-for="(sence, index) in data.scenes" @click="changeHdr(sence, index)">
                    <img :src="`./files/hdr/${sence}.jpg`" alt="" :class="{active: index === data.sceneIndex}">
                </div>
            </div>
        </div>
    </div>
</template>
<script setup>
import {useRoute} from 'vue-router';
import {useStore} from 'vuex';
import Base3d from '../utils/Base3d';
import {onMounted, reactive} from 'vue';
import * as api from '../api/index';
import Loading from '../components/Loading.vue';
import {SketchOutlined, RadarChartOutlined, ShoppingCartOutlined} from '@ant-design/icons-vue';
const route = useRoute();
const store = useStore();
const data = reactive({
    products: [],
    isLoading: true,
    scenes: [],
    pIndex: 0,
    sceneIndex: 0,
    base3d: {},
    descIndex: 0,
    progress: 0
});
onMounted(async () => {
  let result = {
    "result": "ok",
    "list": [
      {
        "id": 7589,
        "title": "GUCCI 古驰新款女包",
        "imgsrc": "./imgs/bag.png",
        "price": 17899,
        "modelPath": "./files/gltf/",
        "modelName": "bag2.glb",
        "desc": [
          {
            "title": "与一款全新的邮差包设计。",
            "content": "该系列手袋同时推出摩登廓形的水桶包款式"
          },
          {
            "title": "向60年前古驰的经典手袋致敬。",
            "content": "Gucci 1955马衔扣系列手袋延续经典手袋线条与造型"
          },
          {
            "title": "手袋结构设计精巧",
            "content": "搭配可调节长度的肩带，肩背或斜挎皆宜。"
          },
          {
            "title": "GUCCI 1955马衔扣系列手袋",
            "content": "标志性的马衔扣设计源于马术运动，由金属双环和一条衔链组合而成。"
          }
        ]
      },
      {
        "id": 7590,
        "title": "Macbook Pro",
        "imgsrc": "./imgs/macpro.jpg",
        "price": 25899,
        "modelPath": "./files/gltf/",
        "modelName": "Macbookpro2.glb",
        "desc": [
          {
            "title": "超高速M1 Pro和M1 Max芯片",
            "content": "带来颠覆性表现和惊人续航"
          },
          {
            "title": "炫目的Liquid视网膜XDR显示屏",
            "content": "Macbookpro各类强大端口也都整装就位"
          },
          {
            "title": "战力更猛，耐力也更强！",
            "content": "无论是剪辑8K视频、编译代码都能随时随地轻松搞定"
          },
          {
            "title": "Pro到MAX，霸气不封顶",
            "content": "图形处理器速度最高提升至4倍，机器学习性能提升至5倍"
          }
        ]
      },
      {
        "id": 7591,
        "title": "水晶凉鞋女细跟",
        "imgsrc": "./imgs/womenshoes.jpg",
        "price": 17899,
        "modelPath": "./files/gltf/",
        "modelName": "shoes.glb",
        "desc": [
          {
            "title": "白变女神季",
            "content": "性感潮品、优雅轻淑范！"
          },
          {
            "title": "舒适、焕新",
            "content": "手感光滑、富有弹性、舒适一整天"
          },
          {
            "title": "个性、魅力",
            "content": "水晶搭配金属，凸显柔美气质"
          },
          {
            "title": "全透、高端水晶",
            "content": "每一处的细节，都很到位!"
          }
        ]
      }
    ],
    "hdr": [
      "000",
      "080",
      "079",
      "077",
      "076",
      "067"
    ]
  }
    // let result = await api.getProducts();
    console.log(result);
    // debugger
    // data.isLoading = false;
    data.products = result.list;
    data.scenes = result.hdr;
    data.base3d = new Base3d('#product', LoadingFinish);
    data.base3d.onProgress(e => {
        let progressNum = e.loaded / e.total;
        progressNum = progressNum.toFixed(2) * 100;
        console.log(progressNum);
        data.progress = progressNum;
    });
});
function LoadingFinish() {
    data.isLoading = false;
}
function changeModel(prod, pI) {
    data.pIndex = pI;
    data.base3d.setModel(prod.modelName);
}
function changeHdr(scene, index) {
    data.sceneIndex = index;
    data.base3d.setEnvMap(scene);
}
function addBuycart(prod) {
    let product = {...prod, num: 1};
    let index = 0;
    let isExist = store.state.buycarts.some((item, i) => {
        if (product.id === item.id) {
            index = i;
            return true;
        } else {
            return false;
        }
    });
    // 原商品存在则+1，否则push进一个新商品
    if (isExist) {
        store.commit('addBuycartsNum', index);
    } else {
        store.commit('addBuycarts', product);
    }
}
// 根据鼠标滚轮来设置是否全屏
window.addEventListener('mousewheel', e => {
    // 向下
    if (e.deltaY > 0) {
        store.commit('setFullScreen', true);
    }
    if (e.deltaY < 0) {
        store.commit('setFullScreen', false);
    }
    let duration = data.base3d.animateAction._clip.duration;
    let time = data.base3d.animateAction.time;
    let index = Math.floor((time / duration) * 4);
    data.descIndex = index;
});
</script>
<style lang="less" scoped>
.desc {
    position: fixed;
    z-index: 100000;
    background-color: rgba(255, 255, 255, .5);
    width: 600px;
    top: 100px;
    left: 50%;
    margin-left: -300px;
    transition: all .5s;
    padding: 15px;
    transform: translate(-100vw, 0);
}
.desc.active {
    transform: translate(0, 0);
}
.prod-list {
    width: 300px;
    height: 100vh;
    padding: 60px 0 0;
    position: fixed;
    z-index: 1000;
    transition: all .5s;
    background-color: rgba(255, 255, 255, .8);
    left: 0;
    top: 0;
    &.hidden {
        transform: translate(-100%, 0);
    }
    h1 {
        font-size: 20px;
        font-weight: 900;
        padding: 10px 25px 0;
    }
    .products {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        .prod-item {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            width: 250px;
            background-color: #fff;
            border-radius: 20px;
            overflow: hidden;
            margin: 10px 0;
            box-shadow: 5px 5px 10px #666;
            transition: all .3s;
            &:hover {
                transform: translate(0, -5px);
                box-shadow: 5px 5px 10px #666, 0 0 20px orangered;
            }
            &.active {
                box-shadow: 5px 5px 10px #666, 0 0 20px red;
            }
            img {
                width: 190px;
            }
            .prod-title {
                padding: 0 20px;
            }
        }
    }
}
.scene-list {
    width: 300px;
    height: 100vh;
    padding: 60px 0 0;
    position: fixed;
    z-index: 1000;
    transition: all .5s;
    background-color: rgba(255, 255, 255, .8);
    right: 0;
    top: 0;
    &.hidden {
        transform: translate(100%, 0);
    }
    h3 {
        font-size: 20px;
        font-weight: 900;
        padding: 0 30px;
    }
    .scenes {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        .scene-item {
            padding: 6px 0;
            img {
                width: 250px;
                border-radius: 10px;
                box-shadow: 5px 5px 10px #666;
                transition: all .3s;
                &:hover {
                    transform: translate(0, -5px);
                }
                &.active {
                    box-shadow: 5px 5px 10px #666, 0 0 20px red;
                }
            }
        }
    }
}
</style>
