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
======================

# 3주차: 미세먼지 데이터 웹 페이지로 조회하기
## 3-1. 소스코드(+코드 설명)
