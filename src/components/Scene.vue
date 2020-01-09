<template>
  <section class="scene" ref="scene"></section>
</template>

<script>
    import * as THREE from 'three'
    import GLTFLoader from 'three-gltf-loader'
    const OrbitControls = require('three-orbitcontrols')
    
    export default {
        name: 'Scene',
        data() {
            return {
                container: null,
                camera: null,
                controls: null,
                scene: null,
                renderer: null,
                start: Date.now(),
                shaderMaterial: null,
                shaderMaterial1: null,
                shaderMaterial2: null
            }
        },
        methods: {
            init() {
                this.container = this.$refs.scene
                this.scene = new THREE.Scene()
                
                this.createCamera()
                this.createControls()
                this.createLights()
                this.createRenderer()
                this.loadModel()
                
                this.renderer.setAnimationLoop(() => {
                    this.update()
                    this.render()
                })

                window.addEventListener( 'resize', this.resizeWindowHandler, false )
            },
            createCamera() {
                this.camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 2000 )
                this.camera.position.set( 0, 0, 20 )
            },
            createControls() {
                this.controls = new OrbitControls( this.camera, this.container )
                this.controls.enableZoom = true
                this.controls.minDistance = 20
                this.controls.maxDistance = 25
                this.controls.enablePan = false
                this.controls.enableDamping = true
                this.controls.dampingFactor = 0.35
                this.controls.minPolarAngle = 1.5708
                this.controls.maxPolarAngle = 1.5708
                this.controls.autoRotate = false
                this.controls.autoRotateSpeed = 0.25
            },
            createLights() {
                const ambientLight = new THREE.AmbientLight( 0xffffff, 1 )
                
                const light_1 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_1.position.set( 0, 5, 0 )

                const helper_1 = new THREE.DirectionalLightHelper( light_1, 5 )

                const light_2 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_2.position.set( 5, 0, 0 )

                const helper_2 = new THREE.DirectionalLightHelper( light_2, 5 )

                const light_3 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_3.position.set( -5, 0, 0 )

                const helper_3 = new THREE.DirectionalLightHelper( light_3, 5 )
                
                const light_4 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_4.position.set( 0, 0, 5 )

                const helper_4 = new THREE.DirectionalLightHelper( light_4, 5 )
                
                const light_5 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_5.position.set( 0, 0, -5 )

                const helper_5 = new THREE.DirectionalLightHelper( light_5, 5 )

                const light_6 = new THREE.DirectionalLight( 0xdddddd, 0.5 )
                light_6.position.set( 0, -5, 0 )

                const helper_6 = new THREE.DirectionalLightHelper( light_6, 5 )
                
                this.scene.add(
                    ambientLight,
                    light_1, //helper_1,
                    light_2, //helper_2,
                    light_3, //helper_3,
                    //light_4, helper_4,
                    // light_5, helper_5,
                    // light_6, helper_6
                )
            },
            createRenderer() {
                this.renderer = new THREE.WebGLRenderer( { antialias: true, alpha: true } )
                this.renderer.setSize( window.innerWidth, window.innerHeight )
                this.renderer.setPixelRatio( window.devicePixelRatio )
                this.renderer.autoClear = false;
                this.renderer.setClearColor(0x000000, 0.0)
                this.container.appendChild( this.renderer.domElement )
            },
            loadModel() {
                const loader = new GLTFLoader()

                loader.load(
                    '/static/models/bust/bbb.glb',
                    ( gltf ) => {
                        gltf.scene.scale.set( 0.2, 0.2, 0.2 )
                        gltf.scene.position.set( -8, -5, 0 )

                        let shaderMaterial = new THREE.ShaderMaterial({
                            uniforms: {
                                time: {
                                    type: "f",
                                    value: 0.0
                                },
                                u_texture: {
                                    type: 't',
                                    //value: THREE.ImageUtils.loadTexture( '/static/images/metall.jpg' )
                                    value: gltf.scene.children[0].material.normalMap
                                }
                            },
                            vertexShader: this.vertexShader(),
                            fragmentShader: this.fragmentShader()
                        })

                        let material = gltf.scene.children[0].material

                        material.onBeforeCompile = (shader) => {
                            shader.uniforms.time = { value: 0 };
                            shader.uniforms.disp = { value: 1 };

                            shader.vertexShader = 'uniform float time;\nuniform float disp;\n' + shader.vertexShader;
                            shader.vertexShader = shader.vertexShader.replace(
                                '#include <begin_vertex>',
                                [
                                    'float theta = sin( time + position.y*0.25 ) / 8.0;',
                                    'float c = 1.*cos( theta ) ;',
                                    'float s = 8.*sin(theta) ;',
                                    'mat3 m = mat3( c*disp+1.*(1.-disp), 0., s*disp, 0., 1., 0., -s*disp, 0., c*disp+1.*(1.-disp) );',
                                    //'mat3 m = mat3( 1, 0, 0, 0, 1, 0, 0, 0, 1 );',
                                    'vec3 transformed = vec3( position ) * m;',
                                    'vNormal = vNormal * m;'
                                ].join( '\n' )
                            );

					        this.shaderMaterial = shader;
                        }
                        
                        //gltf.scene.children[0].material = shaderMaterial

                        //console.log(gltf.scene.children[0].material)

                        this.scene.add(gltf.scene)
                    },
                    ( xhr ) => {
                        console.log( `${( xhr.loaded / xhr.total * 100 )}% loaded` )
                    },
                    ( error ) => {
                        console.error( 'An error happened', error )
                    },
                )

                loader.load(
                    '/static/models/bust/bbb.glb',
                    ( gltf ) => {
                        gltf.scene.scale.set( 0.2, 0.2, 0.2 )
                        gltf.scene.position.set( 0, -5, 0 )

                        let shaderMaterial = new THREE.ShaderMaterial({
                            uniforms: {
                                time: {
                                    type: "f",
                                    value: 0.0
                                },
                                u_texture: {
                                    type: 't',
                                    //value: THREE.ImageUtils.loadTexture( '/static/images/metall.jpg' )
                                    value: gltf.scene.children[0].material.normalMap
                                }
                            },
                            vertexShader: this.vertexShader(),
                            fragmentShader: this.fragmentShader()
                        })

                        let material = gltf.scene.children[0].material

                        material.onBeforeCompile = (shader) => {
                            shader.uniforms.time = { value: 0 };
                            shader.uniforms.disp = { value: 1 };

                            shader.vertexShader = 'uniform float time;\nuniform float disp;\n' + shader.vertexShader;
                            shader.vertexShader = shader.vertexShader.replace(
                                '#include <begin_vertex>',
                                [
                                    'float theta = sin( time + position.y*0.25 ) / 8.0;',
                                    'float c = 1.*cos( theta ) ;',
                                    'float s = 8.*sin(theta) ;',
                                    'mat3 m = mat3( c*disp+1.*(1.-disp), 0., s*disp, 0., 1., 0., -s*disp, 0., c*disp+1.*(1.-disp) );',
                                    //'mat3 m = mat3( 1, 0, 0, 0, 1, 0, 0, 0, 1 );',
                                    'vec3 transformed = vec3( position ) * m;',
                                    'vNormal = vNormal * m;'
                                ].join( '\n' )
                            );

					        this.shaderMaterial1 = shader;
                        }
                        
                        //gltf.scene.children[0].material = shaderMaterial

                        //console.log(gltf.scene.children[0].material)

                        this.scene.add(gltf.scene)
                    },
                    ( xhr ) => {
                        console.log( `${( xhr.loaded / xhr.total * 100 )}% loaded` )
                    },
                    ( error ) => {
                        console.error( 'An error happened', error )
                    },
                )

                loader.load(
                    '/static/models/bust/bbb.glb',
                    ( gltf ) => {
                        gltf.scene.scale.set( 0.2, 0.2, 0.2 )
                        gltf.scene.position.set( 8, -5, 0 )

                        let shaderMaterial = new THREE.ShaderMaterial({
                            uniforms: {
                                time: {
                                    type: "f",
                                    value: 0.0
                                },
                                u_texture: {
                                    type: 't',
                                    //value: THREE.ImageUtils.loadTexture( '/static/images/metall.jpg' )
                                    value: gltf.scene.children[0].material.normalMap
                                }
                            },
                            vertexShader: this.vertexShader(),
                            fragmentShader: this.fragmentShader()
                        })

                        let material = gltf.scene.children[0].material

                        material.onBeforeCompile = (shader) => {
                            shader.uniforms.time = { value: 0 };
                            shader.uniforms.disp = { value: 1 };

                            shader.vertexShader = 'uniform float time;\nuniform float disp;\n' + shader.vertexShader;
                            shader.vertexShader = shader.vertexShader.replace(
                                '#include <begin_vertex>',
                                [
                                    'float theta = sin( time + position.y*0.25 ) / 8.0;',
                                    'float c = 1.*cos( theta ) ;',
                                    'float s = 8.*sin(theta) ;',
                                    'mat3 m = mat3( c*disp+1.*(1.-disp), 0., s*disp, 0., 1., 0., -s*disp, 0., c*disp+1.*(1.-disp) );',
                                    //'mat3 m = mat3( 1, 0, 0, 0, 1, 0, 0, 0, 1 );',
                                    'vec3 transformed = vec3( position ) * m;',
                                    'vNormal = vNormal * m;'
                                ].join( '\n' )
                            );

					        this.shaderMaterial2 = shader;
                        }
                        
                        //gltf.scene.children[0].material = shaderMaterial

                        //console.log(gltf.scene.children[0].material)

                        this.scene.add(gltf.scene)
                    },
                    ( xhr ) => {
                        console.log( `${( xhr.loaded / xhr.total * 100 )}% loaded` )
                    },
                    ( error ) => {
                        console.error( 'An error happened', error )
                    },
                )
            },
            vertexShader() {
                return `
                    vec3 mod289(vec3 x)
                    {
                    return x - floor(x * (1.0 / 289.0)) * 289.0;
                    }

                    vec4 mod289(vec4 x)
                    {
                    return x - floor(x * (1.0 / 289.0)) * 289.0;
                    }

                    vec4 permute(vec4 x)
                    {
                    return mod289(((x*34.0)+1.0)*x);
                    }

                    vec4 taylorInvSqrt(vec4 r)
                    {
                    return 1.79284291400159 - 0.85373472095314 * r;
                    }

                    vec3 fade(vec3 t) {
                    return t*t*t*(t*(t*6.0-15.0)+10.0);
                    }

                    // Classic Perlin noise
                    float cnoise(vec3 P)
                    {
                    vec3 Pi0 = floor(P); // Integer part for indexing
                    vec3 Pi1 = Pi0 + vec3(1.0); // Integer part + 1
                    Pi0 = mod289(Pi0);
                    Pi1 = mod289(Pi1);
                    vec3 Pf0 = fract(P); // Fractional part for interpolation
                    vec3 Pf1 = Pf0 - vec3(1.0); // Fractional part - 1.0
                    vec4 ix = vec4(Pi0.x, Pi1.x, Pi0.x, Pi1.x);
                    vec4 iy = vec4(Pi0.yy, Pi1.yy);
                    vec4 iz0 = Pi0.zzzz;
                    vec4 iz1 = Pi1.zzzz;

                    vec4 ixy = permute(permute(ix) + iy);
                    vec4 ixy0 = permute(ixy + iz0);
                    vec4 ixy1 = permute(ixy + iz1);

                    vec4 gx0 = ixy0 * (1.0 / 7.0);
                    vec4 gy0 = fract(floor(gx0) * (1.0 / 7.0)) - 0.5;
                    gx0 = fract(gx0);
                    vec4 gz0 = vec4(0.5) - abs(gx0) - abs(gy0);
                    vec4 sz0 = step(gz0, vec4(0.0));
                    gx0 -= sz0 * (step(0.0, gx0) - 0.5);
                    gy0 -= sz0 * (step(0.0, gy0) - 0.5);

                    vec4 gx1 = ixy1 * (1.0 / 7.0);
                    vec4 gy1 = fract(floor(gx1) * (1.0 / 7.0)) - 0.5;
                    gx1 = fract(gx1);
                    vec4 gz1 = vec4(0.5) - abs(gx1) - abs(gy1);
                    vec4 sz1 = step(gz1, vec4(0.0));
                    gx1 -= sz1 * (step(0.0, gx1) - 0.5);
                    gy1 -= sz1 * (step(0.0, gy1) - 0.5);

                    vec3 g000 = vec3(gx0.x,gy0.x,gz0.x);
                    vec3 g100 = vec3(gx0.y,gy0.y,gz0.y);
                    vec3 g010 = vec3(gx0.z,gy0.z,gz0.z);
                    vec3 g110 = vec3(gx0.w,gy0.w,gz0.w);
                    vec3 g001 = vec3(gx1.x,gy1.x,gz1.x);
                    vec3 g101 = vec3(gx1.y,gy1.y,gz1.y);
                    vec3 g011 = vec3(gx1.z,gy1.z,gz1.z);
                    vec3 g111 = vec3(gx1.w,gy1.w,gz1.w);

                    vec4 norm0 = taylorInvSqrt(vec4(dot(g000, g000), dot(g010, g010), dot(g100, g100), dot(g110, g110)));
                    g000 *= norm0.x;
                    g010 *= norm0.y;
                    g100 *= norm0.z;
                    g110 *= norm0.w;
                    vec4 norm1 = taylorInvSqrt(vec4(dot(g001, g001), dot(g011, g011), dot(g101, g101), dot(g111, g111)));
                    g001 *= norm1.x;
                    g011 *= norm1.y;
                    g101 *= norm1.z;
                    g111 *= norm1.w;

                    float n000 = dot(g000, Pf0);
                    float n100 = dot(g100, vec3(Pf1.x, Pf0.yz));
                    float n010 = dot(g010, vec3(Pf0.x, Pf1.y, Pf0.z));
                    float n110 = dot(g110, vec3(Pf1.xy, Pf0.z));
                    float n001 = dot(g001, vec3(Pf0.xy, Pf1.z));
                    float n101 = dot(g101, vec3(Pf1.x, Pf0.y, Pf1.z));
                    float n011 = dot(g011, vec3(Pf0.x, Pf1.yz));
                    float n111 = dot(g111, Pf1);

                    vec3 fade_xyz = fade(Pf0);
                    vec4 n_z = mix(vec4(n000, n100, n010, n110), vec4(n001, n101, n011, n111), fade_xyz.z);
                    vec2 n_yz = mix(n_z.xy, n_z.zw, fade_xyz.y);
                    float n_xyz = mix(n_yz.x, n_yz.y, fade_xyz.x);
                    return 2.2 * n_xyz;
                    }

                    // Classic Perlin noise, periodic variant
                    float pnoise(vec3 P, vec3 rep)
                    {
                    vec3 Pi0 = mod(floor(P), rep); // Integer part, modulo period
                    vec3 Pi1 = mod(Pi0 + vec3(1.0), rep); // Integer part + 1, mod period
                    Pi0 = mod289(Pi0);
                    Pi1 = mod289(Pi1);
                    vec3 Pf0 = fract(P); // Fractional part for interpolation
                    vec3 Pf1 = Pf0 - vec3(1.0); // Fractional part - 1.0
                    vec4 ix = vec4(Pi0.x, Pi1.x, Pi0.x, Pi1.x);
                    vec4 iy = vec4(Pi0.yy, Pi1.yy);
                    vec4 iz0 = Pi0.zzzz;
                    vec4 iz1 = Pi1.zzzz;

                    vec4 ixy = permute(permute(ix) + iy);
                    vec4 ixy0 = permute(ixy + iz0);
                    vec4 ixy1 = permute(ixy + iz1);

                    vec4 gx0 = ixy0 * (1.0 / 7.0);
                    vec4 gy0 = fract(floor(gx0) * (1.0 / 7.0)) - 0.5;
                    gx0 = fract(gx0);
                    vec4 gz0 = vec4(0.5) - abs(gx0) - abs(gy0);
                    vec4 sz0 = step(gz0, vec4(0.0));
                    gx0 -= sz0 * (step(0.0, gx0) - 0.5);
                    gy0 -= sz0 * (step(0.0, gy0) - 0.5);

                    vec4 gx1 = ixy1 * (1.0 / 7.0);
                    vec4 gy1 = fract(floor(gx1) * (1.0 / 7.0)) - 0.5;
                    gx1 = fract(gx1);
                    vec4 gz1 = vec4(0.5) - abs(gx1) - abs(gy1);
                    vec4 sz1 = step(gz1, vec4(0.0));
                    gx1 -= sz1 * (step(0.0, gx1) - 0.5);
                    gy1 -= sz1 * (step(0.0, gy1) - 0.5);

                    vec3 g000 = vec3(gx0.x,gy0.x,gz0.x);
                    vec3 g100 = vec3(gx0.y,gy0.y,gz0.y);
                    vec3 g010 = vec3(gx0.z,gy0.z,gz0.z);
                    vec3 g110 = vec3(gx0.w,gy0.w,gz0.w);
                    vec3 g001 = vec3(gx1.x,gy1.x,gz1.x);
                    vec3 g101 = vec3(gx1.y,gy1.y,gz1.y);
                    vec3 g011 = vec3(gx1.z,gy1.z,gz1.z);
                    vec3 g111 = vec3(gx1.w,gy1.w,gz1.w);

                    vec4 norm0 = taylorInvSqrt(vec4(dot(g000, g000), dot(g010, g010), dot(g100, g100), dot(g110, g110)));
                    g000 *= norm0.x;
                    g010 *= norm0.y;
                    g100 *= norm0.z;
                    g110 *= norm0.w;
                    vec4 norm1 = taylorInvSqrt(vec4(dot(g001, g001), dot(g011, g011), dot(g101, g101), dot(g111, g111)));
                    g001 *= norm1.x;
                    g011 *= norm1.y;
                    g101 *= norm1.z;
                    g111 *= norm1.w;

                    float n000 = dot(g000, Pf0);
                    float n100 = dot(g100, vec3(Pf1.x, Pf0.yz));
                    float n010 = dot(g010, vec3(Pf0.x, Pf1.y, Pf0.z));
                    float n110 = dot(g110, vec3(Pf1.xy, Pf0.z));
                    float n001 = dot(g001, vec3(Pf0.xy, Pf1.z));
                    float n101 = dot(g101, vec3(Pf1.x, Pf0.y, Pf1.z));
                    float n011 = dot(g011, vec3(Pf0.x, Pf1.yz));
                    float n111 = dot(g111, Pf1);

                    vec3 fade_xyz = fade(Pf0);
                    vec4 n_z = mix(vec4(n000, n100, n010, n110), vec4(n001, n101, n011, n111), fade_xyz.z);
                    vec2 n_yz = mix(n_z.xy, n_z.zw, fade_xyz.y);
                    float n_xyz = mix(n_yz.x, n_yz.y, fade_xyz.x);
                    return 2.2 * n_xyz;
                    }

                    // Include the Ashima code here!

                    varying vec2 vUv;
                    varying float noise;
                    uniform float time;

                    float turbulence( vec3 p ) {

                    float w = 100.0;
                    float t = -.5;

                    for (float f = 1.0 ; f <= 10.0 ; f++ ){
                        float power = pow( 2.0, f );
                        t += abs( pnoise( vec3( power * p ), vec3( 10.0, 10.0, 10.0 ) ) / power );
                    }

                    return t;

                    }

                    void main() {

                        vUv = uv;

                        
                        // add time to the noise parameters so it's animated
                        noise = 10.0 *  -.10 * turbulence( .5 * normal + time );
                        float b = 5.0 * pnoise( 0.05 * position + vec3( 2.0 * time ), vec3( 100.0 ) );
                        float displacement = - noise + b;

                        vec3 newPosition = position + normal * displacement;
                        gl_Position = projectionMatrix * modelViewMatrix * vec4( newPosition, 1.0 );

                    }
                `
            },
            fragmentShader() {
                return `
                    varying vec2 vUv;
                    uniform sampler2D u_texture;

                    void main() {

                        vec4 image = texture2D( u_texture, vUv );
                        gl_FragColor = vec4(0.5,0.5,0.5,1.);

                    }
                `
            },
            update() {
                this.controls.update()
            },
            render() {
                this.renderer.render( this.scene, this.camera )

                let time = performance.now() / 2000

                if ( this.shaderMaterial ) {

                    this.shaderMaterial.uniforms.time.value = performance.now() / 500;

                    this.shaderMaterial.uniforms.disp.value = Math.abs(Math.sin(time) * Math.sin(time) * Math.sin(time) * Math.sin(time))
                
                }
                
                if ( this.shaderMaterial1 ) {

                    this.shaderMaterial1.uniforms.time.value = performance.now() / 500;
                    
                    this.shaderMaterial1.uniforms.disp.value = Math.abs(Math.sin(time) * Math.sin(time) * Math.sin(time) * Math.sin(time))

                }

                if ( this.shaderMaterial2 ) {

                    this.shaderMaterial2.uniforms.time.value = performance.now() / 500;
                    
                    this.shaderMaterial2.uniforms.disp.value = Math.abs(Math.sin(time) * Math.sin(time) * Math.sin(time) * Math.sin(time))

                }


                // if (this.scene.children[13]) {
                //     //console.log(this.scene.children[13].children[0].material.uniforms['time'].value)
                //     this.scene.children[13].children[0].material.uniforms['time'].value = .00025 * ( Date.now() - this.start )
                // }

                //material.uniforms[ 'time' ].value = .00025 * ( Date.now() - start );
            },
            resizeWindowHandler() {
                this.camera.aspect = window.innerWidth / window.innerHeight
                this.camera.updateProjectionMatrix()

                this.renderer.setSize( window.innerWidth, window.innerHeight )
            }
        },
        mounted() {
            this.init()
        }
    }
</script>

<style lang="sass">
  
  .scene
    width: 100%
    height: 100%
    position: absolute

</style>
