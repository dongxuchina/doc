安卓开发规范
======
<br>

## 编码
UTF-8
___
<br>

## 缩进
Tab键<br>
设置IDE的Tab键为4个空格，用IDE将代码格式化后，Tab 会转换成4个空格。<br>
#### 注：不要连敲4个空格，这样容易多敲或少敲，也影响开发效率。<br>
___
<br>

## 代码行宽
每行代码不超过160个字符（待定）<br>
设置IDE的Maximum line width = 160（待定）<br>
___
<br>

## 换行规范、大括号规范、对齐规范等
完全参照IDE的默认格式化模版，格式后就是标准模式
___
<br>

## 文件夹和包命名规范
全部由小写字母组成，如：<br>
1.文件夹名-com/xiaoyun/app/android/ui/helper/mvp<br>
2.包名-com.xiaoyun.app.android.ui.helper.mvp<br>
___
<br>

## 类文件和类命名规范
UpperCamelCase驼峰法且首字母大写，如：<br>
1.类文件名-NetworkChangeReceiver.java<br>
2.类名-NetworkChangeReceiver<br>

#### 常用类名命名规范
##### Activity类名
以Activity结尾，如：UserLoginActivity<br><br>

##### Fragment类名
以Fragment结尾，如：UserLoginFragment<br><br>

#### 注：类名前不要加模块名，如：直播模块，模块名是Live，录制类不要是LiveRecordActivity，直接RecordActivity即可
___
<br>

## 资源文件命名规范
全部由小写字母组成，每个单词之间用下划线"_"连接<br>
#### Activity Layout xml文件名
以activity开头，如：UserLoginActivity -  activity_user_login.xml<br>
#### 注：用Android Studio创建Activity时，选中Generate Layout File，自动生成的Layout xml文件名符合这种格式<br><br>

#### Fragment Layout xml文件名
以fragment开头，如：UserLoginFragment - fragment_user_login.xml<br><br>

#### Item Layout xml文件名
以item开头，如：item_user_profile.xml<br><br>

#### Dialog Layout xml文件名
以dialog开头，如：dialog_change_password.xml<br><br>

#### 只被其他Layout引用的Layout xml文件名
以include开头，如：include_user_header.xml<br><br>

#### 图片文件命名
图片由设计切图和命名，不便给设计提规范，如果我们不改 文件名的话，暂时不提供规范了。<br><br>

#### 注：资源文件名前不要加模块名，如：直播模块，模块名是Live，录制资源文件名不要是activity_live_record.xml或live_activity_record.xml，直接activity_record.xml即可
___
<br>

## 变量命名规范
lowerCamelCase驼峰法且首字母小写<br>
* 静态全局变量使用s前缀
* 非公共、非静态全局变量使用m前缀
* 集合类型的变量使用类型作为后缀
* 布尔变量名使用is前缀
<br>
如：<br>
```Java
public class MyClass {
    private static MyClass sSingleton;
    public int publicField;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
    public List scopeList;
    private Map mScoreMap;
    private boolean mIsLogin;
    public boolean isUpdate;
}
```
___
<br>

## 常量命名规范
全部由大写字母组成，每个单词之间用下划线"_"连接，如：public static final int SOME_CONSTANT = 22;
___
<br>

## 方法命名规范
lowerCamelCase驼峰法且首字母小写，如：setUserName()、isLogin()<br>
#### 注：Getter和Setter方法不要m前缀，设置IDE在Generate Getter和Setter时省略前缀m
___
<br>

## 一些不好的命名习惯
推荐  | 反对
------------- | -------------
XmlHttpRequest  | XMLHTTPRequest
getCustomerId  | getCustomerID
String url = "http://goyoo.com";  | String URL = "http://goyoo.com";
long id = 100;  | long ID = 100;
___
<br>

## 控件缩写规范
控件名  | 缩写
------------- | -------------
View  | v
EditText  | et
TextView  | tv
ImageView  | iv
ListView  | lv
GridView  | gv
ScrollView  | sv
WebView  | wv
VideoView  | vv
Button  | btn
ImageButton  | ibtn
RadioButton  | rbtn
ToggleButton  | tbtn
ProgressBar  | pbar
SeekBar  | sbar
RatingBar  | rbar
TextSwitcher  | tswt
ImageSwitcher  | iswt
ExpandableList  | epdl
Spinner  | spn
Gallery  | gal
RelativeLayout  | rlay
LeanerLayout  | llay
TableLayout  | tlay
FrameLayout | flay
___
<br>

## 控件ID命名规范
全部由小写字母组成, 每个单词之间用下划线"_"连接，且以控件缩写开头<br/>
如：@+id/tv_article_title
___
<br>

## 注释规范
#### 类注释
在类的开头添加类信息及开发者信息，IDE可以设置注释模版，如：<br>
```Java
/**
 * MyClass
 * Created by Creator 16/4/19 下午10:51
 * Copyright (c) 2014 小云社群. All rights reserved
 */
public class MyClass {
}
```
#### 方法注释
```Java
/**
 * 登录方法
 * @param name 用户名称
 * @param password 密码
 * @return 是否登录成功
 * @throws NullPointerException 导致抛出异常的原因
 */
public boolean login(String name, String password) throws NullPointerException {
    //登录成功
    return true;
}
```
#### 全局变量和常量注释
```Java
/**
 * Discuz社区
 */
public static final String FORUM_DISCUZ = "discuz";

/**
 * 用户名
 */
private String name;
```
#### XML注释
```Java
<!-- Base application theme. -->
<style name="AppTheme" parent="android:Theme.Holo.Light.NoActionBar">
<!-- Customize your theme here. -->
</style>
```
#### 特殊注释:
// TODO: 此处代码未完成, 待完善<br>
// FIXME: 此处代码有Bug, 待修复<br>
如：<br>
```Java
public class MyClass {
    public void todoMethod() {
        // TODO: 2016/08/12
        // FIXME: 2016/08/12
    }
}
```
___
<br>

## 编码规范
#### Activity和Fragment使用原则（待定）
Activity中不再处理View业务，Activity只处理Fragment业务，由Fragment完全取代Activity处理View业务，每个Activity中至少有1个Fragment。<br>
<br>

#### 禁止直接修改Activity和Fragment的全局变量
```Java
public class MainActivity extends Activity {
    public String mName;
}

MainActivity activity = new MainActivity();
activity.mName = "abc"; // 禁止
```

#### 获取资源：
```Java
// 推荐：
mFlayFragmentContainer = findViewById(R.id.flay_fragment_container);
mFlayFragmentContainer.setVisibility(View.VISIBLE);
// 反对：
findViewById(R.id.flay_fragment_container).setVisibility(View.VISIBLE);
```

#### Activity的onCreate中初始化的全局变量，在onResume中使用前需要判null（待定）
```Java
public class MainActivity extends Activity {
    private MyClass mObj;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        mObj = new MyClass();
    }

    @Override
    protected void onResume() {
        super.onResume();

        // 推荐：
        if (mObj != null) {
            mObj.do();
        }

        // 反对：
        mObj.do();
    }
}
```

#### 字符串比较：
```Java
class MyClass {
    public static final String SOME_CONSTANT = "abc";

    public void do(String name) {
        // 推荐：
        SOME_CONSTANT.equals(name);
        // 反对：
        name.equals(SOME_CONSTANT);

        // 推荐：
        "".equals(name);
        // 反对：
        name.equals("");
        
        // 应该用 TextUtils.isEmpty(name); 取代 "".equals(name); 
    }
}
```

#### 使用DZLogUtil类代替安卓自带的Log类
<br>

#### 父类的私有变量，不要向子类公开，用private代替protected，子类想操作父类变量，也需要通过父类提供的Getter和Setter方法
<br>

#### 尽量用foreach取代for和while
<br>

#### 当for循环中有很多if嵌套时，用continue使代码看起来更清晰
```Java
// 推荐：
for () {
    if (false) {
        continue;
    }

    if (false) {
        continue;
    }

    if (false) {
        continue;
    }
}

// 反对：
for () {
    if (true) {
        if (true) {
            if (true) {
            }
        }
    }
}
```

#### 除法运算前，除数需要判0
```Java
int dividend = 10;
int divisor = 5;
int result;
if (divisor != 0) {
    result = dividend / divisor;
}
```

#### 类中尽量保存getApplicationContext，而不是Context，一些特殊情况除外
```Java
class MyClass {
    private final Context mContext;

    public MyClass(Context context) {
        // 推荐：
        mContext = context.getApplicationContext();
        // 反对：
        mContext = context;
    }
}
```

#### StringBuffer和StringBuilder的append方法
```Java
StringBuilder command = new StringBuilder();
String tblName = "contacts";
// 推荐：
command.append("SELECT * FROM ").append(tblName);
// 反对：
command.append("SELECT * FROM " + tblName);
```

#### 捕获的异常不能忽略
```Java
try {
    int i = Integer.parseInt(response);
} catch (NumberFormatException e) {
    // 可以抛出新异常或打日志，但不能什么也不干，如果什么都不干，出错就没办法找了。
}
```

#### 用JSONObject生成JSON，不要自己拼接
```Java
// 推荐：
JSONObject userInfo = new JSONObject();
try {
	userInfo.put("user_id", 1000);
	userInfo.put("user_name", "abc");
} catch (JSONException e) {
	e.printStackTrace();
}

String json = userInfo.toString();

// 反对：
String json = "{\"user_id\": 1000, \"user_name\": \"abc\"}";
```

#### XML编码规范
```Java
// 推荐：
<View
    android:id="@+id/v_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />

// 反对：
<View
    android:id="@+id/v_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" >
</View>
```
___
<br>
