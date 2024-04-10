上线 -> 向阿里云发送设备状态 从阿里云获取STM32在线状态
## 解析数据
- 获取到设备在线 -》 允许查看数据，控制设备
- 获取到环境\电器状态变化数据 -》 更新响应viewModel
- 控制设备发送数据时开关卡片等点击事件上锁，若获取到ack -》 更新viewModel 开关卡片等点击事件解锁
- 添加布局设备在线否
## 添加功能 1
第二页面添加闹钟设置Button
点击后打开time picker:https://github.com/commandiron/WheelPickerCompose
获取系统时间：
```kotlin
import androidx.compose.foundation.Text
import androidx.compose.foundation.layout.Column
import androidx.compose.runtime.*
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.sp
import kotlinx.coroutines.delay
import java.util.*

@Composable
fun Clock() {
    var currentTime by remember { mutableStateOf(Calendar.getInstance()) }

    LaunchedEffect(Unit) {
        while (true) {
            delay(1000) // 每秒更新一次
            currentTime = Calendar.getInstance()
        }
    }

    val hour = currentTime.get(Calendar.HOUR_OF_DAY)
    val minute = currentTime.get(Calendar.MINUTE)
    val second = currentTime.get(Calendar.SECOND)

    Column {
        Text(text = "Current Time:", fontSize = 20.sp)
        Text(text = "%02d:%02d:%02d".format(hour, minute, second), fontSize = 36.sp)
    }
}

@Preview
@Composable
fun ClockPreview() {
    Clock()
}

```
viewModel
```
timeViewModel:
  val alarmClock=mustateof("2024-01-01 12:00);
  fun update
  val alarmClockState = mustateof(false)
  val event1Name=mustateof("null")
  val event2Time
  val event2State =
  val event2Name=mustateof("null")
  val event2Time
  val event2State =
  val event3Name=mustateof("null")
  val event3Time
  val event3State =
```
```
formDataBean:  time:String,data:String
envBean:time:String,humity,temp,light,ppm:String
class EnvVM:
    val envData = mutablestateOf(envBean)
    val humidtyData24hour = mutablestateOf(Listof("20"))
    val tempList = mutablestateOf(ListOf<String>("20"))
    val light24 = mutablestateOf(ListOf<String>("20"))
    val ppm24 = mutablestateOf(ListOf<String>("20"))
    val time24 = mutablestateOf(ListOf<String>("2022-05-06 12:28"))
    val


fun get24HourBean(properName:String,timeList){
    val currentTimeStr = timeList.last();
    val currentDate = LocalDateTime.parse(currentTimeStr, DateTimeFormatter.ISO_DATE_TIME)
    val threeDaysAgoDate = dateTime.minus(3, ChronoUnit.DAYS)

}
        
    
    
```
```
import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

class MyViewModel : ViewModel() {
    // 从云端接收到的数据
    private val _cloudData = MutableStateFlow("")
    val cloudData: StateFlow<String> = _cloudData

    // 页面数据
    private val _pageData = MutableStateFlow("")
    val pageData: StateFlow<String> = _pageData

    // 更新云端数据
    fun updateCloudData(newData: String) {
        _cloudData.value = newData
    }

    // 更新页面数据
    fun updatePageData(newData: String) {
        _pageData.value = newData
    }

    // 发送页面数据到云端
    fun sendPageDataToCloud() {
        val dataToSend = _pageData.value
        // 发送数据到云端的逻辑
    }
}

```

## 添加功能2：定时事件
第二页面添加闹钟设置Button
弹出对话框，可添加三个定时事件
时间：text button(选择时间)  事件：下拉选择框：选择事件：
