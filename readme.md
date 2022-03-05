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

## 材质共有属性、私有属性
尺寸属性.size
高光颜色属性.specular
材质透明度.opacity
是否开启透明transparent
所有子类的材质都会从父类材质Material继承透明度opacity、面side等属性，这些来自父类的属性都是子类共有的属性

# 模型对象
点、线、网格模型介绍
点模型Points、线模型Line、网格网格模型Mesh都是由几何体Geometry和材质Material构成
开发的时候可以通过设置wireframe属性来查看网格模型的三角形分布特点。
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff, //三角面颜色
  wireframe:true,//网格模型以线条的模式渲染
});
也可以通过material.wireframe来设置
<!-- 让模型沿着自定义方向(向量)移动  -->
var axis = new THREE.Vector3(1, 1, 1);
axis.normalize(); //向量归一化
//沿着axis轴表示方向平移100
mesh.translateOnAxis(axis, 100)
<!-- 让模型围绕向量旋转 -->
var axis1 = new THREE.Vector3(1, 1, 0);//向量axis
mesh.rotateOnAxis(axis1, Math.PI / 8);//绕axis轴旋转π/8

## 克隆
var box = new THREE.BoxGeometry(10, 10, 10);//创建一个立方体几何对象
var material = new THREE.MeshLambertMaterial({ color: 0x0000ff });//材质对
var mesh = new THREE.Mesh(box, material);//网格模型对象
var mesh2 = mesh.clone();//克隆网格模型
mesh.translateX(20);//网格模型mesh平移
// box.scale(1.5,1.5,1.5);
scene.add(mesh, mesh2);//网格模型添加到场景中

放大box 会发现两个网格模型都放大了 说明clone是浅拷贝

## 光照原理和常见光源类型
<!-- 环境光AmbientLight -->
环境光是没有特定方向的光源，主要是均匀整体改变Threejs物体表面的明暗效果，这一点和具有方向的光源不同，比如点光源可以让物体表面不同区域明暗程度不同。
<!-- 点光源PointLight -->
点光源类似于灯泡、太阳。
<!-- 平行光DirectionalLight -->
类似于激光笔手电筒
平行光如果不设置.position和.target属性，光线默认从上往下照射，也就是可以认为(0,1,0)和(0,0,0)两个坐标确定的光线方向。
<!-- 聚光源SpotLight -->
var spotLight = new THREE.SpotLight(0xffffff);
// 设置聚光光源位置
spotLight.position.set(200, 200, 200);
// 聚光灯光源指向网格模型mesh2
spotLight.target = mesh2;
// 设置聚光光源发散角度
spotLight.angle = Math.PI / 6
scene.add(spotLight);//光对象添加到scene场景中

光源辅助工具
<!-- 平行光 -->
var light = new THREE.DirectionalLight( 0xFFFFFF );
var helper = new THREE.DirectionalLightHelper( light, 5 );
scene.add( helper );
<!-- 聚光灯 -->
var spotLight = new THREE.SpotLight( 0xffffff );
spotLight.position.set( 10, 10, 10 );
scene.add( spotLight );
var spotLightHelper = new THREE.SpotLightHelper( spotLight );
scene.add( spotLightHelper );
<!-- 点光源 -->
var pointLight = new THREE.PointLight( 0xff0000, 1, 100 );
pointLight.position.set( 10, 10, 10 );
scene.add( pointLight );
var sphereSize = 1;
var pointLightHelper = new THREE.PointLightHelper( pointLight, sphereSize );
scene.add( pointLightHelper );
## 光照计算算法
Threejs在渲染的时候网格模型材质的颜色值mesh.material.color和光源的颜色值light.color会进行相乘，简单说就是RGB三个分量分别相乘。
平行光漫反射简单数学模型：漫反射光的颜色 = 网格模型材质颜色值 x 光线颜色 x 光线入射角余弦值
## 阴影实时计算
Three.js物体投影模拟计算主要设置三部分，一个是设置产生投影的模型对象，一个是设置接收投影效果的模型，最后一个是光源对象本身的设置，光源如何产生投影。
<!-- 尤其注意要给渲染器开启阴影效果 -->
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.BasicShadowMap;
<!-- 模型.castShadow -->
属性值是布尔值，默认false，用来设置一个模型对象是否在光照下产生投影效果。
// 设置产生投影的网格模型
mesh.castShadow = true;
<!-- 模型.receiveShadow -->
属性值是布尔值，默认false，用来设置一个模型对象是否在光照下接受其它模型的投影效果。
planeMesh.receiveShadow = true;
<!-- 光源.castShadow属性 -->
如果属性设置为 true， 光源将投射动态阴影. 警告: 这需要很多计算资源，需要调整以使阴影看起来正确.
directionalLight.castShadow = true;
<!-- 光源.shadow属性 -->
平行光DirectionalLight的.shadow属性值是平行光阴影对象DirectionalLightShadow，聚光源SpotLight的.shadow属性值是聚光源阴影对象SpotLightShadow

# 层级模型、树结构
## 层级模型
Group概念
一个Group可以添加多个mesh，相当于一个头上添加了鼻子和耳朵，头动了，鼻子和耳朵会跟着动。
// 5.创建一个网格模型 
var geometry = new THREE.BoxGeometry(20, 20, 20)
var material = new THREE.MeshLambertMaterial({ color: 0x0000ff });
var group = new THREE.Group();
var mesh1 = new THREE.Mesh(geometry, material);
var mesh2 = new THREE.Mesh(geometry, material);
mesh2.translateX(25);
//把mesh1型插入到组group中，mesh1作为group的子对象
group.add(mesh1);
//把mesh2型插入到组group中，mesh2作为group的子对象
group.add(mesh2);
//把group插入到场景中作为场景子对象
scene.add(group)
group.translateY(100);
group.rotateY(Math.PI/6)
console.log('查看group的子对象',group.children);
console.log('查看Scene的子对象',scene.children);
group.remove(mesh1)

## 层级模型节点命名、查找、遍历
在层级模型中可以给一些模型对象通过.name属性命名进行标记。
group.add(Mesh)
// 网格模型命名
Mesh.name = "眼睛"
// mesh父对象对象命名
group.name = "头"
<!-- 递归遍历方法.traverse() -->
Threejs层级模型就是一个树结构，可以通过递归遍历的算法去遍历Threejs一个模型对象的所有后代，可以通过下面代码递归遍历上面创建一个机器人模型或者一个外部加载的三维模型。
scene.traverse(function(obj) {
  if (obj.type === "Group") {
    console.log(obj.name);
  }
  if (obj.type === "Mesh") {
    console.log('  ' + obj.name);
    obj.material.color.set(0xffff00);
  }
  if (obj.name === "左眼" | obj.name === "右眼") {
    obj.material.color.set(0x000000)
  }
  // 打印id属性
  console.log(obj.id);
  // 打印该对象的父对象
  console.log(obj.parent);
  // 打印该对象的子对象
  console.log(obj.children);
})
<!-- 查找某个具体的模型 -->
看到Threejs的.getObjectById()、.getObjectByName()等方法，如果已有前端基础，很容易联想到DOM的一些方法。
// 遍历查找scene中复合条件的子对象，并返回id对应的对象
var idNode = scene.getObjectById ( 4 );
console.log(idNode);
// 遍历查找scene中复合条件的子对象，并返回id对应的对象
var nameNode = scene.getObjectByName ( "左腿" );
nameNode.material.color.set(0xff0000);
## 本地位置坐标、世界位置坐标
// 声明一个三维向量用来保存世界坐标
cube.position.set(20,20,20)
var worldPosition = new THREE.Vector3();
cube.getWorldPosition(worldPosition);
console.log(worldPosition);
本地坐标系或者说模型坐标系，就是模型对象相对模型的父对象而言
.position表示的坐标值就是以本地坐标系为参考，表示子对象相对本地坐标系原点(0,0,0)的偏移量。
一个模型相对世界坐标系的坐标值就是该模型对象所有父对象以及模型本身位置属性.position的叠加。

# 几何体对象、曲线、三维建模
<!-- 圆弧线ArcCurve -->
var geometry = new THREE.BufferGeometry();// 开始创建立方体
var arc = new THREE.ArcCurve(0, 0, 100, 0, 2 * Math.PI);
var points = arc.getPoints(50);//分段数50，返回51个顶点
geometry.setFromPoints(points);
<!-- 设置线的材质 -->
var material = new THREE.LineBasicMaterial({
            color: 0x000000
        });
var cube = new THREE.Line(geometry, material)// 把立方体和材质结合在一起
scene.add(cube)// 把立方体网格添加到场景中
// 如果几何体是BufferGeometry，setFromPoints方法改变的是.attributes.position属性
console.log(geometry.attributes.position);
<!-- 由于Geometry弃用 暂时只找到这种设置方法 -->
 geometry1.attributes.position = new THREE.BufferAttribute(new Float32Array([
            50,0,0,
            0,70,0
        ]),3)
## 样条曲线、贝塞尔曲线
样条曲线 就是给出几个三维空间的几个点 然后通过  new THREE.CatmullRomCurve3 算法 算出光滑的曲线
再从里面提取出几个点作为空间的顶点 再由three拼接
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
// 三维样条曲线  Catmull-Rom算法
var curve = new THREE.CatmullRomCurve3([
  new THREE.Vector3(-50, 20, 90),
  new THREE.Vector3(-10, 40, 40),
  new THREE.Vector3(0, 0, 0),
  new THREE.Vector3(60, -60, 0),
  new THREE.Vector3(70, 0, 80)
]);
//getPoints是基类Curve的方法，返回一个vector3对象作为元素组成的数组
var points = curve.getPoints(100); //分段数100，返回101个顶点
// setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
geometry.setFromPoints(points);
//材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
<!-- 组合曲线 -->
var geometry = new THREE.BufferGeometry()
var R = 80
var arc = new THREE.ArcCurve(0, 0, R, 0, Math.PI, true)
var line1 = new THREE.LineCurve(new THREE.Vector2(R, 200, 0), new THREE.Vector2(R, 0, 0))
var line2 = new THREE.LineCurve(new THREE.Vector2(-R, 0, 0), new THREE.Vector2(-R, 200, 0));
// 创建组合曲线对象CurvePath
var CurvePath = new THREE.CurvePath();
CurvePath.curves.push(line1, arc, line2);
var points = CurvePath.getPoints(200);
geometry.setFromPoints(points);
var material = new THREE.LineBasicMaterial({
    color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中

## 管道
// path:路径   40：沿着轨迹细分数  2：管道半径   25：管道截面圆细分数、
//创建管道成型的路径(3D样条曲线)
var path = new THREE.CatmullRomCurve3([
    new THREE.Vector3(-10, -50, -50),
    new THREE.Vector3(10, 0, 0),
    new THREE.Vector3(8, 50, 50),
    new THREE.Vector3(-5, 0, 100)
]);
var geometry = new THREE.TubeGeometry(path, 40,2,25);

## 旋转成型
LatheGeometry(points, segments, phiStart, phiLength)
参数	值
points	Vector2表示的坐标数据组成的数组
segments	圆周方向细分数，默认12
phiStart	开始角度,默认0
phiLength	旋转角度，默认2π
