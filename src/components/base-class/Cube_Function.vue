<template>
    <div>
        <div id="container"></div>
    </div>
</template>

<script>
import * as Three from 'three'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
import {AxesHelper} from 'three/src/helpers/AxesHelper'

export default {
    name: "base-class",
    data() {
        return {
            Scene:{
                scene: null,
                mesh: null,
                camera: null,
                renderer: null,
                container: null
            },
            Axis:{
                axis:null
            },
            Line:{
                material:null,
                //轴线的起始点和结束点
                PointStart:null,
                PointEnd:null,
            }

        }
    },
    methods: {
        InitScene() {
            //创建画布，创建几何体，材质，加入到Mesh材质包中，创建相机，相机参数，渲染，渲染的窗口大小，控制
            this.scene = new Three.Scene();
            this.container = document.getElementById('container');
        },
        InitAxis(){
            //param 轴长度
           // this.axis = new AxesHelper(0.5);
            this.scene.add(new AxesHelper(0.5));
        },
        InitRotationLine(){
            //线的材质
            this.material = new Three.LineBasicMaterial( { color: 0xffffff } );
            //旋转轴线的起，末点
            this.PointStart = new Three.Vector3(0,0,0);
            this.PointEnd = new Three.Vector3(2/Math.sqrt(29),3/Math.sqrt(29),4/Math.sqrt(29));

            const points = [];
            points.push(this.PointStart);
            points.push(this.PointEnd);
            const geometry = new Three.BufferGeometry().setFromPoints( points );
            const line = new Three.Line( geometry, this.material );
            this.scene.add(line);
        },
        InitMesh() {
            //http://www.yanhuangxueyuan.com/Three.js/
            //教程：https://www.bilibili.com/read/cv16116533/
            //Lambert代码：https://blog.csdn.net/qw8704149/article/details/109522790

            const geometry = new Three.BoxGeometry(0.1, 0.1, 0.1,50,50,50);
            //const geometry = new Three.SphereGeometry(0.1,100,100,0);

            //根据法向量计算物体表面的颜色，无法更改
            const material = new Three.MeshNormalMaterial();

            //纯色,基本看不出是个啥物体
            //const material = new Three.MeshBasicMaterial(0xffffff);

            //暗淡的非光泽表面的材质，没有镜面高光，并且会对光源做出反应(没光就是黑块，有光才能看见)
            //const material = new Three.MeshLambertMaterial({color: 0xff0000});

            //物体本身无法渲染，需要将物体加入到网格中，对网格进行渲染
            this.mesh = new Three.Mesh(geometry, material);
            //网格加入到背景中
            this.scene.add(this.mesh);



        },
        InitLight(){
            //AmbientLight环境光会均匀的照亮场景中的所有物体。环境光不能用来投射阴影，因为它没有方向。
            this.ambient = new Three.AmbientLight(0x123987);
            this.scene.add(this.ambient);

            //点光源（PointLight）从一个点向各个方向发射的光源。一个常见的例子是模拟一个灯泡发出的光。
            this.point = new Three.PointLight(0xff0000);
            this.point.position.x = 1;
            this.point.position.y =1;
            this.point.position.z = 1;
            this.scene.add(this.point);

            //聚光灯（SpotLight）光线从一个点沿一个方向射出，随着光线照射的变远，光线圆锥体的尺寸也逐渐增大
            const spotLight = new Three.SpotLight( 0xffffff );
            spotLight.position.set( 100, 1000, 100 );

            spotLight.castShadow = true;

            spotLight.shadow.mapSize.width = 1024;
            spotLight.shadow.mapSize.height = 1024;

            spotLight.shadow.camera.near = 500;
            spotLight.shadow.camera.far = 4000;
            spotLight.shadow.camera.fov = 30;

            this.scene.add( spotLight );


        },
        InitCamera() {

            /**
             * @comprehension 透视点相机，远小近大
             * @param fov：视角大小，视角越大东西越小，因为能看x见的越多
             * @param aspect：长宽比例；
             * @param near:距离摄像机多近的位置开始渲染；
             * @param far:相机从它所处位置能够看的多远}
             * @type {PerspectiveCamera}
             */
            this.camera = new Three.PerspectiveCamera(8, this.container.clientWidth / this.container.clientHeight, 0.1, 1000);
            /**
             * @comprehension 正交相机，多个物体在面前的大小相同
             * @param （左，右，上，下）边界；近端距离；远端距离
             * @type {OrthographicCamera}
             */
            //this.Camera = new Three.OrthographicCamera();

            //相机位置
            this.camera.position.set(5,5,5);
            //z轴朝上
            this.camera.up = new Three.Vector3(0,0,1);
            this.camera.lookAt(this.scene);
        },
        InitRender() {
            this.renderer = new Three.WebGLRenderer();
            this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
            this.container.appendChild(this.renderer.domElement);
        },
        Animate() {
            requestAnimationFrame(this.Animate);

            //this.mesh.rotateY(0.01);
            //绕轴转，param1 应为单位向量，param2 应为角度
            this.mesh.rotateOnAxis(this.PointEnd,0.1);

            this.renderer.render(this.scene, this.camera);
        },
        Control() {
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
        }
    },

    mounted() {
        this.InitScene();//场景
        this.InitAxis();//坐标轴
        this.InitRotationLine();//绕哪个轴线转动
        this.InitMesh();//网格，里面包含图形和其材质
        this.InitLight();//光源
        this.InitCamera();//相机
        this.InitRender();//渲染
        this.Control();//鼠标操控
        //动画渲染必须在最后
        this.Animate();//对以上进行渲染
    },
}
</script>

<style scoped>
#container {
    height: 800px;
    width: 750px;
}
</style>