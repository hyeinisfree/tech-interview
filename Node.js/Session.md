# Session으로 로그인 구현하기

### ✔️ session을 어디에 저장할 것 인가?
session은 서버 메모리(MemoryStore)에 저장된다. 서버가 중단되면 세션은 모두 초기화 된다. 따라서 세션을 다른 서버 저장소에 저장할 필요가 있다. 보통 Redis를 사용하여 저장한다. 다른 방법으로는 session-file-store 모듈을 사용하여 FileStore 저장하거나, express-mysql-session 모듈을 사용하여 DB에 저장하는 방법이 있다. 간단한 구현을 위해 FileStore에 저장하여 구현해본다.

### 필요한 모듈 설치
- passport
- passport-local
- express-session
- session-file-store

### app.js
```jsx
// app.js
const passport = require('passport');
const passportConfig = require('./config/passport-session');
const fileStore = require('session-file-store')(session);

app.use(cookieParser('session-secret-key'));
app.use(session({
  secret: 'session-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: false,
  },
  store: new fileStore()
}));

app.use(passport.initialize());
app.use(passport.session());
```

### config/passport-session.js
```jsx
// config/passport-session.js
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;

const bcrypt = require('bcrypt');
var db = require('../db');
var conn = db.init();

db.connect(conn);

passport.serializeUser((user, done) => {
  done(null, user.user_id);
});

passport.deserializeUser(async (user_id, done) => {
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

        console.log(user);
        return done(null, user);
      })
    } catch (e) {
      return done(e);
    }
});

passport.use(new LocalStrategy({
  usernameField: 'user_id',
  passwordField: 'password'
}, async function(user_id, password, done) {
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
))

module.exports = passport
```

### 로그인, 로그아웃, 인증 확인
```jsx
// 로그인
app.post('/login',
  passport.authenticate('local'),
  (req, res) => {
    console.log(req.session);
    res.json(req.user.user_id);
});

// 로그아웃
app.post('/logout', function (req, res, next) {
  req.logout();
  req.session.destroy(function (err) {
    if (err) { return next(err); }
    return res.send({ authenticated: req.isAuthenticated() });
  });
});

// 인증 확인
app.get('/check', (req, res, next) => {
  if(req.isAuthenticated()) return res.json(req.user);
  return res.json({message: 'user 없음'});
});
```