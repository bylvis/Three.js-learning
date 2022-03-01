# three.js
# 渲染三维对象
1.创建场景对象
2.创建摄像机
3.创建渲染器
4.创建立方体
5.创建材质
6.把立方体和材质结合在一起形成一个网格
7.创建光源
# 旋转动画 周期性渲染
1.每执行一次渲染器对象WebGLRenderer的渲染方法.render()，浏览器就会渲染出一帧图像并显示在Web页面上。
2.所以按照一定的周期不停地调用渲染方法.render()就可以不停地生成新的图像覆盖原来的图像。
3.所以只要一边旋转立方体，一边执行渲染方法.render()重新渲染，就可以实现立方体的旋转效果。
<!-- 20ms刷新一次相当于fps50 -->
    setInterval(render,200)
    function render() {
        renderer.render(scene,camera)
        cube.rotateY(0.01)
    }
## requestAnimationFrame()
实际开发中，为了更好的利用浏览器渲染，可以使用函数requestAnimationFrame()代替setInterval()函数，requestAnimationFrame()和setInterval()一样都是浏览器window对象的方法。
requestAnimationFrame()参数是将要被调用函数的函数名。
大约每16.7ms调用一次requestAnimationFrame()
形成了一个递归
function render() {
        renderer.render(scene,camera);//执行渲染操作
        mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
        requestAnimationFrame(render);//请求再次执行渲染函数render
    }
render();
## 实现均匀旋转
在实际执行程序的时候，可能requestAnimationFrame(render)请求的函数并不一定能按照理想的60FPS频率执行，两次执行渲染函数的时间间隔也不一定相同，如果执行旋转命令的rotateY的时间间隔不同，旋转运动就不均匀，为了解决这个问题需要记录两次执行绘制函数的时间间隔。
rotateY()的参数是0.001*t，也意味着两次调用渲染函数执行渲染操作的间隔t毫秒时间内，立方体旋转了0.001*t弧度，很显然立方体的角速度是0.001弧度每毫秒
let T0 = new Date()
render()
function render() {
    let T1 = new Date()
    let t = T1-T0
    T0 = T1;
    requestAnimationFrame(render)
    renderer.render(scene,camera)
    // 实现了每一毫秒旋转0.001度
    cube.rotateY(0.001*t)
}
# 鼠标操作三维场景旋转缩放
可以借助three.js众多控件之一OrbitControls.js
function render(){
            renderer.render(scene,camera)
    }
render()
var controls = new THREE.OrbitControls(camera,renderer.domElement);//创建控象
controls.addEventListener('change',render)

执行构造函数THREE.OrbitControls()浏览器会同时干两件事，一是给浏览器定义了一个鼠标、键盘事件，自动检测鼠标键盘的变化，如果变化了就会自动更新相机的数据， 执行该构造函数同时会返回一个对象，可以给该对象添加一个监听事件，只要鼠标或键盘发生了变化，就会触发渲染函数。
通过requestAnimationFrame()实现渲染器渲染方法render()的周期性调用，当通过OrbitControls操作改变相机状态的时候，没必要在通过controls.addEventListener('change', render)监听鼠标事件调用渲染函数，因为requestAnimationFrame()就会不停的调用渲染函数。
// 创建辅助坐标轴
scene.add( new THREE.AxesHelper( 2000 ) );
# 插入新的几何体
threejs的几何体默认位于场景世界坐标的原点(0,0,0),所以绘制多个几何体的时候，主要它们的位置设置。

# 设置材质效果 透明度
color: 0xff00ff,
opacity: 0.7,
transparent: true
<!-- 高光 -->
new THREE.MeshPhongMaterial({
    color:0x0000ff,
    specular:0x4488ee,
    shininess:12
});//材质对象

# 光照效果设置
几种基础光源
AmbientLight	环境光
环境光颜色与网格模型的颜色进行RGB进行乘法运算

PointLight	点光源
DirectionalLight	平行光，比如太阳光
SpotLight	聚光源

## 立体效果
仅仅使用环境光的情况下，你会发现整个立方体没有任何棱角感
因为环境光只是设置整个空间的明暗效果
需要立方体渲染要想有立体效果，需要使用具有方向性的点光源、平行光源等。
通过光源构造函数的参数可以设置光源的颜色，一般设置明暗程度不同的白光RGB三个分量值是一样的。如果把THREE.AmbientLight(0x444444);的光照参数0x444444改为0xffffff,你会发现场景中的立方体渲染效果更明亮。

ps：点光源相当于太阳或者灯泡 
点光源无法照射的地方相对其他位置会比较暗
可以设置两个点光源让网格对象整体都是亮的

# 顶点概念

## 顶点位置
立方体网格模型Mesh是由立方体几何体geometry和材质material两部分构成
立方体几何体BoxGeometry本质上就是一系列的顶点构成
比如一个立方体网格模型，有6个面，每个面至少两个三角形拼成一个矩形平面，每个三角形三个顶点构成，对于球体网格模型而言，同样是通过三角形拼出来一个球面，三角形数量越多，网格模型表面越接近于球形。

## 几何体顶点颜色
通常每一个顶点都有对应的顶点颜色

<!-- // 1. 创建一个Buffer类型几何体对象 -->
var geometry = new THREE.BufferGeometry();

<!-- // 2. 类型数组创建顶点数据 -->
var vertices = new Float32Array([
    0, 0, 0, //顶点1坐标
    50, 0, 0, //顶点2坐标
    0, 100, 0, //顶点3坐标
    0, 0, 10, //顶点4坐标
    0, 0, 100, //顶点5坐标
    50, 0, 10, //顶点6坐标
]);
// 设置几何体attributes属性的位置属性
geometry.attributes.position = new THREE.BufferAttribute(vertices, 3)

<!-- // 3. 类型数组创建顶点颜色数据 -->
var colors = new Float32Array([
    1, 0, 0, //顶点1颜色
    0, 1, 0, //顶点2颜色
    0, 0, 1, //顶点3颜
    1, 1, 0, //顶点4颜色
    0, 1, 1, //顶点5颜色
    1, 0, 1, //顶点6颜色
]);
设置几何体attributes属性的位置颜色属性
geometry.attributes.color = new THREE.BufferAttribute(colors,3)

<!-- // 4.创建点材质 -->
var material2 = new THREE.PointsMaterial({
    // color: 0xff0000,
    vertexColors:THREE.VertexColors,
    size: 10.0 //点对象像素尺寸
}); //材质对象

<!-- // 5. 创建点模型 -->
var points = new THREE.Points(geometry, material2); //点模型对象
scene.add(points); //点对象添加到场景中

属性.vertexColors的默认值是THREE.NoColors，这也就是说模型的颜色渲染效果取决于材质属性.color，如果把材质属性.vertexColors的值设置为THREE.VertexColors,threejs渲染模型的时候就会使用几何体的顶点颜色数据geometry.attributes.color。

## 顶点法向量数据光照计算
物体表面与光线夹角位置不同的区域明暗程度不同
WebGL中为了计算光线与物体表面入射角，你首先要计算物体表面每个位置的法线方向，在Threejs中表示物体的网格模型Mesh的曲面是由一个一个三角形构成，所以为了表示物体表面各个位置的法线方向，可以给几何体的每个顶点定义一个方向向量。

没有定义顶点法向量数据。没有法向量数据，点光源、平行光等带有方向性的光源不会起作用，三角形平面整个渲染效果相对暗淡，而且两个三角形分界位置没有棱角感。
 var normals = new Float32Array([ //给几何体对象顶点设置法向量
    0, 0, 1, //顶点1法向量
    0, 0, 1, //顶点2法向量
    0, 0, 1, //顶点3法向
    0, 1, 0, //顶点4法向量
    0, 1, 0, //顶点5法向量
    0, 1, 0, //顶点6法向量
]);
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3)

## 顶点索引复用顶点数据
绘制一个矩形网格模型,至少需要两个三角形拼接而成，两个三角形，每个三角形有三个顶点，也就是说需要定义6个顶点位置数据。对于矩形网格模型而言，两个三角形有两个顶点位置是重合的。也就是说可以重复的位置可以定义一次，然后通过通过顶点数组的索引值获取这些顶点位置数据。
<!-- 原本是这样子 -->
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  80, 0, 0, //顶点2坐标
  80, 80, 0, //顶点3坐标

  0, 0, 0, //顶点4坐标   和顶点1位置相同
  80, 80, 0, //顶点5坐标  和顶点3位置相同
  0, 80, 0, //顶点6坐标
]);
<!-- 优化之后 -->
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  80, 0, 0, //顶点2坐标
  80, 80, 0, //顶点3坐标
  0, 80, 0, //顶点4坐标
]);
<!-- 通过index来复用顶点 -->
var indexes = new Uint16Array([
  // 0对应第1个顶点位置数据、第1个顶点法向量数据
  // 1对应第2个顶点位置数据、第2个顶点法向量数据
  // 索引值3个为一组，表示一个三角形的3个顶点
  0, 1, 2,
  0, 2, 3,
])
// 索引数据赋值给几何体的index属性
geometry.index = new THREE.BufferAttribute(indexes, 1); //1个为一组
ps:思考 可以通过一个大数组的方式 通过index从大数组里取点来设置位置 很方便
## 几何体旋转、缩放、平移变换
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 几何体xyz三个方向都放大2倍
geometry.scale(2, 2, 2);
// 几何体沿着x轴平移50
geometry.translate(50, 0, 0);
// 几何体绕着x轴旋转45度
geometry.rotateX(Math.PI / 4);
// 居中：偏移的几何体居中
geometry.center();
console.log(geometry.vertices);

## 常用材质
<!-- 点材质PointsMaterial -->
var material = new THREE.PointsMaterial({
  color: 0x0000ff, //颜色
  size: 3, //点渲染尺寸
});
<!-- 基础线材质LineBasicMaterial。 -->
var material = new THREE.LineBasicMaterial({
  color: 0x0000ff
});
<!-- 虚线材质LineDashedMaterial。 -->
var material = new THREE.LineDashedMaterial({
  color: 0x0000ff,
  dashSize: 10,//显示线段的大小。默认为3。
  gapSize: 5,//间隙的大小。默认为1
});
基础网格材质对象MeshBasicMaterial,不受带有方向光源影响，没有棱角感。
MeshLambertMaterial材质可以实现网格Mesh表面与光源的漫反射光照计算，有了光照计算，物体表面分界的位置才会产生棱角感。
高光网格材质MeshPhongMaterial除了和MeshLambertMaterial一样可以实现光源和网格表面的漫反射光照计算，还可以产生高光效果(镜面反射)。