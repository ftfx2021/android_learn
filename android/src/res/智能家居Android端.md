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
    val environmentDataList = mutablestateOf(envBean)
    val humidityDataList = mutablestateOf(Listof("20"))
    val temperatureDataList = mutablestateOf(ListOf<String>("20"))
    val lightLuxDataList = mutablestateOf(ListOf<String>("20"))
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
envronmentDataList = ..
val humidityDataList = List<String>;
val temperatureDataList = List<String>;
val lightLuxDataList = List<String>;
val ppmDataList = List<String>;
val timeList = List<String>;
for(i in environmentDataList.indices){
    humidityDataList.add(envronmentDataList.get(i).humidity.toString);
    temperatureDataList.add(envronmentDataList.get(i).temperature.toString);
    ppmDataList.add(envronmentDataList.get(i).ppm.toString);
    temperatureDataList.add(envronmentDataList.get(i).temperature.toString);
    lightLuxDataList.add(envronmentDataList.get(i).lightLux.toString);
    timeList.add(envronmentDataList.get(i).time);
}
viewModel.updateEnvironmentDataList(envronmentDataList)
viewModel.updateTemperatureDataList(temperatureDataList)
viewModel.updateHumidityDataList(humidityDataList)
viewModel.updateLightLuxDataList(lightLuxDataList)
viewModel.updatePpmDataList(ppmDataList)
viewModel.updateTimeList(timeList)
}


fun upackageData(){


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
```
data class TimerEventBean(id:Int = 1,time:String="2025-12-25 15:30,eventName:String,state:Int){
}
```
```
class TimerEventVM:ViewModel(){
val timerEvent1 = mutablestateof(TimerEventBean());
fun updateTimerEvent1(newValue:TimerEventBean)
timerEvent1.value = newValue
}
val timerEvent2 = mutablestateof(TimerEventBean());
fun updateTimerEvent2(newValue:TimerEventBean)
timerEvent2.value = newValue
}
val timerEvent3 = mutablestateof(TimerEventBean());
fun updateTimerEvent3(newValue:TimerEventBean)
timerEvent3.value = newValue
}

fun sendTimerEventToCloud(aliyunService:AliyunService,eventId:Int){

val timerEvent = when(eventId){
1 -> timerEvent1.value
2 -> timerEvent2.value
3 -> timerEvent3.value
else -> TimerEventBean()
}
     val gson = Gson()
    val timerEventJson = gson.toJson(mapOf("smart_set" to mapOf("timerEvent" to timerEventBean)))
    sendData(aliyunService,timerEventJson.toString)
}
```

```
fun fun TimerEventCard(title:String,onCheckedChanged:(String,String,Int)->Unit) (){
  val options = listOf(
                        "开客厅灯", "关客厅灯", "开卧室灯", "关卧室灯",
                        "开卫生间灯", "关卫生间灯", "开窗帘", "关窗帘",
                        "开风扇", "关风扇"
                    )

 var showTimePicker by remember { mutableStateOf(false) }
 var selectedOption by remember { mutableStateOf("开开客厅灯") }
 var selectedTime by remember { mutableStateOf("2025-12-25 12:30")}
 var isEnabled by remember { mutableStateOf(true) }
var expand by remember{mutableStateOf(false) }
 Card(
        modifier = Modifier
            .fillMaxWidth()
            .height(150.dp),
        elevation = 8.dp
    ) {
        Column(
            modifier = Modifier
                .padding(16.dp)
                .fillMaxSize(),
            verticalArrangement = Arrangement.SpaceBetween
        ) {
           Text(
                text = title,
                style = MaterialTheme.typography.h6,
                modifier = Modifier.padding(bottom = 8.dp)
            )
         Row(verticalAlignment = Alignment.CenterVertically) {
                Button(onClick = { showTimePicker = true }) {
                    Text(text = "选择时间")
                }
  Spacer(modifier = Modifier.width(16.dp))
//下拉选择框
Spacer(modifier = Modifier.width(16.dp))
 Text(text = "启用定时事件", style = MaterialTheme.typography.body1)
                Spacer(modifier = Modifier.width(8.dp))
                Switch(checked = isEnabled, onCheckedChange = { isEnabled = it,
val eventName=when(selectedOption){

"开客厅灯" -> "livingRoomON;
"关客厅灯" -> "livingRoomOFF"
"开卧室灯" -> "bedRoomON"
"关卧室灯" -> "bedRoomOFF"
"开卫生间灯" -> "toliteON"
"关卫生间灯" -> "toliteOFF"
"开窗帘" -> "curtainON"
"关窗帘" -> "curtainOFF"
else -> "NULL"

}
onCheckedChanged(selectedTime,eventName,if(isEnabled) 1 else 0);

 })
        }
    }
}
```
```
@Composable
fun TimerEventPage(viewModel:TimerEventVM) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        TimerEventCard(timerEvent = TimerEvent("定时事件 1")) {selectedTime,eventName,isEnabled ->
        val timerEvent = TimerEventBean(1,selectedTime,eventName,isEnabled)
           viewModel.updateTimerEvent1(timerEvent)
        }
        Spacer(modifier = Modifier.height(16.dp))
        TimerEventCard(timerEvent = TimerEvent("定时事件 2")) { selectedTime,eventName,isEnabled ->
   val timerEvent = TimerEventBean(2,selectedTime,eventName,isEnabled)
           viewModel.updateTimerEvent2(timerEvent)
            // Handle timer event here
        }
        Spacer(modifier = Modifier.height(16.dp))
        TimerEventCard(timerEvent = TimerEvent("定时事件 3")) { selectedTime,eventName,isEnabled ->
   val timerEvent = TimerEventBean(3,selectedTime,eventName,isEnabled)
           viewModel.updateTimerEvent3(timerEvent)
            // Handle timer event here
        }
    }
}

```
```
@Composable
fun clockPickerDialog(
    showDialog: MutableState<Boolean>,
    onClick: (LocalDateTime) -> Unit
) {
    if (showDialog.value) {
        MaterialDialog(
            onDismiss = { showDialog.value = false },
            buttons = {
                TextButton(
                    onClick = {
                        onClick(selectedDateTime)
                        showDialog.value = false
                    },
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(vertical = 8.dp)
                ) {
                    Text(text = "确定")
                }
            }
        ) {
            Column {
                WheelDateTimePicker(
                    startDateTime = LocalDateTime.of(2025, 10, 20, 5, 30),
                    minDateTime = LocalDateTime.now(),
                    maxDateTime = LocalDateTime.of(20299, 10, 20, 5, 30),
                    timeFormat = TimeFormat.AM_PM,
                    size = DpSize(200.dp, 100.dp),
                    rowCount = 5,
                    textStyle = MaterialTheme.typography.body2,
                    textColor = Color(0xFF000000),
                    selectorProperties = WheelPickerDefaults.selectorProperties(
                        enabled = true,
                        shape = RoundedCornerShape(20.dp),
                        color = Color(0xFFFFFFFF).copy(alpha = 0.2f),
                        border = BorderStroke(2.dp, Color(0xFF000000))
                    )
                ) { snappedDateTime ->
                    selectedDateTime = snappedDateTime
                }
            }
        }
    }
}

```
