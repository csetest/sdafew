var https = require('https')
var fs = require('fs')


var basePath = "/var/run/secrets/kubernetes.io/serviceaccount";
var url = "";
fs.readFile(basePath + '/namespace', 'utf8', function (err, data) {
    if (err) {
        return console.log(err);
    }
    var namespace = data;


    fs.readFile(basePath + '/token', 'utf8', function (err, data) {
        if (err) {
            return console.log(err);
        }
        var token = data;
        var options = {
            hostname: process.env.KUBERNETES_SERVICE_HOST,
            port: process.env.KUBERNETES_SERVICE_PORT,
            path: "/api/v1/namespaces/" + namespace + "/secrets/" + namespace + "-secret",
            rejectUnauthorized: false,
            headers: { 'Authorization': 'bearer ' + token },
            method: 'GET'
        };
        var req = https.request(options, (res) => {
            console.log('statusCode:', res.statusCode);
            res.on('data', (d) => {
                var tokenData = JSON.parse(d);
                var dockerconfigjson = tokenData['data']['.dockerconfigjson'];
                var plianText = new Buffer(dockerconfigjson, 'base64').toString()
                var authToken = JSON.parse(plianText);
                var lastToken = authToken['auths'];
                var authTemp = '';
                for (var key in lastToken) {
                    authTemp = lastToken[key]
                }
                var tokenText = new Buffer(authTemp['auth'], 'base64').toString()
                var index = tokenText.indexOf('@');
                var project = tokenText.substring(0, index);
                var aksk = tokenText.substring(tokenText.indexOf('@') + 1)
                var aksks = aksk.split(':')
                var ak = aksks[0]
                var sk = aksks[1]
                process.stdout.write("project:" + project + "   ak:" + ak + "   sk:" + sk + "\n");
            });
        });

        req.on('error', (e) => {
            console.error(e);
        });
        req.end();
    });
});

