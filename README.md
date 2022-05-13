### AntiAccessibilityService

### Anti AutoJs[https://github.com/hyb1996/AutoJs-Docs]

> 1.disable  AccessibilityManager
 
```kotlin
    fun disableAccessibilityManager() {
        if (hasConfig) {
            try {
                var field = AccessibilityManager::class.java.getDeclaredField("sInstance")
                field.isAccessible = true
                //方案1，mi 8 会崩溃，可使用方案2，3
                val manager = field[null] as AccessibilityManager
                field = AccessibilityManager::class.java.getDeclaredField("mIsEnabled")
                field.isAccessible = true
                field[manager] = false
                //方案2
                field = AccessibilityManager::class.java.getDeclaredField("mRelevantEventTypes")
                field.isAccessible = true
                field[manager] = 0

                //方案3
                field = AccessibilityManager::class.java.getDeclaredField("mIsFinalEnabled")
                field.isAccessible = true
                field[manager] = false
                
                hasConfig = false
            } catch (e: Throwable) {
                Log.e("Anti", "xxxxx", e)
            }
        }
    }
```

> 2.some methods are invoking when AccessibilityService  works .

```kotlin
internal class Wrap(
    private val base: WindowManager
) : WindowManager {
    ...
    override fun addView(view: View, params: ViewGroup.LayoutParams) {
        val v = YTextView(view.context)
        (view as? ViewGroup)?.addView(v, ViewGroup.LayoutParams(30, 30))
        base.addView(view, params)
    }
    ...
}

internal class YTextView : androidx.appcompat.widget.AppCompatTextView {
    constructor(context: Context) : this(context, null)
    constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, -1)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

    override fun findViewsWithText(outViews: ArrayList<View>?, searched: CharSequence?, flags: Int) {
        super.findViewsWithText(outViews, searched, flags)
        // something happen
    }
}

class MainActivity : AppCompatActivity() {
    //replace WindowManager
    override fun getWindowManager(): WindowManager {
        return AccessibilityDetect.windowManager(super.getWindowManager())
    }
}
```
