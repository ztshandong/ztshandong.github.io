
- npm install --save pg
- npm install pg-transaction   
- PostgreSQL事务，PostgreSQL没有存储过程，全是函数，内部不能启用事务

```javascript
begin([mode], [callback]);
query(); // This is pg.Client.query from node-postgres. There are various possible arguments look at its documentation
 savepoint(savepoint, [callback]);
release(savepoint, [callback]);
rollback([savepoint], [callback]);
commit([callback]);
abort([callback]);
````
# 简单连接
```javascript
var pg = require('pg');
// var MATH = require('math');
var constring = "tcp://username:passwd@192.168.0.2/database";

var client = new pg.Client(constring);
client.connect();

// client.query("create  table beatle(name varchar(10),height integer)");
// client.query("insert into beatle(name,height) values('john',50)");
// client.query("insert into beatle(name,height) values($1,$2)",['brown',68]);
var query = client.query("select * from gettable($1)",['zhangsan']);

query.on('row',function(row){
    console.log(row);
    console.log("Beatle name:%s",row.uname);
    // console.log("beatle height:%d' %d\"",MATH.floor(row.height/12),row.height%12);
});

query.on('end',function(){
    client.end();
});
```
# 启用事务
```javascript
var
     // PostgreSQL modules
     pg = require('pg')
     , Transaction = require('pg-transaction')
 
     // Configuration stuff
     // , connectionString = process.env['PG_CON'] || ''
     ,connectionString = "tcp://username:passwd@192.168.0.2/database";
 ;
 // var die = function(err){
 //     if (err) throw err;
 // };
 var client = new pg.Client(connectionString);
 client.connect();
 
 var tx = new Transaction(client);
  // tx.on('error', die);
 tx.begin();
 tx.query("INSERT INTO myuser(uname, upasswd) values($1, $2)", ['wangwu', '456789']);
 tx.savepoint('savepoint1');
 tx.query("INSERT INTO myuser(uname, upasswd) values($1, $2)", ['zhaoliu', '987654']);
 tx.rollback('savepoint1'); // all statements after savepoint1 are undone (John will not be inserted)
 tx.release('savepoint1'); // can no longer use savepoint1 as a point to rollback to
 tx.commit();
 var query=client.query("select * from gettable($1)",['zhangsan']);
 query.on('row',function(row) {
     console.log(row);
     console.log("Beatle name:%s", row.uname);
 })
 query.on('end',function(){
     client.end();
 });
 
 // client.query("SELECT * from myuser", function(err, result){
 //     if (err) console.log(err);
 //     console.log(result.rows[0].count); // 1
 //     client.end();
 // });

```
