var sql = require('mssql');

var config = {
    user: 'username',
    password: 'password',
    server: '127.0.0.1', // You can use 'localhost\\instance' to connect to named instance
    database: 'database',
    port:1433,
    // parseJSON:false, //貌似返回的都是JSON格式
    options: {
        encrypt: false // Use this if you're on Windows Azure
    }
}

sql.connect(config, function(err) {
    // ... error checks

    // Query

    new sql.Request().query('select * from dt_city', function(err, recordset) {
        // ... error checks

        console.dir(recordset);
    });

    // Stored Procedure

    new sql.Request()
         .input('par1', sql.VarChar(50), 'sdf')
        // .output('output_parameter', sql.VarChar(50))
        .execute('usp_Test', function(err, recordsets, returnValue) {
            // ... error checks

            console.dir(recordsets);
        });
});

sql.on('error', function(err) {
    // ... error handler
});
