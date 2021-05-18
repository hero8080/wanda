<template>
  <div ref="wanda">

  </div>
</template>

<script>
  import * as THREE from 'three'
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
  import * as GEOLIB from 'geolib'

  export default {
    name: 'HelloWorld',
    data() {
      return {
        scene: null,
        camera: null,
        renderer: null,
        controls: null,
        center:[117.1359079,36.680099],
        matBuildings:null,
        matBuildings2:null
      }
    },
    mounted() {
      this.init()
      this.getData()
    },
    methods: {
      //初始化
      init() {
        //场景
        this.scene = new THREE.Scene()
        this.scene.background = new THREE.Color(0x222222)
        //摄像机
        this.camera = new THREE.PerspectiveCamera(25, window.innerWidth / window.innerHeight, 1, 1000);
        this.camera.position.set(0, 20, 100)
        let light1 = new THREE.AmbientLight(0xfafafa, 0.25)
        let light2 = new THREE.PointLight(0xfafafa, 0.4)
        let light3 = new THREE.PointLight(0xfafafa, 0.4)
        light2.position.set(200, 90, 40)
        light3.position.set(200, 90, -40)
        //加入灯光
        this.scene.add(light1)
        this.scene.add(light2)
        this.scene.add(light3)
        //参考网格
        let gridHelper = new THREE.GridHelper(60, 30, new THREE.Color(0x555555), new THREE.Color(0x333333));
        this.scene.add(gridHelper);
        //立方体
        // let geometry = new THREE.BoxGeometry(10, 10, 10);
        // let material = new THREE.MeshBasicMaterial({color: 0xff0000});
        // let cube = new THREE.Mesh(geometry, material);
        // this.scene.add(cube);
        //渲染器
        this.renderer = new THREE.WebGLRenderer({antialias: true})
        this.renderer.setPixelRatio(window.devicePixelRatio)
        this.renderer.setSize(window.innerWidth, window.innerHeight)
        this.$refs.wanda.appendChild(this.renderer.domElement)
        //控制器
        this.controls = new OrbitControls(this.camera, this.renderer.domElement)
        //动画
        this.controls.update()
        this.update()
        //立方体材质
        this.matBuildings=new THREE.MeshPhongMaterial()
        this.matBuildings2=new THREE.MeshBasicMaterial({color: 0xff0000});
      },
      update() {
        requestAnimationFrame(this.update)
        this.controls.update()
        this.renderer.render(this.scene, this.camera)
      },
      getData() {
        let data = import('./export.json')
        data.then(data => {
          this.createBuilding(data)
        })
      },
      createBuilding(data) {
        let features = data.features
        features.map(item => {
          //创建建筑物
          // if (item.geometry && (item.geometry.type=='Polygon'||item.geometry.type=='MultiPolygon')) {

          if (item.geometry && item.geometry.type==='Polygon') {
          // if (item.properties && item.properties['building']) {
            this.addBuilding(item.geometry.coordinates,item.properties, item.properties['building:levels'])
          }

        })
      },
      addBuilding(data,info,height=1) {
        let shape
        let holes=[]
        data.map((item,index)=>{
          if(index==0){
            shape = this.getShape(item)
          }else{
            holes.push(this.getShape(item))
          }
        })
        holes.length>0&&holes.map(item=>{
          shape.holes.push(item)
        })
        let mesh=null
        let geometry = new THREE.ExtrudeBufferGeometry( shape, {
          curveSegments: 1,
          depth: 0.3*height,
          bevelEnabled: false
        });
        geometry.computeBoundingBox()
        geometry.rotateX(Math.PI/2)
        geometry.rotateZ(Math.PI)
        geometry.translate(0,0,-10)
        mesh = new THREE.Mesh(geometry,this.matBuildings)
        this.scene.add(mesh)
      },
      getShape(points){
        let shape = new THREE.Shape()
        points.map((item,index)=>{
          let elp = this.GPSRelativePosition(item, this.center)
          if(index==0){
            shape.moveTo(elp[0],elp[1])
          }else{
            shape.lineTo(elp[0],elp[1])
          }
        })
        return shape
      },
      GPSRelativePosition(objPosi, centerPosi) {
        // Get GPS distance
        let dis = GEOLIB.getDistance(objPosi, centerPosi)
        // Get bearing angle
        let bearing = GEOLIB.getRhumbLineBearing(objPosi, centerPosi)
        // Calculate X by centerPosi.x + distance * cos(rad)
        let x = centerPosi[0] + (dis * Math.cos(bearing * Math.PI / 180))
        // Calculate Y by centerPosi.y + distance * sin(rad)
        let y = centerPosi[1] + (dis * Math.sin(bearing * Math.PI / 180))
        // Reverse X (it work)
        return [-x / 100, y / 100]
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
