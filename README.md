# NodeJS Express

```
mysql -u root -p

```
```
create database timeline;
use timeline;
```

```
create user 'timeline'@'localhost' identified by 'password';
grant all on timeline.* to 'timeline'@'localhost';
```

```
create table events (
  id INT AUTO_INCREMENT,
  owner VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  date DATE,
  PRIMARY KEY (id),
  INDEX (owner, date)
);
```

```
npm init
npm install --save-exact express@4.17.1 cors@2.8.5 mysql@2.17.1
```

`vim src/index.js`

```
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const mysql = require('mysql');
const events = require('./events');

const connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'timeline',
  password : 'password',
  database : 'timeline'
});

connection.connect();

const port = process.env.PORT || 8080;

const app = express()
  .use(cors())
  .use(bodyParser.json())
  .use(events(connection));

app.listen(port, () => {
  console.log(`Express server listening on port ${port}`);
});
```

`vim src/events.js`

```
const express = require('express');

function createRouter(db) {
  const router = express.Router();
  const owner = '';

  router.get('/event', function (req, res, next) {
    db.query(
      'SELECT id, name, description, date FROM events WHERE owner=? ORDER BY date LIMIT 10 OFFSET ?',
      [owner, 10*(req.params.page || 0)],
      (error, results) => {
        if (error) {
          console.log(error);
          res.status(500).json({status: 'error'});
        } else {
          res.status(200).json(results);
        }
      }
    );
  });

  router.put('/event/:id', function (req, res, next) {
    db.query(
      'UPDATE events SET name=?, description=?, date=? WHERE id=? AND owner=?',
      [req.body.name, req.body.description, new Date(req.body.date), req.params.id, owner],
      (error) => {
        if (error) {
          res.status(500).json({status: 'error'});
        } else {
          res.status(200).json({status: 'ok'});
        }
      }
    );
  });

  router.delete('/event/:id', function (req, res, next) {
    db.query(
      'DELETE FROM events WHERE id=? AND owner=?',
      [req.params.id, owner],
      (error) => {
        if (error) {
          res.status(500).json({status: 'error'});
        } else {
          res.status(200).json({status: 'ok'});
        }
      }
    );
  });

  return router;
}

module.exports = createRouter;
```

```
node src/index.js
```

```
ALTER USER 'timeline'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

