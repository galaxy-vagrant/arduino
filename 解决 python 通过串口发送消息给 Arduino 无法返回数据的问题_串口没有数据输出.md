项目中需要用 python 通过串口给 [Arduino](https://so.csdn.net/so/search?q=Arduino&spm=1001.2101.3001.7020) 发送指令。还是面向百度编程，查询到以下方案:

#### Arduino 代码

```c
#define adapterPort 3
 
void setup() {
  Serial.begin(9600);
  pinMode(adapterPort,OUTPUT);
}
 
void loop() {
  if (Serial.available()>0){
    char serialData=Serial.read();
    
    if(serialData=='H'){
      digitalWrite(adapterPort,HIGH);
      Serial.print(serialData);
      delay(800);
    }
    
    if(serialData=='L'){
      digitalWrite(adapterPort,LOW);
      Serial.print(serialData);
      delay(800);
    }
    
  }
}
```

通过 Arduino 的串口[监视器](https://so.csdn.net/so/search?q=%E7%9B%91%E8%A7%86%E5%99%A8&spm=1001.2101.3001.7020)发送指令，可以成功执行并返回数据

接下来通过 python 编写串口通信代码实现和 Arduino 的数据通信

#### Python 代码

```c
import serial
import time
 
arduino_serial = serial.Serial("COM4", 9600, timeout=0.5)
arduino_high=b'H'
arduino_low=b'L'
 
arduino_serial.write(arduino_high)
print(arduino_serial.readline())
 
# arduino_serial.write(arduino_low)
# print(arduino_serial.readline())
 
arduino_serial.close()
```

奇怪的事情发生了，**在发送完数据之后 python 端就是接收不到数据**

 试了各种方法，甚至都去看串口的底层原理了，还是不行，真不知道网上那些人是怎么通过那千篇一律的代码跑成的。

1000 years later...... 发现仅仅是需要加一行 **time.sleep()** 代码

```c
import serial
import time
 
arduino_serial = serial.Serial("COM4", 9600, timeout=0.5)
arduino_high=b'H'
arduino_low=b'L'
 
time.sleep(3)
arduino_serial.write(arduino_high)
print(arduino_serial.readline())
 
time.sleep(3)
arduino_serial.write(arduino_low)
print(arduino_serial.readline())
 
 
arduino_serial.close()
```

接着就能正常收发数据了，原因很简单，**串口连接的建立是需要时间的**，网上的代码，在串口建立代码后直接发消息，这时候连接都还没建立完成呢，发过去的消息当然没回应。所以**在建立连接之后 sleep(3) 等连接建立完成之后**再发送消息，这样就可以了。