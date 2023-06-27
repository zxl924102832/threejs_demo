<template>
  <div class="home">
    <div class="canvas-container" ref="canvasDom"></div>
      <div class="home-content">
      <div class="home-content-title">
        <h1>赛车展示与选配</h1>
      </div>
      <h2>选择车身颜色</h2>
      <div class="select">
        <div class="select-item" v-for="(item,index) in colors" :key="index" @click="selectColor(index)">
        <div class="select-item-color" :style="{backgroundColor:item}"></div>
        </div>
      </div>
      <h2>选择纹理贴图</h2>
      <div class="select">
        <div class="select-item" v-for="(item,index) in textures" :key="index" @click="selectTexture(item)">
          <div class="select-item-map" :style="{backgroundImage:`url(${item})`}"></div>
        </div>
      </div>
      <h2>选择车身材质</h2>
      <div class="select">
        <div class="select-item" v-for="(item,index) in materials" :key="index" @click="selectMaterial(index)">
          <div class="select-item-text">{{item.name}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import * as THREE from "three";
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'
import {RGBELoader} from "three/examples/jsm/loaders/RGBELoader"

import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

// 导入动画库
import gsap from "gsap"
import {onMounted,reactive,ref}from "vue";


let canvasDom=ref(null);
// 1.创建场景
const scene=new THREE.Scene();

// 2.创建相机
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

// 设置相机位置
camera.position.set(0,5,20);
scene.add(camera);
// 初始化渲染器
const renderer=new THREE.WebGLRenderer({
  // 抗锯齿
  antialias:true
});
renderer.setSize( window.innerWidth, window.innerHeight );
// 开启场景中的阴影贴图
renderer.shadowMap.enabled=true;
// 创建轨道控制器
let controls = new OrbitControls(camera, renderer.domElement); 
function render(){
    // let time=clock.getElapsedTime();
    // smallBall.position.x=Math.sin(time)*3;
    // smallBall.position.z=Math.cos(time)*3;
    controls.update();
    renderer.render(scene,camera);
    // 渲染下一帧的时候就会调用render函数
    requestAnimationFrame(render);
}
// 加载HDR环境贴图
const rgbeLoader=new RGBELoader();
rgbeLoader.loadAsync("./textures/studioHDR_4_whiteShiny.hdr").then(texture=>{
  texture.mapping=THREE.EquirectangularReflectionMapping;
  scene.background=texture;
});

// 创建平面
// const planeGeometry=new THREE.PlaneBufferGeometry(100,100);
// const material=new THREE.MeshStandardMaterial({
//      metalness:0.5,roughness:0.5//,envMap:envMapTexture
// });
// const plane=new THREE.Mesh(planeGeometry,material);
// plane.position.set(0,-10,0);
// plane.rotation.x=-Math.PI/2;
// // 接受阴影
// plane.receiveShadow=true;
// scene.add(plane);

// const sphereGeometry=new THREE.SphereBufferGeometry(1,5,5);
// const sphere_material=new THREE.MeshStandardMaterial({
//     // metalness:0.9,roughness:0.1//,envMap:envMapTexture
// });
// const sphere=new THREE.Mesh(sphereGeometry,material);
// sphere.castShadow=true;
// scene.add(sphere);

// 导入纹理
const textureLoader=new THREE.TextureLoader();
let cartexture=textureLoader.load('./textures/1.png');

let wheels=[];
let carBody,frontCar;
//创建材质
const bodyMaterial=new THREE.MeshPhysicalMaterial({
  color:0xffffff,
  metalness:1,
  roughness:0.4,
  clearcoat:1,
  clearcoatRoughness:0,
  // map:cartexture
})
const wheelsMaterial=new THREE.MeshPhysicalMaterial({
  color:0x000000,
  metalness:1,
  roughness:0.2,
  transmission:1,
  transparent:true,
  opacity:0.7
})
const wheelborderMaterial=new THREE.MeshPhysicalMaterial({
  color:0xffffff,
  metalness:1,
  roughness:0.1,
})
const wheelrowMaterial=new THREE.MeshPhysicalMaterial({
  color:0x000000,
  metalness:1,
  roughness:0.8,
})
let colors=[
  "red",
  "blue",
  "green",
  "gray",
  "yellow",
  "white",
  "purple"
]
let textures=["./textures/1.png","./textures/2.png","./textures/3.png","./textures/4.png","./textures/5.png","./textures/6.png",null];
let materials=[{name:"磨砂",value:1},{name:"冰晶",value:0}];
let selectColor=(index)=>{
  bodyMaterial.color.set(colors[index]);
  wheelborderMaterial.color.set(colors[index]);
};
let selectMaterial=(index)=>{
  bodyMaterial.clearcoatRoughness=materials[index].value;
};
let selectTexture=(item)=>{
  if(item===null){
    bodyMaterial.map=null;
    return;
  }
  cartexture=textureLoader.load(item);
  bodyMaterial.map=cartexture;
}
// 挂载完毕
onMounted(()=>{
  canvasDom.value.appendChild(renderer.domElement);
  // console.log(canvasDom.value);
  renderer.setClearColor('#000');
  scene.background=new THREE.Color('#ccc');
  scene.environment=new THREE.Color('#ccc');

  render();
  //添加网格
  // const gridHelper=new THREE.GridHelper(50,50);
  // gridHelper.material.opacity=0.4;
  // gridHelper.material.transparent=true;
  // scene.add(gridHelper);
  
  // 设置控制器阻尼
  controls.enableDamping=true;
  // 添加gltf汽车模型
  const loader=new GLTFLoader();
  const dracoLoader=new DRACOLoader();
  dracoLoader.setDecoderPath("./draco/gltf/");
  loader.setDRACOLoader(dracoLoader);
  loader.load('./model/formula_car.glb',(gltf)=>{ //
    const heli=gltf.scene;
    // console.log(heli);
    heli.traverse((child)=>{
      if(child.isMesh){
        // console.log(child);
        child.castShadow=true;
      }
      //Object_4 手臂 5手
      // 6 车内前面 7 连轴 8 车轮中间 9车身及前面
      // 10车尾细线 12方向盘 13车轮纵向线 14 车轮面 15车轮子 16 车轮边 17车轮金属连接
      if(child.isMesh&&child.name.includes("Object_9")){
        child.material=bodyMaterial;
      }
      if(child.isMesh&&child.name.includes("Object_14")){
        child.material=wheelsMaterial;
      }
      if(child.isMesh&&child.name.includes("Object_15")){
        child.material=wheelrowMaterial;
      }
      if(child.isMesh&&child.name.includes("Object_16")){
        child.material=wheelborderMaterial;
      }
      if(child.isMesh&&child.name.includes("Object_7")){
        child.material=wheelborderMaterial;
      }
    })
    heli.position.set(0,-5,0);
    heli.castShadow=true;
    scene.add(heli);
  });
  // 添加灯光
  // let lights=new Array(6);
  // let locations=[(),()];
  // const light1=new THREE.DirectionalLight(0xffffff,1);
  // light1.position.set(0,0,10);
  // scene.add(light1);
  // light1.castShadow=true;
  // const light2=new THREE.DirectionalLight(0xffffff,1);
  // light2.position.set(0,0,-10);
  // scene.add(light2);
  // light2.castShadow=true;
  const light3=new THREE.DirectionalLight(0xffffff,1);
  light3.position.set(0,20,0);
  scene.add(light3);
  light3.castShadow=true;
  // 设置阴影贴图模糊度
  light3.shadow.radius=20;
  // const light4=new THREE.DirectionalLight(0xffffff,1);
  // light4.position.set(0,-10,0);
  // scene.add(light4);
  // light4.castShadow=true;
  // const light5=new THREE.DirectionalLight(0xffffff,1);
  // light5.position.set(10,0,0);
  // scene.add(light5);
  // const light6=new THREE.DirectionalLight(0xffffff,1);
  // light6.position.set(-10,0,0);
  // scene.add(light6);
})
</script>

<style>
*{
  margin:0;
  padding:0;
}
.home-content{
  position:fixed;
  top:0;
  right:20px;
}
.select-item-color{
  width:50px;
  height:50px;
  border:1px solid #ccc;
  margin:10px;
  display:inline-block;
  cursor:pointer;
  border-radius:10px;
}
.select-item-map{
  width:50px;
  height:50px;
  border:1px solid #ffffff;
  margin:10px;
  display:inline-block;
  cursor:pointer;
  border-radius:10px;
}
.select-item-text{
  font-size:24px;
  text-align:center;
  line-height:50px;
  width:100px; 
  background-color:#ffffff;
  height:50px;
  /* border:1px solid #000; */
  margin:10px;
  display:inline-block;
  cursor:pointer;
  border-radius:10px;
}
.select{
  display:flex;
}
</style>
