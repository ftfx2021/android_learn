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
    val humidtyDataList = mutablestateOf(Listof("20"))
    val tempDataList = mutablestateOf(ListOf<String>("20"))
    val lightDataList = mutablestateOf(ListOf<String>("20"))
    val ppmDataList = mutablestateOf(ListOf<String>("20"))
    val timeDataList = mutablestateOf(ListOf<String>("2022-05-06 12:28"))
     val specifiedTimeList = mutablestateOf(ListOf<String>("2022-05-06 12:28"))
    val specifiedDataList = mutablestateOf(ListOf<String>("20"))
    val specifiedFormDataList = mutablestateOf(ListOf<FormDataBean>(FormDataBean()))


fun getSpecifiedRangeData(dataList:List<String>,timeList,rangeDays:Int){
    val endTimeStr = timeList.last();
    val endDate = LocalDateTime.parse(endTimeStr, DateTimeFormatter.ISO_DATE_TIME)
    val start = endDate.minus(rangeDays, ChronoUnit.DAYS)

    val targetDataList = ListOf<String>()
    for(i in time.indices){
            val currentTimeStr = timeList.get(i)
            val currentTimeDate = LocalDateTime.parse(currentTimeStr, DateTimeFormatter.ISO_DATE_TIME)
            val comparison = currentTimeDate.compareTo(start)
            if(comparison > 0){
        if(i < dataList.size()){
            targetDataList.add(dataList.get(i))
        }
    }
}
updateSpecifiedDataList(targetDataList);
}
fun getSpecifiedRangeTime(timeList:List<String>,rangeDays:Int,targerTimeFormat:String){
    val endTimeStr = timeList.last();
    val endDate = LocalDateTime.parse(endTimeStr, DateTimeFormatter.ISO_DATE_TIME)
    val start = endDate.minus(rangeDays, ChronoUnit.DAYS)
   
    val targetTimeList = ListOf<String>()
    for(i in time.indices){
            val currentTimeStr = timeList.get(i)
            val currentTimeDate = LocalDateTime.parse(currentTimeStr, DateTimeFormatter.ISO_DATE_TIME)
            val comparison = currentTimeDate.compareTo(start)
              val targetFormattedDateTime = currentTimeDate.format(DateTimeFormatter.ofPattern(targerTimeFormat))
            if(comparison > 0){
            targetTimeList.add(targetFormattedDateTime.get(i))
        }
    }
    updateSpecifiedTimeList(targetTimeList)
}

fun getSpecifiedRangeFormDataBean(dataList:List<String>,timeList,rangeDays:Int,targerTimeFormat:String){
     val endTimeStr = timeList.last();
     val endDate = LocalDateTime.parse(endTimeStr, DateTimeFormatter.ISO_DATE_TIME)
     val targetFormDataBeanList = ListOf<FormDataBean>();
     for(i in time.indices){
            val currentTimeStr = timeList.get(i)
            val currentTimeDate = LocalDateTime.parse(currentTimeStr, DateTimeFormatter.ISO_DATE_TIME)
            val comparison = 1
            if(rangeDays!=-1){
                 val start = endDate.minus(rangeDays, ChronoUnit.DAYS)
                comparison = currentTimeDate.compareTo(start)
            }
            val targetFormattedDateTime = currentTimeDate.format(DateTimeFormatter.ofPattern(targerTimeFormat))
            if(comparison > 0&&i < dataList.size()){
            val formDataBean = FormDataBean(targetFormattedDateTime,dataList.get(i))
            targetFormDataBeanList.add(formDataBean)
        }
    }
 updateSpecifiedFormDataList(targetFormDataBeanList);
}
fun updateEnvData(){



}


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
