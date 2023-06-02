	<script type="text/javascript">
	    var playlist = ['./model/七里香.mp3', './model/稻香.mp3', './model/反方向的钟.mp3']; // 音乐列表
	    		var count = getRandomInt(0, playlist.length); // 当前播放的音乐索引
	    		var isFirstTime = true;
	    		var audio = createAudio(playlist[count]);
	    
	    		// 创建音频对象
	    		function createAudio(src) {
	    			var audio = new Audio(src);
	    			audio.onended = function() {
	    				playNext();
	    			}
	    			return audio;
	    		}
	    
	    		// 生成随机整数
	    		function getRandomInt(min, max) {
	    			min = Math.ceil(min);
	    			max = Math.floor(max);
	    			return Math.floor(Math.random() * (max - min)) + min;
	    		}
	    
	    		// 播放下一首歌
	    		function playNext() {
	    			count = (count + 1) % playlist.length; // 选择下一首音乐
	    			audio.src = playlist[count]; // 替换音乐路径
	    			audio.play();
	    		}
	    
	    		// 播放音乐
	    		document.addEventListener('mousedown', function(event) {
	    			if (event.button === 0) { // 鼠标左键
	    				if (isFirstTime) {
	    					count = getRandomInt(0, playlist.length); // 随机选择一首音乐
	    					audio.src = playlist[count];
	    					isFirstTime = false;
	    				}
	    				audio.play();
	    			}
	    		});
	    
	    		// 暂停/继续播放音乐
	    		document.addEventListener('keydown', function(event) {
	    			if (event.code === 'Space') {
	    				if (audio.paused) {
	    					audio.play();
	    				} else {
	    					audio.pause();
	    				}
	    			}
	    		});
				document.onkeydown = function(event) {
				      if (event.code === "ArrowRight") { // 向右箭头键
				        playNext();
				      }
				    }
		
				//window.alert("Hello world!");

					
		
	</script>








<script setup>
import * as THREE from "three";
import * as YUKA from "yuka";
// 导入控制器
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { Reflector } from "./mesh/Reflector.js";

// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 50, 20);
// 0 10 20
camera.lookAt(0, 0, 0);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.shadowMap.enabled = true;
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1;
document.body.appendChild(renderer.domElement);

// 创建地面
// const planeGeometry = new THREE.PlaneGeometry(50, 50);
// const planeMaterial = new THREE.MeshStandardMaterial({ color: 0x999999 });
// const plane = new THREE.Mesh(planeGeometry, planeMaterial);
// plane.receiveShadow = true;
// plane.rotation.x = -Math.PI / 2;
// plane.position.y = -0.5;
// scene.add(plane);

// 创建灯光
// const light = new THREE.SpotLight(0xffffff, 3, 100, Math.PI / 6, 0.5);
// light.position.set(10, 40, 10);
// light.castShadow = true;
// scene.add(light);

// 创建控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

const time = new YUKA.Time();
requestAnimationFrame(function animate() {
  const delta = time.update().getDelta();
  controls.update();
  entityManager.update(delta);

  // if (navMesh) {
  //   const currentRegion = navMesh.getRegionForPoint(vehicle.position);
  //   if (currentRegion) {
  //     const distance = currentRegion.distanceToPoint(vehicle.position);
  //     vehicle.position.y -= distance * 0.2;
  //   }
  // }
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
});

class CustomVechicle extends YUKA.Vehicle {
  constructor(navMesh) {
    super();
    this.navMesh = navMesh;
  }
  update(delta) {
    super.update(delta);
    const currentRegion = this.navMesh.getRegionForPoint(this.position);
    if (currentRegion) {
      const distance = currentRegion.distanceToPoint(this.position);
      this.position.y -= distance * 0.2;
    }
  }
}

// 加载模型
const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("./draco/");
loader.setDRACOLoader(dracoLoader);

let plane;

loader.load("./model/city.gltf", function (gltf) {
  let city = gltf.scene;
  plane = city;
  scene.add(city);
});

// 创建yuka的车辆
// const vehicle = new YUKA.Vehicle();

// 设置车辆的渲染对象
// vehicle.setRenderComponent(cone, callback);
function callback(entity, renderComponent) {
  // console.log(entity, renderComponent);
  renderComponent.position.copy(entity.position);
  renderComponent.quaternion.copy(entity.rotation);
  // renderComponent.matrix.copy(entity.worldMatrix);
}

// 创建目标小球
const sphereGeometry = new THREE.SphereGeometry(0.1, 32, 32);
const sphereMaterial = new THREE.MeshStandardMaterial({ color: 0xff00ff });
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphere.receiveShadow = true;
sphere.castShadow = true;
scene.add(sphere);
// 创建目标
const target = new YUKA.GameEntity();
target.setRenderComponent(sphere, callback);

target.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10);

const entityManager = new YUKA.EntityManager();

entityManager.add(target);

// 到达行为
// const arriveBehavior = new YUKA.ArriveBehavior(target.position);
// vehicle.steering.add(arriveBehavior);

// 搜索目标行为
// const seekBehavior = new YUKA.SeekBehavior(target.position);
// vehicle.steering.add(seekBehavior);

// setInterval(() => {
//   target.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10);
// }, 2000);
let line;
function showPathLine(path) {
  if (line) {
    scene.remove(line);
    line.geometry.dispose();
    line.material.dispose();
  }
  // console.log(path);
  const positions = [];
  for (let i = 0; i < path._waypoints.length; i++) {
    positions.push(
      path._waypoints[i].x,
      path._waypoints[i].y,
      path._waypoints[i].z
    );
  }
  const geometry = new THREE.BufferGeometry();
  geometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(positions, 3)
  );
  const material = new THREE.LineBasicMaterial({ color: 0x0000ff });
  line = new THREE.Line(geometry, material);
  scene.add(line);
}

// 点击将目标移动到点击的位置
const ndc = new THREE.Vector2();
const raycaster = new THREE.Raycaster();
window.addEventListener("pointerdown", (event) => {
  ndc.x = (event.clientX / window.innerWidth) * 2 - 1;
  ndc.y = -(event.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(ndc, camera);
  const intersects = raycaster.intersectObject(plane);
  if (intersects.length > 0) {
    const point = intersects[0].point;
    target.position.set(point.x, 0, point.z);
    let from = vehicle.position;
    let to = point;

    // 根据导航网格获取路径
    console.log(navMesh);
    const path = navMesh.findPath(from, to);
    // console.log(path);
    const path1 = new YUKA.Path();
    for (let item of path) {
      path1.add(new YUKA.Vector3(item.x, item.y, item.z));
    }
    showPathLine(path1);
    vehicle.steering.clear();
    // const followPathBehavior = new YUKA.FollowPathBehavior(path1);
    // followPathBehavior.weight = 10;
    // vehicle.steering.add(followPathBehavior);

    const onPathBehavior = new YUKA.OnPathBehavior(path1, 0.1, 0.1);
    // onPathBehavior.weight = 10;
    vehicle.steering.add(onPathBehavior);

    const arriveBehavior = new YUKA.ArriveBehavior(to, 3, 3);
    arriveBehavior.weight = 1;
    vehicle.steering.add(arriveBehavior);
  }
});

// 网格加载器
const navMeshLoader = new YUKA.NavMeshLoader();
// 加载网格
let navMesh, vehicle;
// "./model/citymap1.gltf"
// "./model/MyCityRoad2.gltf"
navMeshLoader.load("./model/citymap.gltf").then(function (navigationMesh) {
  console.log(navigationMesh);
  navMesh = navigationMesh;

  vehicle = new CustomVechicle(navMesh);
  vehicle.maxSpeed = 5;
  vehicle.smoother = new YUKA.Smoother(30);
  entityManager.add(vehicle);
  loader.load("./model/car1.gltf", function (gltf) {
    console.log(gltf);
    const car = gltf.scene;
    // car.children[0].rotation.y = Math.PI / 2;
    car.children[0].scale.set(1, 1, 1);
    scene.add(car);
    vehicle.setRenderComponent(car, callback);
  });
});

// 加载hdr环境贴图
const hdrLoader = new RGBELoader();
hdrLoader.load("./textures/city.hdr", function (texture) {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
});

let mirrorGeometry = new THREE.PlaneGeometry(200, 100);
let groundMirror = new Reflector(mirrorGeometry, {
  clipBias: 0.003,
  textureWidth: window.innerWidth * window.devicePixelRatio,
  textureHeight: window.innerHeight * window.devicePixelRatio,
  color: 0x777777,
});
groundMirror.position.y = 0.1;
groundMirror.rotateX(-Math.PI / 2);
//scene.add(groundMirror);

window.addEventListener("dblclick", () => {
		  const fullScreenElement = document.fullscreenElement;
		  if (!fullScreenElement) {
		    //   双击控制屏幕进入全屏，退出全屏
		    // 让画布对象全屏
		    renderer.domElement.requestFullscreen();
		  } else {
		    //   退出全屏，使用document对象
		    document.exitFullscreen();
		  }
		  //   console.log(fullScreenElement);
		});

</script>

<template>
  <title>this is tiele</title>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
