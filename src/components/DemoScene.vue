<template>
    <div>
        <div id="renderer"></div>
        <button @click="startAnimations">Start</button>      
        <button @click="stopAnimations">Stop</button>      
    </div>
  
</template>

<script>

import * as THREE from 'three';
import { OrbitControls } from 'three-orbitcontrols-ts'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

let scene, camera, renderer, controls, model, hemiLight, spotLight, clock;

export default {
    data () {
        return {
            mixer: null,
            clips: null,
            cameras: null,
            camera: null
        }
    },
    mounted () {
        this.init();
        this.animate();
    },
    methods : {
        init: function () {
            clock = new THREE.Clock();

            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xdddddd);

            // camera = new THREE.PerspectiveCamera( 40, window.innerWidth / window.innerHeight, 1, 100 );
            // camera.position.set(20, 15, 22);
            // camera.rotation.set(-0.60,0.6,0.37);

            // controls = new OrbitControls(camera);

            hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 4);
            scene.add(hemiLight);

            spotLight = new THREE.SpotLight(0xffffff, 4);
            spotLight.castShadow = true;
            spotLight.shadow.bias = -0.0001;
            spotLight.shadow.mapSize.width = 1024*4;
            spotLight.shadow.mapSize.height = 1024*4;
            scene.add(spotLight);
            
            scene.add(new THREE.AxesHelper(500));

            renderer = new THREE.WebGLRenderer();
            renderer.toneMapping = THREE.CineonToneMapping;
            renderer.toneMappingExposure = 4.3;
            renderer.shadowMap.enabled = true;
            renderer.setSize(window.innerWidth, window.innerHeight);
            
            this.$el.appendChild(renderer.domElement);

            new GLTFLoader().load('models/cube.gltf', GLTF => {
                let root = GLTF.scene;
    
                // Create an AnimationMixer, and get the list of AnimationClip instances
                this.mixer = new THREE.AnimationMixer( GLTF.scene );
                this.clips = GLTF.animations;
                this.cameras = GLTF.cameras;

                let camera = this.cameras[0];

                this.camera = camera;

                // Play all animations
                let clips = this.clips;
                for(let i = 0; i < clips.length; i++) {
                    this.mixer.clipAction(clips[i]).play();
                }

                let models = GLTF.scene.children;

                for(const child of models) {
                    if(child.isMesh) {
                        child.traverse(n => {
                            n.castShadow = true;
                            n.receiveShadow = true;
                            if(n.material.map) {
                                n.material.map.anisotropy = 16;
                            }
                        });
                    }
                }        

                scene.add(root);       

                this.animate();
            });
        },
        animate: function () {
            renderer.render(scene, this.camera);
            spotLight.position.set(
                this.camera.position.x + 10,
                this.camera.position.y + 10,
                this.camera.position.z + 10, 
            );
            var delta = 0.75 * clock.getDelta();

            // At the first frame the mixer is not existing
            if(this.mixer !== null) {
                this.mixer.update( delta );
            }
    
            requestAnimationFrame(this.animate);
        },
        startAnimations: function () {
            let clips = this.clips;
            for(let i = 0; i < clips.length; i++) {
                this.mixer.clipAction(clips[i]).play();
            }
        },

        stopAnimations: function () {
            let clips = this.clips;
            for(let i = 0; i < clips.length; i++) {
                this.mixer.clipAction(clips[i]).stop();
            }
        }
    }
}
</script>

<style lang="scss">
    body {
        margin: 0;
        padding: 0;
    }

    canvas {
        height: 100vh;
        max-width: 80rem;
    }
</style>