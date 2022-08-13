3, 4 주차 과제
======================

# 3주차: 미세먼지 데이터 웹 페이지로 조회하기
## 3-1. 소스코드(+코드 설명)
```javascript
var express = require('express') // express 모듈 가져오기
var app = express() // express 객체? 생성
const mqtt = require('mqtt'); // mqtt 모듈 가져오기
var fs = require('fs'); //filesystem 모듈 가져오기
const mongoose = require("mongoose"); //mongodb 사용을 위한 모듈 가져오기
const Sensors = require("./sensors"); //sensors 파일에 만들어 놓은 거 불러오기
var qs = require('querystring'); // qureystring 값을 불러오기 위해 사용

app.get('/', function(request, response) { // /로 접근했을 때 실행되는 부분
    fs.readFile(`data`, 'utf8', function(err, data){ // data 파일의 내용을 불러오기
        
         // 각각의 데이터를 뽑기위한 문자열 나누기
        var data1 = data.split(',');
        var tem = data1[0].slice(7);
        var humi = data1[1].slice(7);
        var pm1 = data1[2].slice(6);
        var pm2 = data1[3].slice(7);
        var pm10 = data1[4].slice(7,8);

        // 시간 예쁘게 저장하려고 만듦
        var date = new Date();
        var year = date.getFullYear();
        var month = date.getMonth();
        var today = date.getDate();
        var hours = date.getHours();
        var minutes = date.getMinutes();
        var seconds = date.getSeconds();
        var created_at = new Date(
            Date.UTC(year, month, today, hours, minutes, seconds)
        );

        const sensors = new Sensors({ //sensors 파일에 나와있는 형태로 데이터를 저장하는 것 같습니다.
            tmp: tem,
            hum: humi,
            pm1: pm1,
            pm2: pm2,
            pm10: pm10,
            created_at: created_at,
          });
        
        console.log(sensors);

        //db에 데이터 저장
        try { 
            const saveSensors = sensors.save();
            console.log("insert OK");
          } catch (err) {
            console.log({ message: err });
          }

        // html 부분
        var template = `
        <!DOCTYPE html>
            <html>
            <head>
                <meta charset="UTF-8">
                <meta http-equiv="X-UA-Compatible" content="IE=edge">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Document</title>
            </head>
            <body>
                <p>온도: ${tem}<p/>
                <p>습도: ${humi}<p/>
                <p>pm1: ${pm1}<p/>
                <p>pm2: ${pm2}<p/>
                <p>pm10: ${pm10}<p/>
                <form action="/on_off" method="post">
                    <input type="hidden" name="id" value="1">
                    <input type="submit" value="on">
                </form>
                <form action="/on_off" method="post">
                    <input type="hidden" name="id" value="2">
                    <input type="submit" value="off">
                </form>
        `
        response.send(template); // web에 보여지는 부분
    });
  });

  app.post('/on_off', function(request, response){ //버튼을 눌렀을 때 실행되는 부분
    var body = ''; // 받은 데이터 붙이기 위해 사용
    request.on('data', function(data){ // post 방식으로 받은 데이터 하나씩 받기
        body = body + data; // 붙여서 저장
    });
    request.on('end', function(){ // 데이터 다 받았을 때 실행
        var post = qs.parse(body); // 받은 값 다지기
        var id = post.id; // 버튼 눌렀을 때 보낸 값 받기
        console.log(id); // id 출력
        client.publish("led", id); // id의 내용으로 led 토픽으로 메세지 전송
        response.redirect(`/`); // /로 보내기
    });
  });


const options = { // 브로커 ip 입력과 프로토콜 입력
    host: '192.168.162.154',
    protocol: 'mqtt',
  };
  
const client = mqtt.connect(options); // client 클래스 반환

client.on("connect", () => { // mqtt 연결
    // 연결됐을 때 실행
    console.log("connected"+ client.connected); //  console에 출력
    client.subscribe("sensors"); // 토픽에 구독
  });

client.on('message', (topic, message, packet) => { // 해당 topic에서 message 받기
	console.log("message is "+ message); // 데이터 내용 출력
    fs.writeFile(`data`, message, 'utf8', function(err){ // 데이터를 파일에 저장하기
        console.log('input file'); // 파일에 저장했을 때 출력
      });
  });

app.listen(3000, function() { // 3000 port로 서버 열기
    console.log('Example app listening on port 3000!') // 서버 실행에 성공했을 때 console창에 출력

    //DB 연결
    mongoose.connect(
        process.env.MONGODB_URL,
        { useNewUrlParser: true, useUnifiedTopology: true },
        () => console.log("connected to DB!")
      );
  });
```
****
# 4주차: PyQT5를 이용해 실내 미세먼지 데이터 시각화하기
## 3-1. 소스코드(+코드 설명)
```python
import sys #pyqt 실행에 필요한 모듈
from PyQt5.QtWidgets import QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, QLabel #기본적인 UI 구성요소를 제공하는 모듈
from matplotlib.backends.backend_qt5agg import FigureCanvas as FigureCanvas # 그래프를 그리기 위한 모듈
from matplotlib.backends.backend_qt5agg import NavigationToolbar2QT as NavigationToolbar # 그래프 툴바를 그리기 위한 모듈
from matplotlib.figure import Figure # 그래프를 그리기 위한 모듈
from pymongo import MongoClient # python에서 mongodb를 불러오기 위한 모듈
from PyQt5.QtCore import Qt #pyqt를 사용하기 위한 모듈
from PyQt5.QtGui import QIcon, QPixmap # 사진을 추가하기 위한 모듈

client = MongoClient("몽고db에서 복붙하기") #내 데이터베이스 불러오기

db = client['test'] #test라는 데이터베이스 불러오기



class MyApp(QMainWindow): #MyApp클래스 생성

  def __init__(self):
      super().__init__()

      self.main_widget = QWidget() #main 위젯 생성
      self.setCentralWidget(self.main_widget) # 
      vbox = QVBoxLayout(self.main_widget) # 수직 박스 생성

      self.label2 = QLabel() # 미세먼지 정보를 담기위한 라벨 생성
      self.label2.setAlignment(Qt.AlignCenter) # 가운데 정렬
      vbox.addWidget(self.label2) # 생성한 라벨을 수직 박스에 추가

      hbox = QHBoxLayout(self.main_widget) # 수평 박스 생성
      vbox.addLayout(hbox) # 수평 박스를 수직 박스에 추가

      self.IMAGE = QLabel() # 미세번지 상태 그림을 담기위한 라벨 생성
      hbox.addStretch(1) # 빈 공간 만들기
      hbox.addWidget(self.IMAGE) # 수평 박스에 그림 라벨 추가

      self.label1 = QLabel() # 미세먼지 상태를 담기 위한 라벨 생성
      hbox.addWidget(self.label1) # 수평 박스에 미세먼지 상태 라벨 추가
      hbox.addStretch(1) # 빈 공간 만들기

      dynamic_canvas = FigureCanvas(Figure(figsize=(4, 3))) # (4, 3)짜리의 그래프를 그리기 위한 캔버스 생성
      self.addToolBar(NavigationToolbar(dynamic_canvas, self)) # 그래프 툴바 생성

      
      vbox.addWidget(dynamic_canvas) # 수직 박스에 그래프 추가
      self.dynamic_ax = dynamic_canvas.figure.subplots()  # 좌표축 준비
      self.timer = dynamic_canvas.new_timer(
          100, [(self.update_canvas, (), {})])  # 그래프가 변하는 주기를 위한 타이머 준비 (0.1초, 실행되는 함수)
      self.timer.start()  # 타이머 시작


      self.setWindowTitle('실내 미세먼지 농도 측정하기') #타이틀바에 나타나는 창의 제목 설정
      self.setGeometry(300, 100, 600, 600) #위젯의 위치와 크기 조절
      self.show() #위젯을 스크린에 보여줌

  def update_canvas(self):  # 그래프 변화 함수
    
      self.count=[] #미세먼지 정보에 대한 시간을 담기위한 리스트
      self.pm2=[] #pm2의 값을 담기 위한 리스트
      self.pm10=[] #pm10의 값을 담기 위한 리스트
      self.pm1=[] #pm1의 값을 담기 위한 리스트
      for d, cnt in zip(db['sensors'].find().sort('created_at', -1), range(100, 0, -1)): #test 데이터베이스의 sensors 값을 가져와서 하나씩 뽑기
        self.count.append(d['created_at']) # 시간을 count에 저장
        self.pm2.append(int(d['pm2'])) # pm2의 값을 저장
        self.pm1.append(int(d['pm1'])) # pm1의 값을 저장
        self.pm10.append(int(d['pm10'])) # pm10의 값을 저장

      status = f"현재 미세먼지 농도는 {self.pm10[0]}입니다." #pm10의 값을 포함한 문자열 만들기
      if self.pm10[0] > 150 : #미세먼지 상태를 정하기 위한 부분
        self.dirty = '매우나쁨'
      elif self.pm10[0] > 80 :
        self.dirty = '나쁨'
      elif self.pm10[0] > 30 :
        self.dirty = '보통'
      else :
        self.dirty = '좋음'

      if self.dirty == '매우나쁨' : #미세먼지 상태에 따른 그림을 정하기 위한 부분
        i = "default/imt/very_bad.png"
      elif self.dirty == '나쁨':
        i = "default/imt/bad.png"
      elif self.dirty == '보통':
        i = "default/imt/good.png"
      elif self.dirty == '좋음':
        i = "default/imt/very_good.png"

      self.label2.setText(status) #위에서 생성한 라벨에 문자열 추가
      self.label1.setText(self.dirty) #위에서 생성한 라벨에 문자열 추가
      self.IMAGE.setPixmap(QPixmap(i)) #위에서 생성한 라벨에 그림 추가

      self.dynamic_ax.clear() # 그래프 밀기
      self.dynamic_ax.plot(self.count, self.pm2, color='pink')  # 점 찍기
      self.dynamic_ax.plot(self.count, self.pm1, color='red')  # 점 찍기
      self.dynamic_ax.plot(self.count, self.pm10, color='blue')  # 점 찍기
      self.dynamic_ax.figure.canvas.draw()  # 그리기




if __name__ == '__main__': # pyqt 실행을 위한 코드
  app = QApplication(sys.argv)
  ex = MyApp()
  sys.exit(app.exec_())
```

## 3-2. 영상
https://user-images.githubusercontent.com/105338988/184513521-9e9ebc72-3f50-4795-afac-521986243d1a.mp4
