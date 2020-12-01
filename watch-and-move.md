## Watch
html
```html
<div id="blocker">
    <div id="instructions">
        <span style="font-size:36px">点击开始</span>
        <br /><br />
        移动: WASD<br/>
        跳跃: SPACE<br/>
        观察: MOUSE
    </div>
</div>
```
css
```css
#blocker {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
}

#instructions {
    width: 100%;
    height: 100%;

    display: -webkit-box;
    display: -moz-box;
    display: box;

    -webkit-box-orient: horizontal;
    -moz-box-orient: horizontal;
    box-orient: horizontal;

    -webkit-box-pack: center;
    -moz-box-pack: center;
    box-pack: center;

    -webkit-box-align: center;
    -moz-box-align: center;
    box-align: center;

    color: #ffffff;
    text-align: center;
    font-family: Arial;
    font-size: 14px;
    line-height: 24px;

    cursor: pointer;
}
```
js
```js
<script type="module">
    import * as THREE from '../three.js/build/three.module.js';
    import { PointerLockControls } from '../three.js/examples/jsm/controls/PointerLockControls.js';
    import { common as my } from './js/common.js';
    let cube, controls;

    // 初始化
    init();

    controls = new PointerLockControls( my.camera, document.body );
    const blocker = document.getElementById( 'blocker' );
    const instructions = document.getElementById( 'instructions' );

    instructions.addEventListener( 'click', function () {
        controls.lock();
    }, false );

    controls.addEventListener( 'lock', function () {
        instructions.style.display = 'none';
        blocker.style.display = 'none';
    } );

    controls.addEventListener( 'unlock', function () {
        blocker.style.display = 'block';
        instructions.style.display = '';
    } );

    my.scene.add( controls.getObject() );

    // 动画
    animate();

    function init() {
        // my.scene.background = new THREE.Color(0xbfe3dd);
        addLight();
        addPlane();
        addCube();
        my.enableWindowResize();
        // my.enableControls();

    }



    function addLight() {
        // light
        const hemisphereLight = new THREE.HemisphereLight(0xFFFFFF, 0x000000, 1);
        my.scene.add(hemisphereLight);
    }

    function addPlane() {
        const geometry = new THREE.PlaneBufferGeometry(2, 2, 100, 100);
        geometry.rotateX(- Math.PI / 2);
        const material = new THREE.MeshPhongMaterial({
            color: 0x909090,
            side: THREE.DoubleSide,
        });
        const mesh = new THREE.Mesh(geometry, material);
        mesh.position.y = -0.2;
        mesh.receiveShadow = true;
        my.scene.add(mesh);
        return mesh;
    }

    function addCube() {
        // geometry, material -> mesh
        const geometry = new THREE.BoxGeometry(0.2, 0.2, 0.2); // width、height、depth
        const material = new THREE.MeshStandardMaterial({
            color: "#ff0000"
        });
        cube = new THREE.Mesh(geometry, material);
        my.scene.add(cube);
    }

    function animate() {
        requestAnimationFrame(animate);
        cube.rotation.x += 0.01; // 面朝 x 正方向顺时针
        cube.rotation.y += 0.02; // 面朝 y 正方向顺时针
        // scene, camera ->  renderer
        my.renderer.render(my.scene, my.camera);
    }

</script>
```
## Jump
初识变量
```js
let canJump = true;                     // 是否可以跳跃
const velocity = new THREE.Vector3();   // 速度
```
按下空格键
```js
const onKeyDown = function ( event ) {
    switch ( event.keyCode ) {
	    case 32: // space
            if ( canJump === true ) {
                velocity.y += 2;
            }
            canJump = false;
            break;
    }
};
document.addEventListener( 'keydown', onKeyDown, false );
```
动画过程
```js
let prevTime = performance.now();
function animate() {
    const time = performance.now();
    if ( controls.isLocked === true ) {
        const delta = ( time - prevTime ) / 1000;
        velocity.y -= 9.8 * delta;
        if ( controls.getObject().position.y < 0 ) {
            velocity.y = 0;
            controls.getObject().position.y = 0;
            canJump = true;
        } else {
            controls.getObject().position.y += ( velocity.y * delta );
        }
    }
    prevTime = time;
}
```
## Move
初始变量
```js
let moveForward = false;
let moveBackward = false;
let moveLeft = false;
let moveRight = false;
const direction = new THREE.Vector3();
```
按下方向键
```js
const onKeyDown = function (event) {
    switch (event.keyCode) {

        case 38: // up
        case 87: // w
            moveForward = true;
            break;

        case 37: // left
        case 65: // a
            moveLeft = true;
            break;

        case 40: // down
        case 83: // s
            moveBackward = true;
            break;

        case 39: // right
        case 68: // d
            moveRight = true;
            break;
    }
};

const onKeyUp = function (event) {
    switch (event.keyCode) {

        case 38: // up
        case 87: // w
            moveForward = false;
            break;

        case 37: // left
        case 65: // a
            moveLeft = false;
            break;

        case 40: // down
        case 83: // s
            moveBackward = false;
            break;

        case 39: // right
        case 68: // d
            moveRight = false;
            break;
    }
};
```
动画过程
```js
// 判断移动方向
// 1: right     -1: left
direction.x = Number(moveRight) - Number(moveLeft);
// 1: forward   -1: back
direction.z = Number(moveForward) - Number(moveBackward);
direction.normalize();
// 是否左右移动
if (moveLeft || moveRight)  {
    // 加速
    velocity.x -= direction.x * delta;
} else {
    // 降速
    if (Math.abs(velocity.x) > 0.0001) {
        velocity.x -= velocity.x * 10.0 * delta;
    } else {
        velocity.x = 0;
    }
}
// 是否前后移动
if (moveForward || moveBackward) {
    // 加速
    velocity.z -= direction.z * delta;
} else {
    // 降速
    if (Math.abs(velocity.z) > 0.0001) {
        velocity.z -= velocity.z * 10.0 * delta;
    } else {
        velocity.z = 0;
    }
}
controls.moveRight(- velocity.x * delta);
controls.moveForward(- velocity.z * delta);
```
