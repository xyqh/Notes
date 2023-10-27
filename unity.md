# Unity

## 生命周期

### 生命周期函数

> 声明周期函数可以分为两种状态：
>
> 编辑器下的状态：只有Reset函数一个
>
> 运行状态：Awake->OnEnable->Start->FixedUpdate->Update->LateUpdate->OnDisable->OnDestroy
>
> ![img_unity_1](.\images\img_unity_1.png)

### 初始化阶段

> #### Awake（唤醒）
>
> > 当一个脚本实例被载入且Active为true时Awake会被调用，一个非激活状态的预制件Instantiate的时候不会调用Awake，在首次调用SetActive(true)时会且只会调用一次，也就是说同一个GameObject的Awake函数只会被调用一次。由于其调用顺序早于Start，所以Awake常用于在游戏开始之前初始化变量或游戏状态，可以判断当满足条件后执行此脚本（**只调用一次**）。
> >
> > 需要注意的是，如果是在Update中调用的SetActive(true)，Awake函数会立即被调用，而Start函数是在下一帧才调用的。
>
> #### OnEnable（当可用）
>
> > 当对象变为可用或激活状态时此函数被调用。（可多次调用）
>
> #### Reset（重置）（Editor）
>
> > Reset是在用户点击检视面板的Reset或者首次添加该组件时被调用。此函数只在编辑模式下被调用。Reset最常用于在检视面板中给定一个最常用的默认值。
>
> #### Start（开始）
>
> > 物体载入且脚本对象启用时被调用1次，常用于数据或逻辑对象初始化（**只调用一次**）。
> >
> > Start仅在Update函数第一次被调用前调用，且只会在脚本实例启用时被调用一次。
> >
> > Awake总是在Start之前执行，可以按需调整延迟初始化代码。

### 物理阶段

> #### FixedUpdate（固定更新）
>
> > FixedUpdate基于一个可靠的定时器被调用，独立于帧率之外，按照固定时间间隔调用。如果固定的时间步长小于实际的帧更新频率，那么每帧的物理循环可能会发生不止一次。处理物体的物理属性（Rigidbody、Force、Collider）或者输入事件时，需要用FixedUpdate代替Update，以使物体的物理表现更平滑。实际上，FixedUpdate并不是真的按照现实时间间隔执行的，而是按照Timer时间间隔执行的，但Timer并不是真正意义上的现实时间，它的作用是在运行环境下创造一个与现实时间高度相近的变量来实现物理帧的逻辑稳定。因为FixedUpdate的这个特质，强烈建议在此环节只做物理相关的处理，不要把其他类型（如网络帧同步）的处理也放入此步骤。默认频率大概为50Hz（即0.02s间隔），该频率可手动修改。
> >
> > **在这期间的操作**
> >
> > 固定更新结束后，系统内部会进行一系列的操作，最重要的莫过于Unity的内部物理更新，这个是真正的物理更新操作执行。具体执行步骤大概如下：
> >
> > ![img_unity_2](.\images\img_unity_2.png)
>
> #### OnTriggerXXX（触发）
>
> > 触发器被触发时调用。
>
> #### OnCollisionXXX（碰撞）
>
> > 产生碰撞事件时调用
>
> #### yield WaitForFixedUpdate（协程：物理帧结束）
>
> > 当物理帧执行完毕后会跳转到此协程，协程的调用跟方法是不同的，可以理解为在一段代码中设置一个卡点，当程序执行到这个卡点所匹配的时机时卡点后面的代码才会继续执行。
>
> #### 物理阶段总结
>
> > 该阶段的发生与渲染无关，其特性决定了其处理物理事件的功能，使物理展示效果更为平滑，另外固定更新的频率并非真是固定的，实际的执行会根据CPU轮转时间片产生偏移，但这个偏移基本可以忽略不记。

### 输入事件阶段（OnMouseXXX）

> 鼠标、键盘、触屏、手柄等各类输入事件会在这个阶段触发，这个时间点物理更新已经执行（如果需要物理更新的话），而逻辑更新和渲染并未执行，要了解这个触发的时机，才能更好地掌握代码逻辑。

### 游戏逻辑阶段

> #### Update（更新）
>
> > Update是真正的每帧调用的，由于系统性能以及游戏体量的区别，每一帧的刷新频率也是不同的，所以不要过分期待在Update方法中按时完成任务。
> >
> > Update和FixedUpdate实际上是使用同一个线程的，Update在loop中的处理方式是本次更新完毕再根据上一帧到现在的偏移时间判断是否进行下一次更新，Update的本质是回调函数。只要是回调函数就存在上下文传递的损耗，可以考虑自己实现一套Update机制，使用虚函数来代替Update。
>
> #### 判断多个协程点
>
> > 在Update执行过后，将会到达以下几个协程的出发点，如果在此之前设置了相关的协程，这时就会生效
> >
> > - yield null
> > - yield WaitForSeconds
> > - yield WWW：**这个比较重要**，当网络任务执行完成后，会在当前帧的这个时间点执行WWW之后的操作。一般用于异步加载资源。
> > - yield StartCoroutine
>
> #### 内部动画更新
>
> > 在几个协程过后，Unity将进行第二次大规模的系统内部操作，主要的工作内容就是动画的更新，具体内容如下：
> >
> > ![img_unity_3](.\images\img_unity_3.png)
>
> #### LateUpdate（延后更新）
>
> > 每帧Update方法调用之后会调用本方法。因为游戏开发过程中经常会有一个二次计算的情况，比如主角移动，相机跟着移动。如果相机也在主角移动时跟随，当有物体跟玩家之间产生了相位，就会出现抽出抖动等情况（因为并没有在这一帧逻辑完全结束后调用跟随）。所以LateUpdate的出现能够使程序更加顺畅。

### 渲染阶段

> #### OnWillRenderObject
>
> > 当即将渲染物体时调用。
>
> #### OnPreCull
>
> > 这个函数仅用于宿主为摄像机的脚本。当此摄像机剔除了某个渲染场景时触发此消息。
>
> #### OnBecameVisible（即将可见）
>
> > 当物体即将可见时调用。
>
> #### OnBecameInVisible（即将不可见）
>
> > 当物体即将不可见时调用。
>
> #### OnPreRender（即将渲染）
>
> > 这个函数仅用于宿主为摄像机的脚本。当此摄像机开始渲染某个场景时触发此消息。
> >
> > 在所有渲染开始之前调用，这个方法其实是很考究的，大部分网友对于这个方法都是抄了几种用法：加入脚本、设定标题、设定按钮客户端事件、设定控件的状态、加入脚本快。
>
> #### OnRenderObject
>
> > 这个函数仅用于宿主为摄像机的脚本。当使用Graphics.DrawMeshNow或者其他函数绘制自己建立的物体渲染完毕时触发。
>
> #### OnPostRender
>
> > 这个函数仅用于宿主为摄像机的脚本。当此摄像机范围内所有渲染都完成时触发此消息。
>
> #### OnRenderImage
>
> > 当所有渲染完成Image的postProcessing effects（只有pro版本支持）后触发。
>
> #### OnDrawGizmos（Gizmos渲染）
>
> > Gizmos一般是开发者使用的，指的是开发时场景编辑器中所展示的那些相机、线框之类的物体。所以此方法里的内容一般不需要发布到生产环境中。
>
> #### OnGUI
>
> > 用户界面渲染的工作会在这一步执行。
>
> #### yield WaitForEndOfFrame（协程：帧结束）
>
> > 当前帧彻底结束后会执行此协程。

### 暂停阶段

> #### OnApplicationPause（应用暂停）
>
> > 应用暂停时会调用此方法，取消暂停后会从FixedUpdate开始重新执行。bool pause标识是否暂停。

### 退出阶段

> #### OnDestroy（销毁）
>
> > 当物理被销毁时调用，一般用于清理内存。
>
> #### OnApplicationQuit（应用退出）
>
> > 当应用退出时调用，但有时会失效，此方法为不稳定的方法，正常情况下用于保存退出前的消息，但最好使用更稳妥的方式，比如Android原生接口，因为此方法有时不会被调用。

### 未激活状态下的哪些声明周期函数会执行

> Awake会执行，其他的都不会。假如GameObject本身是激活状态，在Awake里SetActive(false)之后，会执行一次OnDisable()，其他的都不执行。

### AddComponent之后会调用哪些生命周期函数

> Awake、OnEnable、Start。

### Start会调用几次

> 仅在首次启动之后调用一次。

### Update和FixedUpdate的区别

> Update每帧调用。
>
> FixedUpdate固定时间调用一次，假如卡顿了，FixedUpdate会尝试追回。
>
> ```c#
> Application.targetFrameRate = 30; // 帧率
> Time.fixedDeltaTime = 0.001f; // FixedUpdtae间隔时间
> ```





## 基础

### 如何增加删除获取组件

> 增加：AddComponent
>
> 删除：RemoveComponent
>
> 获取：GetComponent

### 记录节点空间集合信息的组件名称

> Transform，父类是Component

### 旋转

> ```c#
> transform.Rotate(new Vector3(45f, 45f, 45f)); // 基于当前状态旋转，默认基于自身的坐标系
> transform.Rotate(new Vector3(0, 0, .2f), Space.Self); // 绕自身坐标系旋转
> transform.Rotate(new Vector3(0, 0, .2f), Space.World); // 绕世界坐标系旋转
> 
> transform.Rotate(new Vector3(0, 20f, 0));
> transform.Rotate(new Vector3(10f, 0, 0));
> transform.Rotate(new Vector3(0, 0, 30f));
> //transform.Rotate(new Vector3(10f,20f,30f));
> ```
>
> 三轴的旋转顺序是：Y-X-Z（Windows10，DX11）。

### unity中保存和读取数据的类

> PlayerPrefs
>
> ```c#
> PlayerPrefs.SetInt(string key, int value);
> PlayerPrefs.GetInt(string key, int defaultValue = 0);
> PlayerPrefs.SetFloat(string key, float value);
> PlayerPrefs.GetFloat(string key, float defaultValue = 0.0F);
> PlayerPrefs.SetString(string key, string value);
> PlayerPrefs.GetString(string key, string defaultValue = "");
> ```

### 游戏动画类别

> - 关节动画：把角色分成若干独立部分，一个部分对应一个网格模型，部分的动画连接成一个整体的动画，角色比较灵活，Quake2中使用这种动画
> - 骨骼动画：广泛应用的动画方式，集成了以上两个方式的有点，骨骼按角色特点组成一定的层次结构，有关节相连，可做相对运动，皮肤作为单一网格蒙在骨骼之外，决定角色的外观。
> - 单一网格模型动画（关键帧动画）：由一个完整的网格模型构成，在动画序列的关键帧里记录各个顶点的原位置及其改变量，然后插值运算实现动画动过，角色动画较真实。

### 蒙皮如何跟着骨骼动

> Mesh是由顶点和面组成的，不绑定蒙皮数据的称之为静态mesh，不具有动画效果。
>
> 对于绑定蒙皮的mesh，称之为SkinMesh，在SkinMesh中每个mesh的顶点会受到若干个骨骼的影响，并配以一定的权重比例。
>
> 我们通常将计算Mesh的顶点受bone影响而产生的变化过程称之为蒙皮，如果在代码中计算的话，称之为cpu蒙皮，如果在gpu也就是顶点着色器中计算的话，称之为gpu蒙皮。
>
> 所有的计算并不包括世界坐标的变换，都是在mesh空间下进行的，mesh中原始的顶点坐标也是定义在mesh空间下的。当一个顶点受到多根骨骼的影响，加权平均。

### 两种阴影判断方法和工作原理

> 本影：景物表面上那些没有被光源直接照射的区域（全黑的轮廓分明的区域）。
>
> 半影：景物表面上那些被某些特定光源直接照射但并非被所有特定光源直接照射的区域（半明半暗区域）。
>
> 工作原理：从光源处向物体的所有可见面投射光线，将这些面投影到场景中得到投影面，再将这些投影面与场景中的其他平面求交得出阴影多边形，保存这些阴影多边形信息，然后再按视点位置对场景进行相应处理得到所要求的视图（利用空间换时间，每次只需依据视点位置进行一次阴影计算即可，省去了一次消隐过程）。

### 如何让已经存在的gameobject在LoadLevel后不被卸载掉

> ```c#
> DontDestroyOnLoad(gameObject);
> ```

### FSM有限状态机的实现

> FSM的组成：
>
> 1. 若干个状态
> 2. 起始状态
> 3. 状态间的转移条件
>
> 实现：哈希表存储状态及相应转移条件和目标状态。

### unity提供了哪几种光源

> - 点光源
> - 平行光
> - 聚光灯
> - 区域光源

### 拖拽的实现

> onMouseDown, onMouseDrag, onMouseUp

### 不同平台的图片压缩格式

> 安卓ETC，iOSPVRTC4位，pcDXT5；原因：跟各平台的硬件软件支持更匹配。



## UGUI

### Image和RawImgae的区别

> |          | Image          | RawImage |
> | -------- | -------------- | -------- |
> | 性能消耗 | 较大           | 较小     |
> | 图片源   | Sprite         | 都可以   |
> | 支持变换 | 裁剪平铺旋转等 | 不支持   |

### 如何处理适配问题

> 通过Canvas Scaler组件和锚点（Anchor）来处理。

### Anchor和Pivot的区别

> 锚点（Anchor）：影响自身在父节点中的布局，当x<sub>Min</sub>==x<sub>Max</sub>时，x方向的参数为Pos x和width。否则，x方向的参数为Left和Right。同理当y<sub>Min</sub>==y<sub>Max</sub>时，y方向的是Pos y和height。否则，y方向的参数为Top和Bottom。另外，当Left+Right>Width<sub>parent</sub>时，图片显示有问题。y方向同理。
>
> 中心点（Pivot）：自身旋转和缩放的中心点。

### 画布的三种渲染模式

> - 屏幕空间-覆盖模式（Screen Space-Overlay）：Canvas创建出来后，默认就是该模式，该模式和摄像机无关，即使场景里没有摄像机，UI照样渲染。只有x轴和y轴，UI元素永远在3D元素的前面。
> - 屏幕空间-摄像机模式（Screen Space-Camera）：设置成该模式后需要指定一个摄像机游戏物体，指定后UGUI就会自动出现在该摄像机的投射范围内，和NGUI的默认UI Root效果一致，如果隐藏掉摄像机，UGUI就无法渲染。
> - 世界空间模式（World Spcae）：设置成该模式后，UGUI就相当于是场景内的一个普通的“Cube 游戏模型”，可以在场景内任意地移动UGUI元素的位置，通常用于怪物血条显示和VR开发。

### UI的渲染顺序

> 组件层级顺序、渲染顺序设置、Canvas排序层级、画布模式

### UGUI层级计算原理

> 同一个Canvas内，UGUI的层级计算方式：
>
> 1. 一个UI元素，在它所占的屏幕范围内（通常是矩形），如果没有任何UI在它的底下，那么它的层级号就是0（最底下）；
> 2. 如果一个UI在其底下且该UI可以和它合批，那它的层级号与底下的UI层级一样；
> 3. 如果一个UI在其底下但是无法与它Batch，那它的层级号为底下的UI的层级+1；
> 4. 如果有多个UI都在其下面，那么按前两种方式遍历计算所有的层级号，其中最大的那个作为自己的层级号。

### UGUI合批

> 1. 纹理（图集）相同
> 2. 层级连续





## 物理

### 碰撞器和触发器的区别

> 碰撞器是触发器的载体，触发器只是碰撞器的一个属性。
>
> 当Is Trigger=false时，碰撞器根据物理引擎引发碰撞，产生碰撞的效果，对应的函数：OnCollisionEnter/Stay/Exit。
>
> 当Is Trigger=true时，碰撞器被物理引擎忽略，没有碰撞效果，对应的函数：OnTriggerEnter/Stay/Exit。
>
> 想检测物体的接触而不想产生碰撞可以使用触发器。

### 物体发生碰撞的必要条件

> 两个物体都带有碰撞器Collider，其中一个物体还必须带有Rigidbody刚体。

### CharacterController和Rigidbody的区别

> Rigidbody具有完全真实的物理特性，而CharacterController可以说是受限的Rigidbody，具有一定的物理效果但不是完全真实的。

### 施加力的方式

> rigidbody.AddForce
>
> rigidbody.AddForceAtPosition

### 链条关节

> Hinge Joint组件，可以模拟两个物体间用一根链条连接在一起的情况，能保持两个物体在一个固定距离内部相互移动而不产生作用力，但是达到固定距离后就会产生拉力。

### 物理更新一般在哪个生命周期函数里

> FixedUpdate，固定时间间隔调用。

### 当一个细小的高速物体撞向另一个较大的物体时，会出现什么情况，如何避免

> 穿透（碰撞检测失败）。可以考虑射线检测。

### 射线Raycast原理

> 从一个起点向一个方向发射一条物理射线，返回碰撞到的物体的碰撞信息。





## 网格

### MeshRender中material和sharedMaterial的区别

> 修改sharedMaterial会改变所有使用这个材质的物体的外观，并且也改变存储在工程里的材质设置。material只改变自身的。





## 资源管理与热更

### 如何安全地在不同工程间迁移asset数据

> 1. 将Assets和Library一起迁移
> 2. 导出包package
> 3. 用unity自带的assets Server功能

### 预制体prefab的用处

> prefab相当于一个模板，对已有的素材、脚本、参数等做了一个默认的配置，方便直接实例化以及后续的修改，同时prefab打包的内容简化了导出的操作。

### 动态加载资源的方式

> - instantiate：最简单的一种方式，以实例化的方法动态生成一个物体
> - Assetsbundle：将资源打包成assetsbundle，放在服务器或者本地磁盘，然后使用WWW模块get，然后从这个bundle中load某个object，unity官方推荐也是绝大多数商业化项目使用的一种方式。
> - Resource.Load：可以直接加载并返回某个类型的对象，前提是要把这个资源放在Resource命名的目录下，不管有没有场景引用，unity都会将其全部打入到安装包内。
> - AssetDatabase.loadasset：这种方式只在editor范围内有效，游戏运行时没有这个函数，通常是在开发中调试用的。

### AssetBundle加载

> #### LoadFromMemory（LoadFromMemoryAsync）
>
> ```c#
> IEnumerator Start()
> {
>   string path = "AssetBundles/wall.unity3d";
> 
>   AssetBundleCreateRequest request =AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
> 
>   yield return request;
>   
>   AssetBundle ab = request.assetBundle;
>   
>   GameObject wallPrefab = ab.LoadAsset<GameObject>("Cube");
>   
>   Instantiate(wallPrefab);
> }
> ```
>
> #### LoadFromFile（LoadFromFileAsync）
>
> ```c#
> IEnumerator Start()
> {
> string path = "AssetBundles/wall.unity3d";
> 
>   AssetBundleCreateRequest request = AssetBundle.LoadFromFileAsync(path);
> 
>   yield return request;
> 
>   AssetBundle ab = request.assetBundle;
> 
>   GameObject wallPrefab = ab.LoadAsset<GameObject>("Cube");
> 
>   Instantiate(wallPrefab);
> }
> ```
>
> #### UnityWebRequest
>
> ```c#
> IEnumerator Start()
> {
> 
>   string uri = @"http://localhost/AssetBundles/cubewall.unity3d";
> 
>   UnityWebRequest request =   UnityWebRequest.GetAssetBundle(uri);
> 
>   yield return request.Send();
> 
>   AssetBundle ab = DownloadHandlerAssetBundle.GetContent(request);
> 
>   GameObject wallPrefab = ab.LoadAsset<GameObject>("Cube");
> 
>   Instantiate(wallPrefab);
> }
> ```
>
> #### LoadAssetsByWWW（LoadFromCacheOrDownload）无依赖
>
> ```c#
> private IEnumerator LoadNoDepandenceAsset()
>     {
>         string path = "";
>         
>         if (loadLocal)
>         {
> #if UNITY_EDITOR_WIN || UNITY_STANDALONE_WIN
>             path += "File:///";
> #endif
> #if UNITY_EDITOR_OSX || UNITY_STANDALONE_OSX
>             path += "File://";
> #endif
>             path += assetBundlePath + "/" + assetBundleName;
>             
>             //www对象
>             WWW www = new WWW(path);
>  
>             //等待下载【到内存】
>             yield return www;
>  
>             //获取到AssetBundle
>             AssetBundle bundle = www.assetBundle;
>  
>             //加载资源
>             GameObject prefab = bundle.LoadAsset<GameObject>(assetRealName);
>  
>             //Test:实例化
>             Instantiate(prefab);
>         }
> ```
>
> #### LoadAssetsByWWW（LoadFromCacheOrDownload）有依赖
>
> ```c#
> using System.Collections;
> using System.IO;
> using UnityEngine;
> using UnityEngine.Networking;
>  
> public class LoadAssetsDemo : MonoBehaviour
> {
>     [Header("版本号")]
>     public int version = 1;
>     
>     [Header("加载本地资源")]
>     public bool loadLocal = true;
>     [Header("资源的bundle名称")]
>     public string assetBundleName;
>     [Header("资源的真正的文件名称")]
>     public string assetRealName;
>     
>     //bundle所在的路径
>     private string assetBundlePath;
>     //bundle所在的文件夹名称
>     private string assetBundleRootName;
>  
>     private void Awake()
>     {
>         assetBundlePath = Application.dataPath + "/OutputAssetBundle";
>         assetBundleRootName = assetBundlePath.Substring(assetBundlePath.LastIndexOf("/") + 1);
>         
>         Debug.Log(assetBundleRootName);
>     }
>  
> IEnumerator LoadAssetsByWWW()
>  
> {
>    string path="";
> //判断是不是本地加载
>    if(loadLocal)// loadLocal=true为本地资源
>    {
> #if UNITY_EDITOR_WIN || UNITY_STANDALONE_WIN
>         path+="File:///";
> #endif
> #if UNITY_EDITOR_OSX || UNITY_STANDALONE_OSX
>         path+="File://";
> #edif
>    }
>     //获取要加载的资源路径【bundle的总说明文件】
>    path+=assetBundle+"/"+assetBundleRootName;
>    //加载
>    WWW www=WWW.LoadFromCacheOrDownload(path,version);
>    yield return www;
>    //拿到其中的bundle
>    AssetBundle manifestBundle=www.assetsBundle;
>    //获取到说明文件
>    AssetBundleManifest manifest=manifest.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
>    //获取资源的所有依赖
>    string[] dependencies=manifest.GetAllDependencies(assetBundleName);
>    //卸载Bubdle和解压出来的manifest对象
>    manifestBundle.Unload(true);
>    //获取到相对路径
>    path =path.Remove(path.LastIndexOf("/")+1);
>    //声明依赖的Bundle数组
>    AssetBundle[] depAssetBundle=new AssetBundle[dependencies.Length];
>    
>    //遍历加载所有的依赖
>    for(int i=0;i<dependencies.Length;i++)
>    {  //获取到依赖Bundle的路径
>        string depPath=path+ dependencies[i];
>        //获取新的路径进行加载
>        www=WWW.LoadFromCacheOrDownload(depPath,version);
>        yield return www;
>        //将依赖临时保存
>        depAssetBundles[i]=www.assetsBundle;
>    }
>     
>    //获取路径
>    path+=assBundleName;
>    //加载最终资源
>    www=WWW.LoadFromCacheOrDownload(path,version);
>    //等待下载
>    yield return www;
>    //获取到真正的AssetBundle
>    AssetBundle realAssetBundle=www.assBunle;
>    //加载真正的资源
>    GameObject prefab=realAssetBundle.LoadAsset<GameObject>(assetBundle);
>    //生成
>    Instantiate(prefab);
>  
>    //卸载依赖
>    for(int i-0;i<depAssetBundle.Length;i++)
>     {
>        depAssetBundle[i].Unload(true);
>     }
>        realAssetBundle.Unload(true);
>     } 
> }
> ```

### AssetBundle卸载流程

> AssetBundle.Unload(bool)
>
> true卸载所有资源
>
> false只卸载没使用的资源，而正在使用的资源与AssetBundle依赖关系会丢失，调用Resources.UnloadUnusedAssets可以卸载。
>
> 或者等场景切换的时候自动调用Resources.UnloadUnusedAssets。

### unity常用资源路径有哪些

> ```c#
> //获取的目录路径最后不包含/
> //获得的文件目录最后包含/
> Application.dataPath;//Asset文件夹的绝对路径
> 
> Application.streamingAssetsPath;//只读，StreamingAssets的文件夹绝对路径
> Application.persistentData;//可读写
> 
> //资源数据库（AssetDatabase）是允许您访问工程中的资源的API
> AssetDatabase.GetAllAssetPaths;//获取所有的资源文件路径（不包含meta文件）
> AssetDatabase.GetAssetPath(object);//获取object对象的相对路径
> AssetDatabase.Refresh();//刷新
> AssetDatabase.GetDependencies(string);//获取依赖项文件
> 
> Directory.Delete(p, true);//删除p路径目录
> Directory.Exists(p);//是否存在p路径目录
> Directory.CreateDirectory(p);//创建p路径目录
> ```

### aa包

> Unity Addressables 是 Unity 引擎提供的一种高级资产管理系统，用于管理和加载游戏中的各种资源。它提供了更灵活的资源加载方式，能够优化内存和加载时间，提高游戏性能和开发效率。
>
> 使用 Addressables，你可以将游戏中的资源划分为地址化组件，每个组件都有一个唯一的地址标识符。你可以根据需要异步加载和卸载这些组件，而不是直接引用和加载整个资源。这样可以更加精细地控制资源的加载和卸载，减少内存占用和加载时间。
>
> Addressables 提供了以下一些主要功能：
>
> 1. 动态加载：可以根据需要在运行时异步加载和卸载资源，而不是一次性加载所有资源。
> 2. 按需加载：可以根据场景中的需求，按照资源的优先级和使用顺序进行加载，减少不必要的资源加载。
> 3. 远程加载：可以从远程服务器下载和更新资源，实现版本管理和在线更新。
> 4. 内容更新：可以将游戏内容进行分组管理，可以对指定的内容进行增量更新，而不是全量更新。
> 5. 资源定位：可以使用简单的地址标识符来定位和加载资源，无需手动管理文件路径。
> 6. 缓存管理：可以对已加载的资源进行缓存管理，减少重复加载和提高性能。
>
> 通过使用 Unity Addressables，你可以更好地管理游戏中的资源，优化游戏性能，并提供更灵活的资源加载方式，适应不同平台和需求的变化。





## 性能优化

### 哪些行为会打断渲染批次

> 1. 材质切换
> 2. 着色器切换
> 3. 纹理切换
> 4. 网格切换
> 5. 变换矩阵切换
> 6. 绘制顺序变化
> 7. 渲染状态修改
>
> > 在Unity渲染其实也是调用的OpenGL或者是DirectX，因为我只了解过OpenGL，所以以OpenGL为例（DirectX也是同理）。OpenGL中要渲染一个东西出来的，需要顶点着色器和片元着色器，这个对应的是ShaderLab中的顶点着色器和片元着色器，然后还需要把要渲染的网格的顶点属性作为一个数组绑定到VBO（顶点缓存对象）中去，然后绑定VAO（顶点数组对象）并设置顶点数组的属性，然后一些需要外部设置的着色器属性也是在这个阶段进行设置，当做完这些之后，再调用glDrawElements（也就是一次DrawCall）去渲染这个物体。这就是OpenGL渲染一个东西的最简单的流程。所以回到问题，为什么贴图，材质属性等必须一样呢？因为如果不一样的话，他们就不能通过一次DrawCall去设置这些属性和贴图了，想想看，如果A物体使用了贴图A，B物体使用了贴图B，如果把他们合并成一个大网格，要直接在一个DrawCall里渲染出来的话，这个合并好的大网格到底该用贴图A还是贴图B呢？无论使用哪个，都是不对的。所以A物体和B物体不能进行批处理。

### DrawCall影响

> 一次drawcall指调用一次gpu渲染函数。
>
> 减少drawcall不是为了减少这一行为本身，而是为了减少调用drawcall前的cpu消耗。cpu的内存读写、数据处理和渲染状态切换相对于gpu渲染来说非常慢，因此实际的瓶颈在CPU这边，因此要想办法降低drawcall的次数或者数据准备消耗。

### OverDraw

> Overdraw指屏幕上的某个像素在同一帧的时间内被绘制多次。过多的overdraw可能引起GPU过载，影响动画的播放和界面响应速度。
>
> **如何降低**
>
> > 1. 将Mask（自带2层Overdraw）替换为RectMask2D（自带1层Overdraw）
> > 2. 全屏遮挡的情况，则为遮挡的canvas添加canvas group组件，然后在被挡住时将其alpha值设为0，不参与绘制，没有drawcall也没有overdraw，同时顶点也不会参与绘制
> > 3. unity的文字显示中自带阴影组件shadow/outline，减少使用这些组件，自带overdraw
> > 4. UI摆放，减少UI的叠加效果
> > 5. 将不显示的物体直接禁用，而不是透明度改为0

### 性能优化方式

> #### 静态合批
>
> > static静态物体（永远不会移动、旋转和缩放），如果相同材质球，面数在一定范围之内，unity会自动合并成一个batch送往gpu处理
> >
> > **怎么做**
> >
> > > 在gameobject的Inspector面板右上角的Static勾选
> >
> > **优点**
> >
> > > 因为只需要进行一次，所以性能会比动态合批好
> >
> > **缺点**
> >
> > > 使用静态合批需要额外的内存开销来存储合并后的集合数据，因为需要额外维护多一份数据，所以包体会变大，占用的内存也会变多。
> > >
> > > 进行了静态批处理之后的gameobject不能在游戏运行时改变位置或者是跟渲染有关的设置。并且因为把所有要静态批处理的gameobject都合并成一个大网格保存起来，所以这实际上即使是同一个gameobject，也需要复制一份网格数据一起保存在这个大网格的顶点数据里面去，这样就导致了占用的内存变多了。
> >
> > **原理**
> >
> > > 在开始阶段把需要静态批处理的go进行一次网格合并操作，然后把这个合并之后的大网格保存起来，后续都是用这个网格而不需要再进行合并。
> > >
> > > 在预处理阶段，把一些材质相同的模型的顶点统一变换到世界空间坐标下，并且新构建一个大的VB把数据保存下来，在绘制时，就把这个大的VB提交上去，只需要设置一次渲染状态，再进行多次drawcall绘画出每个子模型。所以静态合批是不会减少drawcall的，但由于只修改了一次渲染状态依然可以减少CPU的消耗。
>
> #### 动态合批
>
> > 如果动态物体共用着相同的材质，那么Unity会自动对这些物体进行批处理。动态批处理操作是自动完成的，并不需要你进行额外的操作。
> >
> > **优点**
> >
> > > 不用自己做任何事情，unity会在游戏中自动进行动态批处理，只需要满足下述条件：
> > >
> > > 1. 顶点属性要小于900。例如，如果shader中需要使用顶点位置、法线和纹理坐标这三个顶点属性，那么要想让模型能够被动态批处理，它的顶点数据不能超过300。
> > > 2. 多pass的shader会中断批处理。
> > > 3. 使用lightingmap的物体需要小心处理，需要它们指向lightingmap的同一位置。
> >
> > **原理**
> >
> > > unity会检测哪些go使用了同一个共享材质，然后去合并这些使用了同一个共享材质的网格顶点数据，形成一个新的大网格，然后传给显存，直接渲染这个大网格就相当于渲染了所有的被合并的小网格，而这只需要一次drawcall。
>
> #### 动静合批的区别
>
> > 动态合批：处理一切都是自动的，不需要做任何操作，而且物体是可以移动的，但是限制很多。减少drawcall，发生在游戏运行时。
> >
> > 静态合批：自由度很高，限制很少，缺点是会占用很多内存，而且静态合批之后的所有物体都不可以再移动了。不减少drawcall，发生在场景加载时。
>
> #### 遮挡剔除
>
> > 当场景中包含大量模型时，drawcall就会非常大，造成的性能消耗也会比较大，从而会造成渲染效率降低，使用遮挡剔除技术，可以使那些被遮挡的物体不被渲染，达到提高渲染效率的目的。
> >
> > 在场景中创建一个区域，该遮挡区域由单元格组成，每个单元格构成整个场景的一部分，这些单元格会把整个场景拆分成多个部分，当摄像机能够看到该单元格时，表示该单元格中的物体会被渲染出来。
>
> #### 层消隐技术
>
> > 如果场景中存在大量小物件，则可以使用该技术来优化场景。在比较远的距离将小物体剔除以减少绘图调用的数量。
>
> #### LOD多层次细节
>
> > Level Of Detail
> >
> > 按照模型的位置和重要程度决定物体渲染的资源分配，降低非重要物体的面数和细节度，从而获得高效率的渲染运算。
> >
> > 优点：低面数模型渲染快，减轻cpu和gpu负担。
> >
> > 缺点：同一个物体要准备多个模型，消耗内存
>
> #### MipMap
>
> > 为了加快渲染速度和减少图像锯齿，贴图被处理成一系列被预先计算和优化过的图片组成的文件，这样的贴图被称为mipmap。
> >
> > 优点：优化显存带宽，用来减少渲染，根据距离摄像机远近选择适合的贴图。
> >
> > 缺点：运行时占用更多内存，且增加包的容量。（33%，等比数列求和极限）
> >
> > **原理**：
> >
> > Mipmap纹理技术是目前解决纹理分辨率与视点距离关系的最有效途径,它会先将图片压缩成很多逐渐缩小的图片,例如一张64*64的图片,会产生 64*64 , 32*32 , 16*16 , 8*8 , 4*4 , 2*2 , 1*1的7张图片,(从2的0次幂 到2的n次幂，n+1即为图片大小）当屏幕上需要绘制像素点为20*20 时，程序只是利用 32*32 和 16*16 这两张图片来计算出即将显示为 20*20 大小的一个图片，这比单独利用 32*32 的那张原始片计算出来的图片效果要好得多，速度也更快。
>
> #### UI图集
>
> > 图集就是碎图合成大图 降低内存，减少dc。
> >
> > UI图集有合批没有的优点，就是热更新的时候因为小文件变少了，所以会快一些。
> >
> > UI图集就是UI的动态合批。
> >
> > UI图集完成合批的条件是什么？
> >
> > 深度 贴图 材质 => 排序好的列表当前这个依次和前面对比是否贴图和材质ID相同决定是否合批。
>
> #### 光照贴图
>
> > 指在三维软件里实现打好光，然后渲染把场景各表面的光照输出到贴图上，最后又通过引擎贴到场景上，这样就使物体有了光照的感觉。而不是使用光源实时计算。
>
> #### 背包优化
>
> > tableview，循环复用列表
> >
> > 每个滚动条目都是同一个预设体的实例
> >
> > 做缓存，只要实例化视野范围内滚动条目，往上超出视野部分，自动填补到视野的下面
>
> #### AOI
>
> > 例如当mmo类型游戏中 存在多个玩家时，一个玩家移动，将通知所有玩家随之跟新，发送与接收数据太多造成卡顿，利用AOI机制（兴趣范围）
> >
> > 以状态修改的角色为原点，设置半径，获取该角色的兴趣范围，当其他角色存在该兴趣范围的时候，进行接收通知并更新状态，减少cpu压力。
> >
> > 优点：易于实现，且不需要特殊的数据结构。
> >
> > 缺点：计算成本较高：需要该角色与所有角色的距离判断。（1+n）n/2。
> >
> > 
> >
> > 优化：
> >
> > 多线程并行计算，提升计算效率（需要考虑线程切换本身的消耗）
> > 延迟计算，减少计算间隔（避免不必要的每帧都要计算）
> > 分批计算，降低每帧计算角色的数量。
> >
> > 优化设计方案：
> >
> > 由单项角色寻找变为双向寻找（每个角色都与其他角色遍历 变为 A与B计算过，B就不用与A计算）
> > 将地图优化为格子，计算每个格子中的角色
> > 区域大小改为2的N次方表示，角色所区域可以利用位运算来计算。
> > 将计算的区域变小，虽然格子变多，但格子里的角色变少，需要计算的量也会变少。
> > 扩大地图比例，玩家不变，玩家更加稀疏，优化算法。
>
> #### GPU实例化
>
> > 将多个相同的网格数据在GPU上复制成多个实例，然后使用单个渲染调用同时绘制多个实例。Graphics.DrawMeshInstanced
>
> #### 动静分离
>
> > 将需要动的和不需要动的ui元素分别合并。原因是需要移动的ui元素会重构ui，动静分离可以减少cpu在重绘和合并时的消耗。

#### 内存优化

> 压缩自带类库;
> 将暂时不用的以后还需要使用的物体隐藏起来而不是直接Destroy掉;
> 释放AssetBundle占用的资源;
> 降低模型的片面数，降低模型的骨骼数量，降低贴图的大小;
> 使用光照贴图，使用多层次细节(LOD)，使用着色器(Shader)，使用预设(Prefab)。
> 警惕配置表内存占用
> 排查项目冗余的shader
> 减少容器扩容或者利用string字符串拼接等一系列产生GC的操作