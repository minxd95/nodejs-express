## Hello World
```javascript
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello World!'))
// get method의 인자 : path, callback
// res.send()는 nodejs에서의 res.writeHead() + res.end()와 같은 역할

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```

## Pretty URL
```javascript
app.get('/board/:boardId/page/:pageId', (req, res) => {

//               content

});

/* 
'/board/free/page/13' 으로 접속했다고 가정.
req.params = {
	boardId: free;
    pageId: 13;
}
*/
```

## Redircet
```javascript
res.redirect('/');
```

### 301, 302 정하여 redirect
You want to redirect the request by setting the status code and providing a location header. 302 indicates a temporary redirect, 301 is a permanent redirect.
```javascript
app.get('/', function (req, res) {
  res.statusCode = 302;
  res.setHeader("Location", "http://www.url.com/page");
  res.end();
});
```
# 미들웨어
## body-parser
npm install body-parser --save
```javascript
const bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: false }));
```
위 코드를 추가하면 콜백함수의 req 인자에 body 속성이 추가된다.
req.body.인자명 으로 post data 값 참조 가능

ex) console.log(req.body.message);

## compression
npm install compression --save
```javascript
const compression = require("compression");
app.use(compression());
```
위 코드 추가 시 전송되는 데이터가 자동으로 압축된다.

## static (정적인 파일의 서비스)
```javascript
app.use(express.static("public"));
// public 폴더 밑의 파일들을 정적으로 서비스 하겠다.
```

## 404 ERROR 의 처리
하단에 아래 코드 추가 (미들웨어는 순차적으로 실행되기 때문)
```javascript
app.use((req, res, next) => {
  res.status(404).send("Not Found");
});
```

# 미들웨어 만들기

**미들웨어는 위에서부터 순차적으로 실행된다**

```javascript
app.use((req, res, next) => {
  fs.readdir("./data", function (error, filelist) {
    req.list = filelist;
    next();
  });
}); // 모든 요청에 대해 적용

app.get("*", (req, res, next) => {
  fs.readdir("./data", function (error, filelist) {
    req.list = filelist;
    next();
  });
}); // 모든 GET 방식의 요청에 대해 적용
```
## next()의 실행
```javascript
app.use((req, res, next) => {
  fs.readdir("./data", function (error, filelist) {
    req.list = filelist;
    next();
  });
},(req, res, [next]) => {

/* next()의 내용 */

});
```

next('route') => 라우터 미들웨어 스택의 나머지 미들웨어 함수들을 건너뛴다.
next(err) => 에러내용을 출력

## 오류처리 미들웨어 함수의 정의
next(err)를 통해 호출된다.
```javascript
app.use(function(err, req, res, next) { // err 인자가 추가됨
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

## 라우터별로 코드 나누기
main.js에 아래 내용 추가
```javascript
const topicRouter = require('./routes/topic');
// '/topic' 경로에 대한 라우터 모듈

app.use('/topic',topicRouter);
```
./routes/topic.js 에 아래 내용 추가
```javascript
/* 필요한 모듈들 임포트 */
const express = require('express');
...........생략...........

/* 라우터 객체 가져오기 */
const router = express.Router();

/* 라우터들 정의 시작 */
router.get('/create /* [/topic]은 생략한다*/ ',...........생략
router.post('/update',............생략
/* 라우터들 정의 끝 */

module.exports = router;

```