---
title: threejs
date: 2021-12-29 13:44:58
comments: true #是否可评论
toc: true #是否显示文章目录
categories: #分类
  - vue #这是一级分类
tags:
  - vue3.0
  - threejs
---

### <center>vue 加载 3D 模型</center>

```javascript
$ npm install --save three
```

- _**注意：vue2 模型文件放在 static 下，vue3 模型文件放在 public 下**_

<!-- more -->

#### vue 加载 OBJ 模型

```javascript
<template>
    <div>
        <div id="container"></div>
    </div>
</template>
<script>
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js'
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader.js'
import { ColladaLoader } from 'three/examples/jsm/loaders/ColladaLoader'
// import { GUI } from 'three/examples/jsm/libs/dat.gui.module.js'
export default {
  setup() {
   let mesh
   let meshDAE
   let plane
   let camera
   let scene
   let gui
   let renderer
   // 创建场景
   const createScene = () => {
     scene = new Three.Scene()
   }
   // 加载OBJ模型
    const loadOBJ = () => {
      // const mtlLoader = new MTLLoader()
      const loader = new OBJLoader()
      // mtlLoader.load('/model/obj/file.mtl', function (material) {
      // 预加载
      // material.preload()
      // 设置当前加载的纹理
      // loader.setMaterials(material)
      loader.load('/modules/uploads_files_2792345_Koenigsegg.obj', loadedMesh => {
        // 创建材质 覆写材质可改变颜色
        /* const material = new Three.MeshStandardMaterial({
          color: '#F3F3F3',
          metalness: 0.4,
          roughness: 0.1
        })
        // 给几何体成员赋该材质
        loadedMesh.children.forEach(child => {
          child.material = material
          child.geometry.computeFaceNormals()
          child.geometry.computeVertexNormals()
        }) */
        mesh = loadedMesh
        // 添加到场景
        scene.add(mesh)
      }, (xhr) => {
        // 加载进度
        if (xhr.lengthComputable) {
          const percentComplete = xhr.loaded / xhr.total * 100
          console.log(Math.round(percentComplete, 2) + '%')
        }
      })
      //   })
      // 地板
      plane = new Three.Mesh(new Three.PlaneBufferGeometry(2000, 2000), new Three.MeshPhongMaterial({ color: 0xffffff, depthWrite: false }))
      plane.rotation.x = -Math.PI / 2
      plane.receiveShadow = true
      scene.add(plane)
    }
    // 加载dae模型
    const loadDAE = () => {
      const loader = new ColladaLoader()
      loader.load('/model/dae/file.dae', function (result) {
        meshDAE = result.scene.children[0].clone()
        scene.add(meshDAE)
      }, (xhr) => {
        // 加载进度
        if (xhr.lengthComputable) {
          const percentComplete = xhr.loaded / xhr.total * 100
          console.log(Math.round(percentComplete, 2) + '%--DEA')
        }
      })
    }
    // 初始化dat.GUI简化试验流程
    const initGui = () => {
      // 声明一个保存需求修改的相关数据的对象
      gui = {
      }
      const datGui = new GUI()
      console.log(datGui, gui)
      // 将设置属性添加到gui当中，gui.add(对象，属性，最小值，最大值）
    }
    // 创建光源
    const createLight = () => {
      // 环境光
      const ambientLight = new Three.AmbientLight(0xffffff, 0.3) // 创建环境光
      scene.add(ambientLight) // 将环境光添加到场景
      const spotLight = new Three.SpotLight('#ffffff', 0.7) // 创建聚光灯
      spotLight.position.set(1600, 1660, 100)
      spotLight.castShadow = true
      scene.add(spotLight)
    }
    // 创建相机
    const createCamera = () => {
      const width = window.innerWidth // 窗口宽度
      const height = window.innerHeight // 窗口高度
      const k = width / height // 窗口宽高比
      camera = new Three.PerspectiveCamera(75, k, 0.1, 1000)
      camera.position.set(400, 400, 100)
      camera.lookAt(scene.position)
      scene.add(camera)
    }
    // 创建渲染器
    const createRender = () => {
      renderer = new Three.WebGLRenderer({ antialias: true, alpha: true })
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.shadowMap.enabled = true // 显示阴影
      renderer.shadowMap.type = Three.PCFSoftShadowMap
      renderer.setClearColor('#000000') // 设置背景颜色
      document.body.appendChild(renderer.domElement)
    }
    const render = () => {
      // if (mesh) {
      //   mesh.rotation.y += 0.005
      // }
      renderer.render(scene, camera)
      requestAnimationFrame(render)
    }
    // 创建控件对象
    const createControls = () => {
      // eslint-disable-next-line no-new
      new OrbitControls(camera, renderer.domElement)
    }
       // 窗口自适应
    const resize = () => {
      window.addEventListener('resize', () => {
        // 初始化摄像机
        camera && (camera.aspect = window.innerWidth / window.innerHeight)
        camera && camera.updateProjectionMatrix()
        // 初始化渲染器尺寸
        renderer && renderer.setSize(window.innerWidth, window.innerHeight)
      })
    }
    // 初始化
    const init = () => {
      createScene() // 创建场景
      //   initGui() // 试验流程
      loadOBJ() // 加载OBJ模型
      //   loadDAE() // 加载DAE模型
      createCamera() // 创建相机
      createLight() // 创建光源
      createRender() // 创建渲染器
      createControls() // 创建控件对象
      render() // 渲染
      resize() // 自适应
    }
    init()
  }
}
</script>
```

#### vue 加载 GTL 模型

```javascript
<template>
    <div>
        <div id="container"></div>
    </div>
</template>
<script>
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { ColladaLoader } from 'three/examples/jsm/loaders/ColladaLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment.js';
import Stats from 'three/examples/jsm/libs/stats.module';
export default {
  setup() {
    let plane
    let camera
    let scene
    let renderer
    let clock
    let mixer
    let controls
    let stats
    // 创建场景
    const createScene = () => {
      scene = new Three.Scene()
      scene.background = new Three.Color(0xbfe3dd);
      const pmremGenerator = new Three.PMREMGenerator(renderer);
      scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
      clock = new Three.Clock();
      stats = new Stats();
    }
    // 加载GTL模型
    const loadGTL = () => {
      const dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderPath('/js/gltf/');
      const loader = new GLTFLoader();
      loader.setDRACOLoader(dracoLoader);
      loader.load('/modules/LittlestTokyo.glb', function (gltf) {
        const model = gltf.scene;
        model.position.set(1, 1, 0);
        model.scale.set(0.01, 0.01, 0.01);
        scene.add(model);
        mixer = new Three.AnimationMixer(model);
        mixer.clipAction(gltf.animations[0]).play();
        render();
      }, undefined, function (error) {
        console.error(error);
      });
      // 地板
      plane = new Three.Mesh(new Three.PlaneBufferGeometry(500, 400), new Three.MeshPhongMaterial({ color: 0xa08c66, depthWrite: false }))
      plane.rotation.x = -Math.PI / 2
      plane.receiveShadow = true
      scene.add(plane)
    }
    // 创建光源
    const createLight = () => {
      // 环境光
      const ambientLight = new Three.AmbientLight(0xffffff, 0.3) // 创建环境光
      scene.add(ambientLight) // 将环境光添加到场景
      const spotLight = new Three.SpotLight('#ffffff', 0.7) // 创建聚光灯
      spotLight.position.set(1600, 1660, 100)
      spotLight.castShadow = true
      scene.add(spotLight)
    }
    // 创建相机
    const createCamera = () => {
      const width = window.innerWidth // 窗口宽度
      const height = window.innerHeight // 窗口高度
      const k = width / height // 窗口宽高比
      camera = new Three.PerspectiveCamera(40, k, 1, 100);
      camera.position.set(5, 2, 8);
      camera.lookAt(scene.position)
      scene.add(camera)
    }
    // 创建渲染器
    const createRender = () => {
      renderer = new Three.WebGLRenderer({ antialias: true, alpha: true })
      renderer.setSize(window.innerWidth, window.innerHeight - 84)
      renderer.shadowMap.enabled = true // 显示阴影
      renderer.shadowMap.type = Three.PCFSoftShadowMap
      renderer.setClearColor('#000000') // 设置背景颜色
      document.body.appendChild(renderer.domElement)
    }
    const render = () => {
      const delta = clock.getDelta();
      mixer.update(delta);
      controls && controls.update();
      stats.update();
      renderer && renderer.render(scene, camera)
      requestAnimationFrame(render)
    }
    // 创建控件对象
    const createControls = () => {
      // eslint-disable-next-line no-new
      controls = new OrbitControls(camera, renderer.domElement);
      controls.target.set(0, 0.5, 0);
      controls.update();
      controls.enablePan = false;
      controls.enableDamping = true;
    }
    // 窗口自适应
    const resize = () => {
      window.addEventListener('resize', () => {
        // 初始化摄像机
        camera && (camera.aspect = window.innerWidth / window.innerHeight)
        camera && camera.updateProjectionMatrix()
        // 初始化渲染器尺寸
        renderer && renderer.setSize(window.innerWidth, window.innerHeight - 84)
      })
    }
    // 初始化
    const init = () => {
      createRender() // 创建渲染器
      createScene() // 创建场景
      loadGTL() // 加载GTL模型
      createCamera() // 创建相机
      createLight() // 创建光源
      createControls() // 创建控件对象
      resize() // 自适应
    }
    init()
  }
}
</script>
```

#### 模型交互

```javascript
// 用户交互
let selectObject = (event) => {
  event.preventDefault();
  if (event.button != 0) return;
  const mouse = new Three.Vector3();
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  console.log("mouse: ", mouse);
  const raycaster = new Three.Raycaster();
  raycaster.setFromCamera(mouse, camera);
  const intersected = raycaster.intersectObjects(scene.children, false);
  console.log(intersected);
  if (intersected.length) {
    const found = intersected[0];
    const faceIndex = found.faceIndex;
    const geometry = found.object.geometry;
    found.object.material.color.set(0xff0000);
    const modelID = found.object.modelID;
  }
};
window.onpointerdown = selectObject;
```

#### 场景销毁

```javascript
// 销毁
const removeCube = () => {
  var allChildren = scene.children;
  if (allChildren.length > 0) {
    renderer.setSize(0, 0);
    allChildren.forEach((item) => {
      scene.remove(item);
    });
  }
  scene.traverse((child) => {
    if (child.material) {
      child.material.dispose();
    }
    if (child.geometry) {
      child.geometry.dispose();
    }
    child = null;
  });
  renderer.forceContextLoss();
  renderer.dispose();
  scene.clear();
  scene = null;
  camera = null;
  controls = null;
  renderer.domElement = null;
  renderer = null;
};
```
