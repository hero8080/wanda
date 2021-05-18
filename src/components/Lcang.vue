<template>
  <div ref="lcang">
    <div class="loading" :style="{width: width*100 +'%'}"></div>
  </div>
</template>

<script>
  import * as THREE from 'three'
  //状态
  import Status from 'three/examples/jsm/libs/stats.module'
  //视图控制
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
  //模型加载器
  import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader'
  export default {
    name: "Lcang",
    data() {
      return {
        scene: null,
        camera: null,
        renderer: null,
        width:0
      }
    },
    mounted() {
         this.init()
    },
    methods:{
      init(){
        //创建场景
        this.scene = new THREE.Scene()
        //创建几何体
        let geometry = new THREE.BoxGeometry(100, 100, 100)
        //创建材质
        let material = new THREE.MeshLambertMaterial({color: 0x0000ff})
        //创建网格
        this.mesh = new THREE.Mesh(geometry, material) //网格模型对象Mesh
        // this.scene.add(this.mesh) //网格模型添加到场景中
        //加载模型
        let group = new THREE.Group()
        let gltfLoader = new GLTFLoader()
        gltfLoader.load('/static/scene/model.gltf',
          (gltf)=>{
            console.log(gltf)
            group.add(gltf.scene)
            this.scene.add(group)
            this.stopLoading()
          },
          (xhr)=>{
            this.width=(xhr.loaded/105528)/2
            //开启定时器加载
            this.startLoading()
          }
        )
        // 平行光
        let directionalLight1 = new THREE.DirectionalLight(0xffffff, 0.6)
        directionalLight1.position.set(400, 200, 300)
        this.scene.add(directionalLight1)
        let directionalLight2 = new THREE.DirectionalLight(0xffffff, 0.6)
        directionalLight2.position.set(-400, -200, -300)
        this.scene.add(directionalLight2)
        //环境光
        let ambient = new THREE.AmbientLight(0xffffff, 0.6)
        this.scene.add(ambient)

        let width = window.innerWidth
        let height = window.innerHeight
        //相机设置
        let k = width / height //Three.js输出的Canvas画布宽高比
        let s = 200 //控制相机渲染空间左右上下渲染范围，s越大，相机渲染范围越大
        this.camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
        this.camera.position.set(200, 300, 200) //相机在Three.js坐标系中的位置
        this.camera.lookAt(0, 0, 0) //相机指向Three.js坐标系原点
        //创建渲染器对象
        this.renderer = new THREE.WebGLRenderer({antialias: true})//开启锯齿
        this.renderer.setPixelRatio(window.devicePixelRatio)//设置设备像素比率,防止Canvas画布输出模糊。
        this.renderer.setSize(width, height) //设置渲染区域尺寸
        this.renderer.setClearColor(0xb9d3ff, 1) //设置背景颜色
        this.renderer.outputEncoding = THREE.sRGBEncoding
        this.$refs.lcang.appendChild(this.renderer.domElement) //body元素中插入canvas画布
        //辅助工具
        this.status=new Status()
        this.$refs.lcang.appendChild(this.status.domElement) //Status
        //视图控制
        new OrbitControls(this.camera, this.renderer.domElement)
        //执行渲染操作   指定场景、相机作为参数
        this.render()
      },
      render(){
        this.mesh.rotateY(0.10);//立方体绕y轴旋转动画
        this.renderer.render(this.scene, this.camera); //执行渲染操作
        this.status.update()
        requestAnimationFrame(this.render); //请求再次执行渲染函数render，渲染下一帧
      },
      startLoading(){
        clearInterval(this.timer)
        this.timer=setInterval(()=>{
          if(this.width<0.8){
            this.width+=0.1
          }else{
            clearInterval(this.timer)
          }
        },1000)
      },
      stopLoading(){
        clearInterval(this.timer)
        this.width=1
        setTimeout(()=>{
          this.width=0
        },300)
      }
    }
  }
</script>

<style scoped>
.loading{
  position: absolute;
  top: 0;
  left: 0;
  height: 3px;
  background-color: dodgerblue;
}
</style>
