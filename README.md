# Unity_UI_MVC_EventCenter_Singleton
Unity运用MVC思想+事件中心的UI

项目简介

Unity轻量级解耦框架，内涵事件中心模块，MVC框架

操作步骤：将文件夹放入Asset文件直接使用

框架架构
流程图展示事件驱动过程（mermaid语法）
#单例 #事件中心 #MVC框架

功能清单：

单例模式：
不继承Mono的单例模式  
通过继承BaseManager<泛型填写继承类>成为单例模式类
继承BaseManager的类必须实现私有构造函数
如果存在多线程，为方法加锁lock(lockObj){}（lockObj来自父类）
使用时直接类名点Instance点想使用的方法
继承Mono自动挂载的单例模式
需要是继承Mono的类，通过继承SingletonAutoMono<泛型填写继承类>在运行开始时会在场景中自动创建一个名为继承类的空对象来挂载继承的类
不用担心切换场景挂载对象消失
不能通过多线程访问继承Mono的单例模式类对象
使用时直接类名点Instance点想使用的方法
继承Mono手动挂载的单例模式
通过继承SingletonMono<泛型填写继承类>成为单例模式类
可以通过重写Awake生命周期函数来进行初始化
会自动删除重复挂载的单例脚本
不用担心切换场景挂载对象消失事件中心

事件中心模块：
事件触发方法EventCenter.EventTrigger<T>(E_EventType.事件名,T 参数信息)
事件触发方法（无参数）EventCenter.EventTrigger(E_EventType.事件名)
添加事件监听EventCenter.AddEventListener<T>(E_EventType.事件名，UnityAction<T> 添加的逻辑处理)
添加事件监听（无参数）EventCenter.AddEventListener(E_EventType.事件名，UnityAction 添加的逻辑处理)
添加事件监听的类在销毁时一定要移除添加的对应监听，不然会导致内存泄露
不要使用匿名函数和lambda表达式添加事件监听，因为无法移除事件
移除事件时泛型参数一定要和添加时相同，否则无法移除
外部类添加监听逻辑处理函数函数名(T 参数信息)
移除事件监听EventCenter.RemoveEventListener<T>(E_EventType.事件名，<T>UnityAction 移除的逻辑处理)
移除事件监听（无参数）EventCenter.RemoveEventListener(E_EventType.事件名，UnityAction 移除的逻辑处理)
移除所有事件与事件监听EventCenter.Clear()
移除指定事件的所有事件监听EventCenter.Clear(E_EventType.事件名)
事件类型枚举：
打开E_EventType枚举添加事件类型，格式可以如下
/// <summary>
/// 怪物死亡事件 —— 参数：Monster
/// </summary>
E_Monster_Dead,

MVC框架
将UI制作成预设体存储入Resources
预制体UI挂载控制层与界面层脚本

Controller文件夹 控制层
继承Mono
继承BaseController，在三个泛型中填写对应层的类
使用instance对控制层进行访问
实现界面显隐方法：
显示界面
public static void ShowMe()
OnShow方法重写可实现以下逻辑和其他实现显示界面时的逻辑：
GameObject res = Resources.Load<GameObject>("UI/"+ UI预制体名);
GameObject obj = Instantiate(res);
//设置面板父对象
obj.transform.SetParent(GameObject.Find("Canvas").transform, false);
隐藏界面
public static void HideMe()
OnHide方法重写可以实现在隐藏界面时的逻辑

用生命周期Start进行初始化，如下
private void Start()
{
    //填写UI面板控件的事件的监听，处理对应的逻辑
    //告知数据模块 当更新时 通知哪个函数做处理
EventCenter.Instance.AddEventListener<数据层类>(事件枚举名,UpdateInfo);
}

填写对应UI控件处理的逻辑函数
在脚本销毁时，移除对应的界面更新函数委托，如下
private void OnDestroy()
{
EventCenter.GetInstance().RemoveEventListener<数据层类>(事件枚举名,UpdateInfo);
}
Model文件夹 数据层
不继承Mono
继承BaseModel
填写UI面板所有成员变量（如private int hp;)
填写UI面板所有成员变量属性（如public int Hp{get{return hp;}}）

通知外部更新数据的方法
private void UpdateInfo()
{
EventCenter.Instance.EventTrigger<数据层类名>(事件枚举名,this);
}
成员变量初始化方法（如public void Init(){HP=PlayerPrefs.GetInt(“hp”,10);}）
外部调用改变数据逻辑（如public void Recover(){hp+=1;}）改变数据后使用保存成员变量方法
成员变量保存方法（如public void SaveData(){PlayerPrefs.SetInt(“hp”,hp);}在保存变量方法中使用执行外部更新方法

View文件夹 界面层
继承Mono
继承BaseView
填写UI面板控件（如public Text txtHp）
提供面板更新相关方法给外部（如public void UpdateInfo(数据层类名 data){txtHp.text=data.Hp.ToString();}）
