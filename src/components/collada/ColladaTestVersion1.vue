<template>
    <div>
        <div id="container"></div>
    </div>
</template>

<script>
//官方的版本
    import * as THREE from '../../../node_modules/three/build/three.module';
    import Stats from '../../../node_modules/three/examples/jsm/libs/stats.module.js';
    import { TWEEN } from '../../../node_modules/three/examples/jsm/libs/tween.module.min.js';
    import { ColladaLoader } from '../../../node_modules/three/examples/jsm/loaders/ColladaLoader.js';
    export default {
        name: "ColladaTestVersion1",
        data(){
            return{
                container:null,
                stats:null,

                camera:null,
                scene:null,
                renderer:null,

                particleLight:null,

                kinematics:null,
                kinematicsTween:null,
                tweenParameters:{}
            }
        },
        mounted(){
          this.variabledefine()
        },
        methods: {
            variabledefine(){
                var dae;
                var kine = this.kinematics;
                const loader = new ColladaLoader();
                loader.load( '/models/daes/abb_irb52_7_120.dae', function ( collada ) {
                    dae = collada.scene;
                    dae.traverse( function ( child ) {
                        if ( child.isMesh ) {
                            // model does not have normals
                            child.material.flatShading = true;
                        }
                    } );
                    dae.scale.x = dae.scale.y = dae.scale.z = 10.0;
                    dae.updateMatrix();

                    kine = collada.kinematics;
                    this.init();
                    this.animate();

                } );
              this.kinematics =kine;
            },

            init() {
                this.container = document.createElement('div');
                document.body.appendChild(this.container);

                this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 2000);
                this.camera.position.set(2, 2, 3);

                this.scene = new THREE.Scene();

                // Grid

                const grid = new THREE.GridHelper(20, 20, 0x888888, 0xffffff);
                this.scene.add(grid);

                // Add the COLLADA

                this.scene.add(this.dae);

                this.particleLight = new THREE.Mesh(new THREE.SphereGeometry(4, 8, 8), new THREE.MeshBasicMaterial({color: 0xffffff}));
                this.scene.add(this.particleLight);

                // Lights

                const light = new THREE.HemisphereLight(0xffeeee, 0x111122);
                this.scene.add(light);

                const pointLight = new THREE.PointLight(0xffffff, 0.3);
                this.particleLight.add(pointLight);

                this.renderer = new THREE.WebGLRenderer();
                this.renderer.setPixelRatio(window.devicePixelRatio);
                this.renderer.setSize(window.innerWidth, window.innerHeight);
                this.container.appendChild(this.renderer.domElement);

                this.stats = new Stats();
                this.container.appendChild(this.stats.dom);

                this.setupTween();

                //

                window.addEventListener('resize', this.onWindowResize);

            },

            setupTween() {

                var prop;

                const duration = THREE.MathUtils.randInt(1000, 5000);

                const target = {};

                for (prop in this.kinematics.joints) {

                    if (this.kinematics.joints.hasOwnProperty.call(prop)) {

                        if (!this.kinematics.joints[prop].static) {

                            const joint = this.kinematics.joints[prop];

                            const old = this.tweenParameters[prop];

                            const position = old ? old : joint.zeroPosition;

                            this.tweenParameters[prop] = position;

                            target[prop] = THREE.MathUtils.randInt(joint.limits.min, joint.limits.max);

                        }

                    }

                }

                this.kinematicsTween = new TWEEN.Tween(this.tweenParameters).to(target, duration).easing(TWEEN.Easing.Quadratic.Out);

                this.kinematicsTween.onUpdate(function (object) {

                    for (const prop in this.kinematics.joints) {

                        if (this.kinematics.joints.hasOwnProperty.call(prop)) {

                            if (!this.kinematics.joints[prop].static) {

                                this.kinematics.setJointValue(prop, object[prop]);

                            }

                        }

                    }

                });

                this.kinematicsTween.start();

                setTimeout(this.setupTween, duration);

            },

            onWindowResize() {

                this.camera.aspect = window.innerWidth / window.innerHeight;
                this.camera.updateProjectionMatrix();

                this.renderer.setSize(window.innerWidth, window.innerHeight);

            },

            //

            animate() {

                requestAnimationFrame(this.animate);

                this.render();
                this.stats.update();
                TWEEN.update();

            },

            render() {

                const timer = Date.now() * 0.0001;

                this.camera.position.x = Math.cos(timer) * 20;
                this.camera.position.y = 10;
                this.camera.position.z = Math.sin(timer) * 20;

                this.camera.lookAt(0, 5, 0);

                this.particleLight.position.x = Math.sin(timer * 4) * 3009;
                this.particleLight.position.y = Math.cos(timer * 5) * 4000;
                this.particleLight.position.z = Math.cos(timer * 4) * 3009;

                this.renderer.render(this.scene, this.camera);

            }
        }
    }
</script>

<style scoped>

</style>