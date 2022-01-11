# MyIssue

### Issue_1：引入的三方库使用v4/v7库，而AppModule使用的是AndroidX
gradle.properties中增加
```
android.useAndroidX=true
android.enableJetifier=true
```

### Issue_2：部分全面屏机型，界面下方有白条
查看下Theme设置以及状态栏设置（透明状态栏）

### Issue_3：设置状态栏颜色
```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    val window = window
    window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS)
    window.statusBarColor = color
}
```

### Issue_4：StringSpannable使用
```
private fun changeTextColor(source: String, targetView: TextView) {
    val termForegroundColorSpan = ForegroundColorSpan(Color.parseColor("#FF1CD28B"))

    val spannableStringBuilder = SpannableStringBuilder()
    val termContent = "个"
    val termsIndex = source.indexOf(termContent) + 1

    spannableStringBuilder.append(source)

    spannableStringBuilder.setSpan(
        termForegroundColorSpan, 0, termsIndex, Spanned.SPAN_INCLUSIVE_INCLUSIVE
    )

    targetView.highlightColor = Color.TRANSPARENT
    targetView.movementMethod = LinkMovementMethod.getInstance()
    targetView.text = spannableStringBuilder
}
```

### Issue_5：Kotlin接口使用建议
```
context?.let { _context ->
        if (_context is ItemBoxSelectInterface) {
            _context.itemItemSelectNotify()
        }
    }
```
在Activity中实现`ItemBoxSelectInterface`接口就行了

### Issue_6：调用系统方式查看img/video
```
 public static void lookupImgSource(Context context, File file) {
    Uri uri;
    Intent intent = new Intent(Intent.ACTION_VIEW);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        uri = FileProvider.getUriForFile(context, context.getPackageName() + ".fileprovider", file);
    } else {
        uri = Uri.fromFile(file);
    }
    intent.setDataAndType(uri, "image/*");

    try {
        context.startActivity(intent);
    } catch (Exception e) {
        LogUtil.print("lookupImgSource e: " + e.getMessage());
    }
}

public static void lookupVideoSource(Context context, File file) {
    Uri uri;
    Intent intent = new Intent(Intent.ACTION_VIEW);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        uri = FileProvider.getUriForFile(context, context.getPackageName() + ".fileprovider", file);
    } else {
        uri = Uri.fromFile(file);
    }
    intent.setDataAndType(uri, "video/*");

    try {
        context.startActivity(intent);
    } catch (Exception e) {
        LogUtil.print("lookupImgSource e: " + e.getMessage());
    }
}
```

### Issue_7：file.delete无效
出现在Android高版本上，使用DocumentFile删除
```
public static void deleteTest(Context mContext, File file) {
    if (PermissionUtil.isAndroid5() && (FileUtil.getDocumentFile(file, false, false, mContext) != null
            || FileUtil.getDocumentFile(file, true, false, mContext) != null)) {
        mResult = (FileUtil.deleteFile(file, mContext)) ? 1 : 0;
    } else {
        File[] files = file.listFiles();
        if (files != null && files.length != 0)
            for (File childFile : files) {
                if (childFile.isDirectory()) {
                    deleteTest(mContext, childFile);
                } else {
                    boolean res = childFile.delete();
                    mResult *= res ? 1 : 0;
                }
            }
        boolean res = file.delete();
        mResult *= res ? 1 : 0;
    }
}
```

```
public static boolean isAndroid5() {
    return Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP;
}
```

### Issue_8：签名
命令格式：jarsigner -verbose -keystore [签名文件路径] -signedjar [签名后apk的文件路径] [未签名apk的文件路径] [证书别名]

### Issue_9：SHA1
`keytool -list -printcert -jarfile [*].apk`

### Issue_10：Dialog Match时左右不平铺
setBackgroundDrawable
```
private val confirmDialog by lazy {
    val dialog = Dialog(this)
    val view = View.inflate(this, R.layout.dialog_uninstall_app, null)
    dialog.setContentView(view)

    val window = dialog.window
    window?.setGravity(Gravity.BOTTOM)
    window?.setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT)
    window?.setBackgroundDrawable(resources.getDrawable(R.drawable.bg_dialog))

    view.findViewById<TextView>(R.id.tv_cancel).setOnClickListener(this)
    view.findViewById<TextView>(R.id.tv_uninstall).setOnClickListener(this)
    dialog
}
```

### Issue_11：Kotlin Handler
```
private val handler: Handler = Handler {
    if (it.what == Msg_Jump_Main) {
        gotoMain()
    }
    false
}
```

```
private var mTimeHandler = Handler(Looper.getMainLooper())
```

### Issue_12：Kotlin Dialog
```
private val confirmDialog by lazy {
    val dialog = Dialog(this)
    val view = View.inflate(this, R.layout.dialog_delete_app, null)
    dialog.setContentView(view)

    val window = dialog.window
    window?.setGravity(Gravity.BOTTOM)
    window?.setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT)

    view.findViewById<TextView>(R.id.tv_cancel_delete_img).setOnClickListener(this)
    view.findViewById<TextView>(R.id.tv_delete_img).setOnClickListener(this)
    dialog
}
```

### Issue_13：Kotlin 延时
delay(50L)

### Issue_14：Kotlin 保留两位小数
```
val result = curRandomNum + it.animatedFraction * 10

val format = DecimalFormat("0.##")
format.roundingMode = RoundingMode.FLOOR
val speed = format.format(result)

tv_optimize_net_title.text = speed
```

### Issue_15：Gradle 混淆
排除是否混淆导致问题时，可以通过关闭混淆开关来验证
```
shrinkResources false
 minifyEnabled false
```

### Issue_16：ImageView tink属性
二次着色


### Issue_17：SpannableString使用
```
val privacyStr = activity.getString(R.string.privacy_agreement)
        val privacySp = SpannableString(privacyStr)
        privacySp.setSpan(
            object : ClickableSpan() {
                override fun onClick(widget: View) {
                    WebviewActivity.start(
                        getActivity(),
                        activity.getString(R.string.privacy_agreement),
                        StaticUrl.getPrivacyUrl()
                    )
                }

                override fun updateDrawState(ds: TextPaint) {
                    ds.isUnderlineText = false
                }
            },
            0,
            privacySp.length,
            Spannable.SPAN_INCLUSIVE_EXCLUSIVE
        )
        val termsOfUsStr = activity.getString(R.string.terms_of_us)
        val termsOfUsSp = SpannableString(termsOfUsStr)
        termsOfUsSp.setSpan(
            object : ClickableSpan() {
                override fun onClick(widget: View) {
                    WebviewActivity.start(getActivity(), activity.getString(R.string.terms_of_us), StaticUrl.getTermsOfUsUrl())
                }

                override fun updateDrawState(ds: TextPaint) {
                    ds.isUnderlineText = false
                }
            },
            0,
            termsOfUsSp.length,
            Spannable.SPAN_INCLUSIVE_EXCLUSIVE
        )

        val fontBoldSpan = StyleSpan(Typeface.BOLD)
        val fontBoldSpan2 = StyleSpan(Typeface.BOLD)

        val fCp2 =
            ForegroundColorSpan(ContextCompat.getColor(activity, R.color.privacy_file_name_color))
        val fCp3 =
            ForegroundColorSpan(ContextCompat.getColor(activity, R.color.privacy_file_name_color))
        privacySp.setSpan(fontBoldSpan, 0, privacySp.length, Spannable.SPAN_INCLUSIVE_EXCLUSIVE)
        termsOfUsSp.setSpan(fontBoldSpan2, 0, termsOfUsSp.length, Spannable.SPAN_INCLUSIVE_EXCLUSIVE)

        val cs =
            activity.getString(R.string.privacy_desc_detail)
        val csS = cs.split("—s")

        val contextStrBuilder =
            SpannableStringBuilder()
                .append(csS[0])
                .append(termsOfUsSp)
                .append(csS[1])
                .append(privacySp)
                .append(csS[2])
```

```
private fun changeHintTextColor(source: String, targetWord: String) {
        val termForegroundColorSpan = ForegroundColorSpan(Color.parseColor("#77D388"))

        val spannableStringBuilder = SpannableStringBuilder()

        val termsStartIndex = source.indexOf(targetWord)

        spannableStringBuilder.append(source)

        spannableStringBuilder.setSpan(
            termForegroundColorSpan, termsStartIndex, termsStartIndex + targetWord.length,
            Spanned.SPAN_INCLUSIVE_INCLUSIVE
        )

        frag_clear_mid_layout_net_optimized_des!!.highlightColor = Color.TRANSPARENT
        frag_clear_mid_layout_net_optimized_des!!.movementMethod = LinkMovementMethod.getInstance()
        frag_clear_mid_layout_net_optimized_des!!.text = spannableStringBuilder
    }
```

### Issue_18：Kotlin方法参数使用
某些时候可以代替接口，更为方便轻量
```
showPrivacy(this, binding.root) {
    startMain()
}

 fun showPrivacy(
        activity: AActivity,
        rootView: ViewGroup,
        agreeInvoke: () -> Unit
    ): PopupWindow {

    	view.dialogPrivacyAgree.setOnClickListener {
            pop.dismiss()
            SettingSp.setAgreePrivacy()
            agreeInvoke.invoke()
        }

}
```

### Issue_19：
webView进程问题
```
 if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        String processName = getProcessName();
        WebView.setDataDirectorySuffix(processName);
}
```

### Issue_20：导入新项目时，AS报错 `Invalid Gradle JDK configuration found. Open Gradle Settings Change JDK location` 
此时删除掉所有的 `.idea` 和 `.gradle`文件，重新rebuild


### Issue_21：
tools:replace="android:taskAffinity , android:resource"

### Issue_22：
UnsatisfiedLinkError 检查下JNI方法名

### Issue_23：圆点指示器动画
提供一种思路：使用AnimationDrawable
```
private val leftScrollAnimation by lazy {
        val animationDrawable = AnimationDrawable()
        animationDrawable.isOneShot = true
        for (i in 1..5) {
            val bitmap = BitmapFactory.decodeStream(assets.open("anim_$i.png"))
            animationDrawable.addFrame(BitmapDrawable(resources, bitmap), frameDuration)
        }
        animationDrawable
    }

private val rightScrollAnimation by lazy {
    val animationDrawable = AnimationDrawable()
    animationDrawable.isOneShot = true
    for (i in 5 downTo 1) {
        val bitmap = BitmapFactory.decodeStream(assets.open("anim_$i.png"))
        animationDrawable.addFrame(BitmapDrawable(resources, bitmap), frameDuration)
    }
    animationDrawable
}

iv_frame_animation.setImageDrawable(leftScrollAnimation)
leftScrollAnimation.start()
```

### Issue_24：Android保存图片并通知图库刷新
```
public class BitmapUtil {
    public static void saveImageToGallery(Context context, Bitmap bmp) {
        // 首先保存图片
        File appDir = new File(Environment.getExternalStorageDirectory(), context.getString(R.string.app_name));
        if (!appDir.exists()) {
            appDir.mkdir();
        }
        String fileName = System.currentTimeMillis() + ".jpg";
        File file = new File(appDir, fileName);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            bmp.compress(Bitmap.CompressFormat.JPEG, 100, fos);
            fos.flush();
            fos.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 其次把文件插入到系统图库
        try {
            MediaStore.Images.Media.insertImage(context.getContentResolver(),
                    file.getAbsolutePath(), fileName, null);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        // 最后通知图库更新
        context.sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, Uri.parse("file://" + file.getAbsolutePath())));
    }
}
```

### Issue_25：XML线背景
```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#F6C995">
    <item>
        <shape android:shape="rectangle">
            <stroke
                android:width="1dp"
                android:color="@color/pro_grey" />
            <corners android:radius="8dp" />
        </shape>
    </item>

</ripple>

```

### Issue_26：内联函数，防止view多次点击
```
inline fun View.setSafeListener(crossinline action: () -> Unit) {
    var lastClick = 0L
    setOnClickListener {
        val gap = System.currentTimeMillis() - lastClick
        lastClick = System.currentTimeMillis()
        if (gap < 1500) return@setOnClickListener
        action.invoke()
    }
}
```

### Issue_27：XML渐变背景
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:layout_width="122dp"
        android:layout_height="23dp" />
    <corners android:radius="8dp" />
    <gradient
        android:endColor="#00FFDAA2"
        android:startColor="#FFFBD390" />

</shape>
```

### Issue_28：动态修改margin

```kotlin
val guideNextLayoutParams = tv_guide_next.layoutParams
if (guideNextLayoutParams is ViewGroup.MarginLayoutParams) {
		guideNextLayoutParams.bottomMargin = 40f.dp.toInt()
}
```

### Issue_29：overridePendingTransition Activity动画

> override pending transition 覆盖 即将到来 动画

通过这个方法添加的跳转动画会覆盖掉即将到来的跳转动画效果

Activity的切换动画从业务层面上来说可以分为两种：

1. Activity启动时的动画，
2. 从Activity返回时的动画，

它们都可以通过overridePendingTransition()来设置

要设置启动时的动画需要在执行startActivity()或startActivityForResult()之后调用overridePendingTransition()

要设置返回时的动画需要在finish()之后调用overridePendingTransition()

启动动画和返回动画是相互独立的，设置启动动画不会对返回动画产生影响

如果只在startActivity()或startActivityForResult()之后调用了overridePendingTransition()，没有在finish()的时候调用，则Activity返回的时候仍然是默认的动画效果，也可以在finish()的时候使用和启动时不同的动画效果

### Issue_30：震动

```kotlin
private fun vibrate(millisecond: Long = 300) {
        val vibrateService = getSystemService(Service.VIBRATOR_SERVICE) as Vibrator
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            vibrateService.vibrate(VibrationEffect.createOneShot(millisecond, 10))
        } else {
            vibrateService.vibrate(millisecond)
        }
    }
```

### Issue_31：ConstraintLayout maxWidth

```
app:layout_constraintWidth_max="wrap"  <!-- 设置为 wrap -->
app:layout_constraintWidth_percent="0.1"  <!-- 设置为想要的比例 -->
android:layout_width="0dp"  <!-- 注意是 0dp -->
```

### Issue_32：ConstraintLayout layout_constraintDimensionRatio

宽高比设置

```
android:layout_width="match_parent"
android:layout_height="0dp"
android:layout_marginStart="20dp"
android:layout_marginEnd="20dp"
android:background="@drawable/bg_white_corner_8"
android:visibility="gone"
app:layout_constraintDimensionRatio="h,1:1.18"
app:layout_constraintTop_toBottomOf="@+id/span_bar"
```

高：宽 == 1.18

###  Issue_33：Activity背景透明，DialogActivity

```
    <style name="dialogActivityTheme" parent="Theme.AppCompat.Light.Dialog">
        <!--设置dialog的背景-->
        <item name="android:background">#00000000</item>
        <item name="android:windowBackground">@color/transparent</item>
        <item name="android:windowFrame">@null</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowIsFloating">false</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowContentOverlay">@null</item>
        <item name="android:backgroundDimEnabled">true</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
```

###  Issue_34：体外无权限情况下，获取ssid
```
private val WIFISSID_UNKNOW = "<unknown ssid>"

    private fun getWifiSSID(context: Context): String {
        val wifiManager: WifiManager = context.applicationContext.getSystemService(WIFI_SERVICE) as WifiManager
        val info = wifiManager.connectionInfo
        val wifiId = info.ssid
        var result = wifiId.trim { it <= ' ' }
        if (!TextUtils.isEmpty(result)) {
            if (result[0] == '"' && result[result.length - 1] == '"') {
                result = result.substring(1, result.length - 1)
            }
        }
        if (TextUtils.isEmpty(result) || WIFISSID_UNKNOW.equals(result.trim { it <= ' ' }, ignoreCase = true)) {
            val networkInfo = getNetworkInfo(context)
            networkInfo?.let { _networkInfo ->
                if (_networkInfo.isConnected) {
                    if (_networkInfo.extraInfo != null) {
                        result = _networkInfo.extraInfo.replace("\"", "")
                    }
                }
            }
        }
        if (TextUtils.isEmpty(result) || WIFISSID_UNKNOW.equals(result.trim { it <= ' ' }, ignoreCase = true)) {
            result = getSSIDByNetworkId(context)
        }
        return result
    }

    private fun getNetworkInfo(context: Context): NetworkInfo? {
        try {
            val connectivityManager: ConnectivityManager = context.getSystemService(CONNECTIVITY_SERVICE) as ConnectivityManager
            return connectivityManager.activeNetworkInfo
        } catch (e: java.lang.Exception) {
            e.printStackTrace()
        }
        return null
    }

    @SuppressLint("MissingPermission")
    private fun getSSIDByNetworkId(context: Context): String {
        var ssid = WIFISSID_UNKNOW
        val wifiManager = context.applicationContext.getSystemService(WIFI_SERVICE) as WifiManager
        val wifiInfo = wifiManager.connectionInfo
        val networkId = wifiInfo.networkId
        val configuredNetworks: List<WifiConfiguration> = wifiManager.configuredNetworks
        for (wifiConfiguration in configuredNetworks) {
            if (wifiConfiguration.networkId == networkId) {
                ssid = wifiConfiguration.SSID
                break
            }
        }
        return ssid
    }
```

###  Issue_35：获取运营商
```
private fun getOperators(context: Context): String {
        val tm = context.getSystemService(TELEPHONY_SERVICE) as TelephonyManager
        val operator = tm.simOperator
        if (operator.equals("46000") || operator.equals("46002") || operator.equals("46007")) {
            return "中国移动"
        } else if (operator.equals("46001")) {
            return "中国联通"
        } else if (operator.equals("46003")) {
            return "中国电信"
        }
        return ""
    }
```

###  Issue_36：DialogActivity
第一步：修改Theme：

```
<style name="dialogActivityTheme" parent="Theme.AppCompat.Light.Dialog">
    <item name="android:background">@color/transparent</item>
    <item name="android:windowBackground">@color/transparent</item>
    <item name="android:windowFrame">@null</item>
    <item name="windowNoTitle">true</item>
    <item name="android:windowIsFloating">false</item>
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:windowContentOverlay">@null</item>
    <item name="android:backgroundDimEnabled">false</item>
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
</style>
```

backgroundDimEnabled 决定是否使用蒙层

第二步：代码中修改 `window.attributes`
实现“包裹”效果，即只显示布局内容，空白区域不显示。
此行代码实现后，具有点击空白区域，自动dismiss的效果

```
val p = window.attributes
p.height = WindowManager.LayoutParams.WRAP_CONTENT
p.width = WindowManager.LayoutParams.WRAP_CONTENT
window.attributes = p
```

###  Issue_37：Px转Dp

```
private fun px2Dip(pxValue: Int): Float {
    val scale = this.resources.displayMetrics.density
    return (pxValue.toFloat() / scale + 0.5f)
}
```

###  Issue_38：Kotlin输出路径下所有文件
```
import java.io.File

fun main(args : Array<String>){
    val inputFile = File("filepath")
    eachFileRecurse(inputFile,::collect)
}

fun eachFileRecurse(inputFile: File, operation: (File) -> Unit) {
    val files = inputFile.listFiles() ?: return
    for (file in files) {
        if (file.isDirectory) {
            eachFileRecurse(file,operation)
        } else {
            collect(file)
        }
    }
}

fun collect(file:File){
    println(file.absolutePath)
}
```

###  Issue_39：java.lang.AssertionError: annotationType(): unrecognized Attribute name MODULE

> 目前使用的Android Studio版本不支持您设置的compileSdkVersion

### Issue_40：Android帧动画
```kotlin
private val animation by lazy {
    val animation = AnimationDrawable()
    animation.isOneShot = false
    for (i in 0..45) {
    val bitmap = BitmapFactory.decodeStream(assets.open("animation_500_kwlffqm500$i.jpg"))
    animation.addFrame(BitmapDrawable(resources, bitmap), 50)
    }
    animation
}
```

```kotlin
iv_frame_animation.setImageDrawable(animation)
animation.start()
```

### Issue_41：AndroidStudio adb命令无效
确保macOS terminal.app 中 adb生效并确保环境变量配置没有问题后
在IDEA terminal中执行：`source ~/.bash_profile`

### Issue_42：android:angle

android:angle是渐变角度，必须为45的整数倍

android:angle＝“0”时，是从左到右，android:angle＝“90”是从下到上来渲染的，android:angle＝“270”是从上到下来渲染的，android:angle＝“180”是从右到左来渲染的，android:angle＝“360”和android:angle＝“0”是一样的

渲染时按照最原始的渲染色板（把控件内部看作一块可以绕中心旋转的板子）围绕控件中心来逆时针旋转相应的度数

### Issue_43：view动画简单使用

```kotlin
private fun startTitleAnimation(vararg v: View) {
        val animation = AnimationUtils.loadAnimation(this, R.anim.anim_guide_sub_item)
        v.forEach {
            it.startAnimation(animation)
        }
    }
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300">
    <translate
        android:fromYDelta="100%"
        android:toYDelta="0" />
    <alpha
        android:fromAlpha="0"
        android:toAlpha="1"/>
</set>
```

### Issue_44：通过Uri形式，读取assets中资源

```kotlin
 const val PATH_ASSETS = "file:///android_asset/"
 
 val uri = PathConstant.PATH_ASSETS + "iphone_A1.mp4"
```

### Issue_45：_view.animate().scaleX(1.1f).setDuration(300).start() 用法

> https://www.cnblogs.com/dasusu/p/8647702.html

### Issue_46：homeBrew更新失败

```
//1.brew update失败后
brew update --verbose
//2.打开报错路径
% cd /opt/homebrew/Library/Taps/homebrew/homebrew-core
% ls -al
//3.执行如下命令
% git fetch --prune origin
% git pull --rebase origin master
//4.再次执行
% brew update
```

### Issue_47：Can't build android. Execution failed for task ':generateJsonModelDebug'

降低CMkae版本至 3.6以下

或者

install **Ninjia**

待观察，安装后显示正常，但是删除build，gradle，idea后，又失效了 = =！

后面发现是有效的，代码仍不关联，不显示颜色，问题是kotlin版本问题。见Issue50

### Issue_48：usesCleartextTraffic

android：usesCleartextTraffic 指示应用程序是否打算使用明文网络流量，例如明文HTTP。高版本默认关闭。

引入某些方法，在使用计算网速，上下行流量时，可能需要手动打开。

### Issue_49：ContextCompat使用/代码中获取颜色

获取资源，而非context.resource.getXXX

```kotlin
ContextCompat.getColor(this, R.color.white)
ContextCompat.getDrawable(this, R.drawable.group)
```

### Issue_50：AndroidStudio一直显示Analyzing

Kotlin版本问题，修改成1.5.10

### Issue_51：Fragment懒加载

建议使用ViewPager2 + TabLayout

ViewPager2默认情况下**不会**预加载出两边Fragment，相当于默认就是懒加载。但是对于Fragment多个情况下，可能会造成Fragment多次销毁和创建。

懒加载实现：

1. 使用ViewPager2 + TabLayout
2. Adapter使用`FragmentStateAdapter`
3. offscreenPageLimit为Fragment size

将加载数据的逻辑，放到Fragment的onResume()中，如此保证了在Fragment可见时才会加载

```kotlin
val adapter = ViewPagerAdapter(supportFragmentManager, lifecycle, fragmentList)

            main_viewpager.adapter = adapter
            main_viewpager.offscreenPageLimit = fragmentList.size

            main_tablayout.addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {
                override fun onTabSelected(tab: TabLayout.Tab?) {
                    val view = tab?.customView?.findViewById<TextView>(R.id.tv_home_tab_item)
                    view?.setTextColor(
                        ContextCompat.getColor(
                            this@MainActivity,
                            R.color.tab_select
                        )
                    )
                    view?.typeface = Typeface.defaultFromStyle(Typeface.BOLD)
                }

                override fun onTabUnselected(tab: TabLayout.Tab?) {
                    val view = tab?.customView?.findViewById<TextView>(R.id.tv_home_tab_item)
                    view?.setTextColor(
                        ContextCompat.getColor(
                            this@MainActivity,
                            R.color.tab_un_select
                        )
                    )
                    view?.typeface = Typeface.defaultFromStyle(Typeface.NORMAL)
                }

                override fun onTabReselected(tab: TabLayout.Tab?) {
                }
            })

            TabLayoutMediator(main_tablayout, main_viewpager) { tab, pos ->
                val tabCustomView = LayoutInflater.from(this@MainActivity)
                    .inflate(R.layout.item_home_tab, main_tablayout, false)

                val tabCustomTextview = tabCustomView.findViewById<TextView>(R.id.tv_home_tab_item)
                tabCustomTextview.text = tabList[pos]

                if (pos == main_viewpager.currentItem) {
                    tabCustomTextview.setTextColor(
                        ContextCompat.getColor(
                            this@MainActivity,
                            R.color.tab_select
                        )
                    )
                } else {
                    tabCustomTextview.setTextColor(
                        ContextCompat.getColor(
                            this@MainActivity,
                            R.color.tab_un_select
                        )
                    )
                }
                tab.customView = tabCustomView
            }.attach()
```

```kotlin
inner class ViewPagerAdapter(
        fm: FragmentManager,
        lifecycle: Lifecycle,
        private val fragmentList: LinkedList<Fragment>
    ) : FragmentStateAdapter(fm, lifecycle) {

        override fun getItemCount(): Int {
            return fragmentList.size
        }

        override fun createFragment(position: Int): Fragment {
            return fragmentList[position]
        }
    }
```

### Issue_52：Android权限请求，一次请求完成

在Splash中直接判断，请求所有需要的权限

```kotlin
private var mPermissionList = arrayListOf<String>()

fun checkSinglePermission(context: Context, permissionName: String): Boolean {
        return if (Build.VERSION.SDK_INT >= 23) {
            ContextCompat.checkSelfPermission(
                context,
                permissionName
            ) == PackageManager.PERMISSION_GRANTED
        } else true
    }
```

```kotlin
private fun initPermission() {
        mPermissionList.clear()
        for (i in permissionArray.indices) {
            if (!PermissionUtil.checkSinglePermission(this, permissionArray[i]))
                mPermissionList.add(permissionArray[i])
        }
        if (mPermissionList.size > 0) {
            ActivityCompat.requestPermissions(
                this,
                permissionArray,
                "requestAllPermission".absHash()
            )
        } else {
            realGotoMain()
        }
    }
```

```kotlin
override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        var hasPermissionDismiss = false
        if (requestCode == "requestAllPermission".absHash()) {
            for (i in grantResults.indices) {
                if (grantResults[i] != PermissionChecker.PERMISSION_GRANTED)
                    hasPermissionDismiss = true
            }
            if (hasPermissionDismiss) {
                val packageURI = Uri.parse("package:${BuildConfig.APPLICATION_ID}")
                val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS, packageURI)
                startActivity(intent)
            } else {
                realGotoMain()
            }
        }
    }
```

### Issue_53：childFragmentManager / supportFragmentManager

childFragmentManager：

Return a private FragmentManager for placing and managing Fragments inside of this Fragment

返回一个私有FragmentManager，用于在此片段中放置和管理片段

supportFragmentManager：

Return the FragmentManager for interacting with fragments associated with this activity

返回FragmentManager以与与此活动关联的片段进行交互

### Issue_54：room数据库问题

1. cannot find implementation for com.useful.toolkits.feature_wallpaper.utils.AppDatabase. AppDatabase_Impl does not exist
2. 使用kapt，错误Execution failed for task ':feature_wallpaper:kaptDebugKotlin'. > A failure

解决方法：

1. ```kotlin
   def room_version = "2.4.0-alpha03"
       implementation "androidx.room:room-runtime:$room_version"
       kapt "androidx.room:room-compiler:$room_version"
       implementation "androidx.room:room-ktx:$room_version"
   ```

2. 只在对应module中添加`room依赖` ,项目中不需要添加

3. 此时错误2，会提示不能使用kapt，要是用 `annotationProcessor` ，但是检查gradle中有使用`kotlin-kapt` 这是为啥呢？添加方式原因：

4. ```kotlin
   //plugins {
   //    id 'com.android.library'
   //    id 'kotlin-android'
   //    id 'kotlin-android-extensions'
   //    id 'kotlin-kapt'
   //}
   ```

5. ```kotlin
   apply plugin: 'com.android.library'
   apply plugin: 'kotlin-android'
   apply plugin: 'kotlin-android-extensions'
   apply plugin: 'kotlin-kapt'
   ```

6. 使用第二种，而非第一种









