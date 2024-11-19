

# 前言




  深入理解相机视口，摸索相机视口旋转功能，背景透明或者不透明。  本篇，实现了一个左下角旋转HUD且背景透明的相机视口。



 

# Demo




  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342334-314764659.gif)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341899-1832416911.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341975-612221992.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341956-1991491437.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342032-403600919.png)



 

# HUD相机的坐标




  抬头HUD就是通过投影矩阵来实现，具体可参看《OSG开发笔记（二十）：OSG使用HUD显示文字》




* Hud要单独创建一个新相机
* 注意关闭光照，不受光照影响，所以内容以同一亮度显示
* 关闭深度测试
* 渲染顺序设置为POST，否则可能会被场景中的其他图形所覆盖。
* 设置参考贴为绝对型：setReferenceFrame(osg::Transform:ABSOLUTE\_RF)
* 使其不受父节点变换的影响：setMatrix(osg::Matrix::identity())
* 投影矩阵通常会设置为屏幕尺寸大小



 

# 相机（Camera）




  相机（osg::Camera）和视口（Viewport）是两个核心概念，对于理解OSG中的三维场景渲染至关重要。  相机在OSG中用于模拟真实世界中的摄影机，它负责捕捉和渲染三维场景。相机类（osg::Camera）继承自osg::Transform和osg::CullSetting类，用来管理OSG中的模型——视图矩阵。相机的管理主要是通过各种变换实现的，这些变换包括：




* 视点变换：设置视点的方向和位置。默认情况下，视点定位为坐标原点，指向Y正方向。可以通过调整视点的位置和参考点的位置来改变相机的观察方向和角度。
* 投影变换：由于显示器只能用二维图像显示三维物体，因此要靠投影来降低维数。投影变换的目的是定义一个视景体，使视景体外多余的部分被裁减掉，最终进入图像的只是视景体内的有关部分。OSG支持两种投影方式：透视投影（Perspective Projection）和正视投影（Orthographic Projection）。透视投影能够模拟人眼的视觉效果，使远处的物体看起来更小，而正视投影则保持物体的大小不变，不受距离影响。
* 视口变换：将视景体内投影的物体显示在二维的视口平面上。即将经过几何变换、投影变换和裁剪变换后的物体显示于屏幕窗口内指定的区域内，这个区域通常为矩形，称为视口。



 

# 视口（ViewPort）




  具体来说，视口变换涉及以下几个参数：




* 屏幕左下角的坐标：定义了视口在屏幕上的左下角位置。
* 屏幕宽度和高度：定义了视口的宽度和高度，即相机捕捉的场景在屏幕上显示的区域大小。  在OSG中，可以通过调用相机的setViewport方法来设置视口。例如：





```
pCamera->setViewport(new osg::Viewport(0, 0, width, height));

```



  这行代码创建了一个新的视口，并将其设置为相机的当前视口。其中，0和0是屏幕左下角的坐标，width和height是视口的宽度和高度。




# 相机与视口的关系




  相机和视口在OSG中紧密相连，共同决定了三维场景的渲染效果。相机负责捕捉和渲染场景，而视口则定义了相机捕捉的场景在屏幕上的显示位置和大小。通过调整相机的各种变换和设置视口的大小和位置，可以实现丰富的三维视觉效果和交互体验。




# 设置相机观察函数





```
void setViewMatrixAsLookAt(const osg::Vec3d& eye, const osg::Vec3d& center, const osg::Vec3d& up);

```



* eye：表示相机的位置。这是一个三维向量，指定了相机在世界坐标系中的位置。
* center：表示相机观察的中心点。这也是一个三维向量，指定了相机应该对准的物体或场景的中心位置。
* up：表示哪个方向是正方向。这同样是一个三维向量，通常用于指定相机的上方方向（例如，通常设置为 (0,0,1\) 表示Y轴正方向为上方）。  设置相机位置和方向：通过指定 eye、center 和 up 三个参数，你可以精确地控制相机的位置和姿态。eye 和 center 之间的向量表示相机的观察方向，而 up 向量则用于确定相机的上方方向。  关闭漫游器：在使用 setViewMatrixAsLookAt 函数之前，通常需要关闭相机的漫游器（Camera Manipulator）。这是因为漫游器会自动更新相机的观察矩阵，从而覆盖你通过 setViewMatrixAsLookAt 设置的参数。可以通过调用 viewer\-\>setCameraManipulator(NULL) 来关闭漫游器。  坐标系：OSG 使用右手坐标系，其中 X 轴向右，Y 轴向上，Z 轴向前。因此，在设置 eye、center 和 up 参数时，需要确保它们符合右手坐标系的规则。  视图矩阵：setViewMatrixAsLookAt 函数实际上是通过设置相机的视图矩阵来实现相机位置和姿态的调整。视图矩阵是一个 4x4 的矩阵，用于将相机坐标系中的点转换到世界坐标系中。setViewMatrixAsLookAt 是一个强大的函数，它允许你以直观的方式设置相机的位置和姿态。通过合理地使用这个函数，你可以创建出各种复杂而逼真的三维场景和视觉效果。



 

# Demo关键源码




## 创建Hud相机





```
    // 步骤一：创建HUD摄像机
//        osg::ref_ptr pCamera = new osg::Camera;
    osg::ref_ptr pCamera = new HudRotateCamera;
    pCamera->setMasterCamera(_pViewer->getCamera());
    // 步骤二：设置投影矩阵
//        pCamera->setProjectionMatrix(osg::Matrix::ortho2D(0, 1280, 0, 800));
    // 步骤三：设置视图矩阵,同时确保不被场景中其他图形位置变换影响, 使用绝对帧引用
    pCamera->setReferenceFrame(osg::Transform::ABSOLUTE_RF);
    pCamera->setViewMatrix(osg::Matrix::identity());
    // 步骤四：清除深度缓存
    pCamera->setClearMask(GL_DEPTH_BUFFER_BIT);
    // 步骤五：设置POST渲染顺序(最后渲染)
//        pCamera->setRenderOrder(osg::Camera::PRE_RENDER);       // 渲染不显示
//        pCamera->setRenderOrder(osg::Camera::NESTED_RENDER);
    pCamera->setRenderOrder(osg::Camera::POST_RENDER);
    // 步骤六：设置为不接收事件,始终得不到焦点
    pCamera->setAllowEventFocus(false);

//        osg::ref_ptr pGeode = new osg::Geode();
//        pGeode = new osg::Geode();
    osg::ref_ptr pStateSet = pGeode->getOrCreateStateSet();
    // 步骤七：关闭光照
    pStateSet->setMode(GL_LIGHTING, osg::StateAttribute::OFF);
    // 步骤九：关闭深度测试
    pStateSet->setMode(GL_DEPTH_TEST, osg::StateAttribute::OFF);

//        pGeode->addDrawable(pGeometry.get());

    pCamera->addChild(pGeode.get());

    pGroup->addChild(pCamera.get());

```



## HudRotateCamera.h





```
#ifndef HUDROTATECAMERA_H
#define HUDROTATECAMERA_H

#include "osg/Camera"
#include "osg/CopyOp"

class HudRotateCamera : public osg::Camera
{
public:
    HudRotateCamera();

    HudRotateCamera(const HudRotateCamera& copy, const osg::CopyOp &copyOp = osg::CopyOp::SHALLOW_COPY);

    META_Node(osg, HudRotateCamera);

public:
    void setMasterCamera(Camera* camera);

public:
    virtual void traverse(osg::NodeVisitor& nodeVisitor);

protected:
    virtual ~HudRotateCamera();

protected:
    osg::observer_ptr _pMasterCamera;    // 新增了相机，主要是用来获取举证的
};

#endif // HUDROTATECAMERA_H

```



## HudRotateCamera.cpp





```
#include "HudRotateCamera.h"

HudRotateCamera::HudRotateCamera(): Camera()
{

}

HudRotateCamera::HudRotateCamera(const HudRotateCamera & copy,  const osg::CopyOp & copyOp)
    : Camera(copy, copyOp),
      _pMasterCamera(copy._pMasterCamera)
{

}

HudRotateCamera::~HudRotateCamera()
{

}

void HudRotateCamera::setMasterCamera(osg::Camera *camera)
{
    _pMasterCamera = camera;
}

void HudRotateCamera::traverse(osg::NodeVisitor &nodeVisitor)
{
    double fovy, aspectRatio, vNear, vFar;
    _pMasterCamera->getProjectionMatrixAsPerspective(fovy, aspectRatio, vNear, vFar);

    // 设置投影矩阵，使缩放不起效果, 改为正投影，正投影不会随相机的拉近拉远而放大、缩小，这样就没有缩放效果，
    // 放大缩小是根据左右，上下距离，越大就物体越小，越小就物体越大
    this->setProjectionMatrixAsOrtho(-50 * aspectRatio,
                                      50 * aspectRatio,
                                     -50,
                                      50,
                                      vNear,
                                      vFar);
    // 让坐标轴模型位于窗体左下角
    osg::Vec3 vec3(-40, -40, 0);
    if (_pMasterCamera.valid())
    {
        // 改变视图矩阵, 让移动位置固定
        osg::Matrix matrix = _pMasterCamera->getViewMatrix();

        // 让移动固定, 即始终位于窗体右下角，否则鼠标左键按住模型可以拖动或按空格键时模型会动
        matrix.setTrans(vec3);
        this->setViewMatrix(matrix);
    }
    osg::Camera::traverse(nodeVisitor);
}

```


 

# 工程模板v1\.35\.0




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341918-350827123.png)



 

# 入坑




## 入坑一：没有按照预期的方式全屏显示在正中间




### 问题




  想一直显示在中间，且能旋转，移动中心，但是实际效果如下，方格100x100，间距1\.0，测试：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341949-875730634.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342090-172877880.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341979-1837192764.png)




### 尝试




  将面方格缩小为10x10，线放小，测试：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342191-1900686487.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342020-1387680465.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342242-1248682696.png)




  缩小正交投影：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341858-2142326336.png)  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342124-166317494.png)




  可能跟相机查看位置有关，新增相机位置和方向等信息：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341834-2087719459.png)




  没什么影响：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342028-634927114.png)




  这里可能理解有问题，我们需要区域投影到视口，那么一个是投影的区域三维区域的大小，一个是投影到桌面2D他的大小，这里其实类似于HUD，通过HUD的方式，添加了几行代码设置投影矩阵：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341978-247674071.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342165-1767074295.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342167-558956271.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341973-1748942497.png)




  经过测试，可以跳过相机调整视口、中心改变后也会移动，所以他一种在其位置区域。  然后回到前面，发现也可以，再次摸索，发现如下特点：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342237-1651773861.png)




  所以此时，纵横都是10，所以窗口大小要是符合1：1（10：10\=1：1）的比例，改为400，400测试，还是一样：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342157-1698549363.png)




  但是调整为800x800就好了：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341885-1312942337.png)




  放最大也不会截取少了：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341828-783485972.png)




  所以这个有点搞不明白了，总之是解决了，且投影矩阵和正交矩阵都可以解决，测试投影矩阵和正交举证都收视口大小影响，但是影响具体不知，就好像800x800是最小一样（其他的没测了，只测了400x400、600x600不行，看比例800x800是最小正好满窗口了）。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342110-270888958.png)




  又怀疑投过去的区域小了，将区域放大，其位置反倒缩小，所以跟理解不一样：




* 一种是理解直接投射过去，投射过去区域变大所以变大（不是的）；
* 一种是投射过去区域不变，那么区域变大视图区域可见空间范围变大（实际是这样，但是视口对屏幕的大小未变）；  综合以上，又测试了加大视口，也正常，所以怀疑有可能是qt和osg结合的时候这个地方设置了一个最小值，而可能吧，欢迎探讨，这里深究暂时也没结果，且解决了，所以不继续深究了。




### 解决




  修改相机视口大小为最小800x800。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341922-741164912.png)




### 后续补充




  后续查看做的这个qtosg兼容类，做的时候，自己设置的800x800，就是这个原因了：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342067-1418858693.png)




## 入坑二：相机视口区域不透明




### :[wgetCloud机场](https://longdu.org)问题




  当作最前面的文本hud，是可以透明，但是这里进行调整之后，无法透明。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341865-2047122340.png)




### 尝试




  修改了语句，可以透明了部分，但是没了。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341766-2125662432.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341863-822792322.png)




### 解决




  相机是一个投影矩阵，没有透明，但是文字hud为什么透明呢？。




## 入坑三：内置几何体关闭光照后纯白色




### 问题




  关闭光照后，几何体白色  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342059-1201052626.png)




### 原理




  光照关闭要设置颜色，不想设置颜色，就单独给体开放关照。




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342023-1663310983.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342058-1760141233.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342275-1340893966.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342153-1486236898.png)




## 入坑四：文本看不到




### 问题




  文本看不到，旋转后发现是太大了。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342088-385126377.png)




  位置较大。




### 尝试




  测试下是把整个坐标区域的展示范围扩大，那么实际看起来就是缩小。




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341770-298810320.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342117-602662411.png)




## 入坑五：hud旋转中心不对




### 问题




  Hud旋转中心不对  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342282-1028401302.png)




  这时旋转中心还不对，可能需要调整旋转中心  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342260-1925089756.png)




### 原理




  开始去修改矩阵，发现都不对，其实中间点一直是0，0，0，其就是中心，那么我们设置文本的显示点不是从0，0开始即可。  下面将四边形的角点改为0，0，0来标识，然后修改文本的中心点：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342279-1305271726.png)




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342257-1117301821.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342262-76906304.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341981-1764390012.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341833-1169624834.png)




## 入坑六：hud旋转反向了




### 问题




  旋转是反的，然后光照也反了，变换矩阵有问题  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342286-2101650215.png)




### 原理




  数据几何变换算来算去很费劲，直接测试的结论：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342267-1145448583.png)




  代码写反了，让上下反向了，应该是\-50\~50




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341837-477798180.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143342025-1034779327.png)




  还剩下光照问题，这个不好咋弄了，反正是关闭光照，或者是自己手动添加光源，用系统的可能有点问题。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341971-1442836910.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241118143341968-1418303569.png)




  这个暂时没解决，实际使用就是用一个，可以从长度单独给这个相机设置一个光源，这里因为需求本身不需要投影，不做测试了。



