# JWT로 로그인 구현하기

### passport.js란?
사용자를 인증하기 위해 사용하는 Node.js용 미들웨어

### JWT란?
JWT(JSON Web Token)은 클라이언트와 서버 혹은 서비스 간의 통신시 정보를 JSON 객체를 통해 안전하게 전송하고 권한(Authorization)을 위해 사용하는 토큰이다.

### 필요한 모듈 설치
- passport
- passport-local
- passport-jwt
- jsonwebtoken

```sh
npm install passport passport-local passport-jwt jsonwebtoken --save
```

### app.js

```jsx
// app.js
...

const passport = require('passport');
const passportConfig = require('./passport');

...

app.use(passport.initialize());
passportConfig();
```

### config/passport.js
```jsx
// config/passport.js
var db = require('./db');
var conn = db.init();
const bcrypt = require('bcrypt');
const passport = require('passport');
const passportJWT = require('passport-jwt');
const JWTStrategy = passportJWT.Strategy;
const { ExtractJwt } = passportJWT;
const LocalStrategy = require('passport-local').Strategy;

db.connect(conn);

const LocalStrategyOption = {
  usernameField: "user_id",
  passwordField: "password"
};
async function localVerify(user_id, password, done) {
  var user;
  try {
    var sql = 'select * from user where user_id = ?';
    var params = [user_id];
    await conn.query(sql, params, async function (err, rows, fields) {
      if(err) {
        console.log(err);
        return done(null, false);
      }
      if(!rows[0]) return done(null, false);
      user = rows[0];

      console.log(password, user.password);
      const checkPassword = await bcrypt.compare(password, user.password);
      console.log(checkPassword);
      if(!checkPassword) return done(null, false);

      console.log(user);
      return done(null, user);
    })
  } catch (e) {
    return done(e);
  }
}

const jwtStrategyOption = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: 'jwt-secret-key',
};
async function jwtVerift(payload, done) {
  var user;
  try {
    var sql = 'select * from user where user_id = ?';
    var params = [payload.user_id];
    await conn.query(sql, params, function (err, rows, fields) {
      if(!rows[0]) return done(null, false);
      user = rows[0];

      console.log(user);
      return done(null, user);
    });
  } catch (e) {
    return done(e);
  }
}

module.exports = () => {
  passport.use(new LocalStrategy(LocalStrategyOption, localVerify));
  passport.use(new JWTStrategy(jwtStrategyOption, jwtVerift));
```

### controllers/authController.js
```jsx
// controllers/authControllers.js
var express = require('express');
var db = require('../db');
var conn = db.init();
const passport = require('passport');
const jwt = require('jsonwebtoken');

db.connect(conn);

const login =  async (req, res, next) => {
  try {
    passport.authenticate('local', { session: false }, (err, user) => {
      if (err || !user) {
        console.log(err);
        return res.status(400).json({success : false, message : "로그인 실패"});
      }
      req.login(user, { session: false }, (err) => {
        if (err) {
          console.log(err);
          return res.send(err);
        }
        const token = jwt.sign(
          { user_id : user.user_id },
          'jwt-secret-key',
          {expiresIn: "7d"}
        );
        return res.json({ success : true, message : "로그인 성공", token });
      });
    })(req, res);
  } catch (e) {
    console.error(e);
    return next(e);
  }
};

const check =  (req, res) => {
  res.json(req.decoded);
};

module.exports = {
  login,
  check
}
```

### middleware/auth.js
```jsx
// middleware/auth.js
const jwt = require('jsonwebtoken');

exports.verifyToken = (req, res, next) => {
  try {
    const token = req.headers.authorization.split('Bearer ')[1];
    let jwt_secret = 'jwt-secret-key';

    req.decoded = jwt.verify(token, jwt_secret);
    return next();
  } catch (err) {
    if (err.name == 'TokenExpiredError') {
      return res.status(419).json({success: false, message : "token 만료"});
    }
    return res.status(401).json({success: false, message : "token이 유효하지 않습니다."});
  }
}
```

### routes/authRouter.js
```jsx
// routes/authRouter.js
const express = require('express');
const router = express.Router();
const { authController } = require('../controllers');
const { verifyToken } = require('../middleware/auth');

router.post('/login', authController.login);
router.get('/check', verifyToken ,authController.check);

module.exports = router;
```

### 테스트
- POST /auth/login아이디, 비밀번호를 json request로 전송하면 token이 발급된다.
  ![https://media.vlpt.us/images/hyeinisfree/post/ec178910-ad49-4ea8-aedb-24f9160502b6/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.13.18.png](https://media.vlpt.us/images/hyeinisfree/post/ec178910-ad49-4ea8-aedb-24f9160502b6/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.13.18.png)

- GET /auth/check토큰 값을 헤더에 넣어 전송하면 토큰이 유효한지 검사한 후 user 정보를 보내준다.
  ![https://media.vlpt.us/images/hyeinisfree/post/c48c91ae-7552-4239-a333-dd979c7f0a90/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.22.51.png](https://media.vlpt.us/images/hyeinisfree/post/c48c91ae-7552-4239-a333-dd979c7f0a90/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.22.51.png)
  ![https://media.vlpt.us/images/hyeinisfree/post/9d2409ce-7651-49af-9219-53ff8c1dc9b4/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.23.42.png](https://media.vlpt.us/images/hyeinisfree/post/9d2409ce-7651-49af-9219-53ff8c1dc9b4/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-05-24%20%EC%98%A4%ED%9B%84%202.23.42.png)

### 📗 참고
  - [NodeJs - Access / Refresh Token ?](https://velog.io/@neity16/NodeJs-Access-Refresh-Token)
  - [[Node.js] JWT 사용하기](https://surprisecomputer.tistory.com/39)
  - [[Node.js] JWT: Access Token & Refresh Token 인증 구현](https://cotak.tistory.com/102)