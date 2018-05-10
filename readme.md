# web3 钱包案例

* web3 + express
* 前端使用bootstrap制作页面。使用express做接口。
* 后端调用web3 实现账户查询，转账操作



在上一节中我们讲到了了如何使用web3调来用以太坊API，学习了如何创建账户，使用私钥签名交易，查询余额，转账等操作。本节我们将创建一个可视化的界面，来完成创建账户，转账、查询余额等功能。
目录：


## 创建项目
打开命令行工具，新建一个文件夹 `EthWallet_web3`
```
mkdir EthWallet_web3
```
进入到项目目录中，使用NPM初始化
```
cd EthWallet_web3
npm init -y
```
使用参数 `-y`可以直接生产`package.json`文件，避免了总是输入回车下一步的麻烦。这是可以看到项目目录下生成了`package.json`。

## 安装项目依赖
我们使用`express`来作为项目后端，通过它来为前端页面提供调用接口。
```
npm install --save express
```
`--save`参数可以使项目依赖保存在`package.json`文件中。

安装`body-parser`,这个库是用来支持post请求的。
```
npm install --save body-parser
```
最后，最重要的，我们还要安装`web3`
```
npm install --save web3
```
## 获取web3实例对象
在项目根目录中新建 `getWeb3.js`
代码：
```
var Web3 = require('web3');

var web3;

if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
  } else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
  }

module.exports = web3;
```
最后通过`module.exports = web3;`把web3实例对象导出，方便其他文件调用。

## 创建前端页面
#### 前端依赖
前端页面我使用了Bootstrap框架,你可以去https://getbootstrap.com/（英文）或者 http://www.bootcss.com/（中文）下载 Bootstrap。
并且使用了jQuery（https://jquery.com/）来方便DOM操作和ajax。

#### 前端目录结构
新建`public`目录，把Bootstrap和jQuery（如果你使用CDN则不需要）放到`public`目录下,新建`index.html`文件。现在项目目录结构如下：
![image.png](https://upload-images.jianshu.io/upload_images/4834364-149f7139f0b87122.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 注意：这里面有一个 .gitignore的文件，这是git仓库的忽略配置文件，用来在推送远程仓库的时候忽略仓库里的某些文件。
如果你没有使用git的话，可以忽略这个文件，当它不存在好了。

接下来编辑前端页面，代码如下
```
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="/bootstrap/css/bootstrap.min.css">
    <script src="/jquery.min.js"></script>
    <script src="/bootstrap/js/bootstrap.min.js"></script>
    <style>
        .row{
            margin-top: 10px;
        }
    </style>
</head>
<body>
    
    <div class="container" style="width: 600px;">
            <div class="row" style="text-align: center">
                    <h1> 以太坊钱包 </h1>
                </div>
        <div class="row">
            <div class="col-md-12">
                <input type="text" class="form-control" placeholder="请输入账户密码" id="password">
            </div>
        </div>
        <div class="row">
                <div class="col-md-12">                   
                    <input class="btn btn-primary btn-block" type="submit" value="提交" id="addaccount">
                </div>
        </div>
    </div>

    <div class="container" style="width: 600px;margin-top: 10px" id="account-list">    
    </div>

    <div class="container" style="width: 600px;margin-top: 10px">
        <div class="row">
            <div class="col-md-12">
                    <div class="form-group form-group-sm">
                        <label class="col-sm-2 control-label" for="address_from">from</label>
                        <div class="col-sm-10">
                            <input class="form-control" type="text" id="address_from" placeholder="转账方地址">
                        </div>
                    </div>
            </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="address_to">to</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="address_to" placeholder="接收方地址">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="trans_value">金额</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="trans_value" placeholder="转账金额">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="trans_password">密码</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="trans_password" placeholder="密码">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">                   
                    <input class="btn btn-primary btn-block" type="submit" value="转账" id="trans_btn">
                </div>
        </div>
    </div>
</body>
</html>
```
## 创建后端
在项目根目录下新建`index.js`,使用express创建一个服务器
```
var express = require("express");  
var app = express();
app.listen(8081,function(){
    console.log('server start...@8081')
});
```
配置静态文件路径
```
var path = require('path');
app.use(express.static(path.join(__dirname, 'public')));
```
这不操作的目的是吧 `public`设置问静态文件路径，这样再浏览器访问public目录下的文件的时候，可以不用添加 public，比如如果想访问index.html,就可以直接在地址栏输入`http://localhost:8081/index.html`,同样的，如果你注意的话，会发现我们在index.html里引用的bootstrap和jQuery也是如此
```
 <link rel="stylesheet" href="/bootstrap/css/bootstrap.min.css">
 <script src="/jquery.min.js"></script>
 <script src="/bootstrap/js/bootstrap.min.js"></script>
```
使用`body-parser`中间件支持post请求
```
var bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```
接下来，我们先添加一个接口，用来访问index.html,在index.js添加如下代码：
```
app.get("/", function(req, res){
    res.sendFile(__dirname + "/public/index.html");
})
```
`__dirname`这是一个全局变量，用来获取当前文件的路劲，这里用它获取到的是`index.js`的路劲，我们通过拼接就得到了`index.html`的文件绝对路径了。
```
__dirname + "/public/index.html"
```
接下来启动服务器，在命令行（注意要在当前项目目录下）执行 
```
node index.js
```
命令行显示
![命令行显示](https://upload-images.jianshu.io/upload_images/4834364-32a92e4adfa9f983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后打开浏览器，输入`http://localhost:8081/`，回车
![浏览器显示](https://upload-images.jianshu.io/upload_images/4834364-2e1f85801f525120.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 获取所有账户
接下来我们添加一个接口用来获取所有账户，在`index.js`中添加如下代码
```
//获取所有用户
app.get('/accounts',function(req,res){
    web3.eth.getAccounts(function(error, result){
        res.send(result)
    })
})
```
然后前端页面也要相应的调用这个接口,修改`index.html`，添加一对`<script></script>`标签，在其中添加：
```
        var accounts = {};

        function gtAccounts(){
            $.get('http://localhost:8081/accounts',function(accs){
                console.log(accs)
                for(var i = 0;i < accs.length;i++){
                    accounts[accs[i]] = 0;
                }
                
                for( k in accounts){
                    getBalance(k)
                }
                
                
            })
        }
        gtAccounts()
  ```
这里使用get的方式请求`http://localhost:8081/accounts`接口，获取所有的账户，展示在页面。

## 注册新账户
后端接口：
```
//注册用户
app.post("/register", function(req, res){

    var password = req.body.password;
    console.log('password:');
    console.log(password);
    web3.eth.personal.newAccount(password)
    .then(function(addr){
        console.log('新增账户:',addr)
        res.send({address:addr,balance:0})
    });

})
```
前端页面调用如下：
```
$("#addaccount").click(function(){

            if($("#password").val() != ""){
                $.post('http://localhost:8081/register/',
                {password:$("#password").val()},
                function(res){
                    console.log(res)
                    accounts[res.address] = res.balance;
                    showAccountList();
                })
            }
            $("#password").val("")
        })
```
## 转账
后端代码：
```
//发送以太币
app.post("/sendcoin", function(req, res){

    var address_from = req.body.address_from;  
    var address_to = req.body.address_to;
    var trans_value = req.body.trans_value;
    var password = req.body.trans_password;
    
    web3.eth.personal.unlockAccount(address_from,password,9999,function(){
        console.log('unlock accounts ok')
        web3.eth.sendTransaction({
            from: address_from,
            to: address_to,
            value: web3.utils.toWei(trans_value,"ether"),
        },function(err,transactionHash){
            if(!err){
                console.log('transactionHash:',transactionHash)
                res.send({msg:"ok",hash:transactionHash});
            }else{
                console.log('-------------error-----------')
                console.log(err)
                console.log('-------------error-----------')
            }
        })
    });
    
}) 
```
前端调用代码：
```
$("#trans_btn").click(function(){

            var address_from = $("#address_from").val();
            var address_to = $("#address_to").val();
            var trans_value = $("#trans_value").val();
            var trans_password = $("#trans_password").val();
            if(address_from != '' && address_to != '' && trans_value != "" && trans_password != ""){
                $.post("http://localhost:8081/sendcoin/",
                {
                    address_from,
                    address_to,
                    trans_value,
                    trans_password
                },function(res){
                    if(res.msg == 'ok'){
                        accounts[address_to] = trans_value;
                        showAccountList();
                        // getBalance(address_from)
                        // getBalance(address_to)
                        
                    }
                })
            }
            // $("#address_from").val("");
            $("#address_to").val("");
            $("#trans_value").val("");
            $("#trans_password").val("");

        })
```
## 完整代码
后端代码
```
var express = require("express");  
var app = express();
var path = require('path');
var bodyParser = require('body-parser');

// var BigNumber = require('bignumber.js');
app.use(express.static(path.join(__dirname, 'public')));

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

var web3 = require("./getWeb3");

app.get("/", function(req, res){
    res.sendFile(__dirname + "/public/index.html");
})

//获取所有用户
app.get('/accounts',function(req,res){
    web3.eth.getAccounts(function(error, result){
        res.send(result)
    })
})

//注册用户
app.post("/register", function(req, res){

    var password = req.body.password;
    console.log('password:');
    console.log(password);
    web3.eth.personal.newAccount(password)
    .then(function(addr){
        console.log('新增账户:',addr)
        res.send({address:addr,balance:0})
    });

})

//查询余额
app.get("/getBalance", function(req, res){

    var address = req.query.address;
    web3.eth.getBalance(address).then(function(balance){
        var ether = web3.utils.fromWei(balance, 'ether');
        res.send(ether);
   })
})



//发送以太币
app.post("/sendcoin", function(req, res){

    var address_from = req.body.address_from;  
    var address_to = req.body.address_to;
    var trans_value = req.body.trans_value;
    var password = req.body.trans_password;
    
    web3.eth.personal.unlockAccount(address_from,password,9999,function(){
        console.log('unlock accounts ok')
        web3.eth.sendTransaction({
            from: address_from,
            to: address_to,
            value: web3.utils.toWei(trans_value,"ether"),
        },function(err,transactionHash){
            if(!err){
                console.log('transactionHash:',transactionHash)
                res.send({msg:"ok",hash:transactionHash});
            }else{
                console.log('-------------error-----------')
                console.log(err)
                console.log('-------------error-----------')
            }
        })
    });
    
}) 



app.listen(8081,function(){
    console.log('server start...@8081')
});
```
前端代码
```
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="/bootstrap/css/bootstrap.min.css">
    <script src="/jquery.min.js"></script>
    <script src="/bootstrap/js/bootstrap.min.js"></script>
    <style>
        .row{
            margin-top: 10px;
        }
    </style>
</head>
<body>
    
    <div class="container" style="width: 600px;">
            <div class="row" style="text-align: center">
                    <h1> 以太坊钱包 </h1>
                </div>
        <div class="row">
            <div class="col-md-12">
                <input type="text" class="form-control" placeholder="请输入账户密码" id="password">
            </div>
        </div>
        <div class="row">
                <div class="col-md-12">                   
                    <input class="btn btn-primary btn-block" type="submit" value="提交" id="addaccount">
                </div>
        </div>
    </div>

    <div class="container" style="width: 600px;margin-top: 10px" id="account-list">    
    </div>

    <div class="container" style="width: 600px;margin-top: 10px">
        <div class="row">
            <div class="col-md-12">
                    <div class="form-group form-group-sm">
                        <label class="col-sm-2 control-label" for="address_from">from</label>
                        <div class="col-sm-10">
                            <input class="form-control" type="text" id="address_from" placeholder="转账方地址">
                        </div>
                    </div>
            </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="address_to">to</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="address_to" placeholder="接收方地址">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="trans_value">金额</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="trans_value" placeholder="转账金额">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">
                        <div class="form-group form-group-sm">
                            <label class="col-sm-2 control-label" for="trans_password">密码</label>
                            <div class="col-sm-10">
                                <input class="form-control" type="text" id="trans_password" placeholder="密码">
                            </div>
                        </div>
                </div>
        </div>
        <div class="row">
                <div class="col-md-12">                   
                    <input class="btn btn-primary btn-block" type="submit" value="转账" id="trans_btn">
                </div>
        </div>
    </div>



    <script>
        
        var accounts = {};

        function gtAccounts(){
            $.get('http://localhost:8081/accounts',function(accs){
                console.log(accs)
                for(var i = 0;i < accs.length;i++){
                    accounts[accs[i]] = 0;
                }
                
                for( k in accounts){
                    getBalance(k)
                }
                
                
            })
        }
        gtAccounts()
        

        $("#addaccount").click(function(){

            if($("#password").val() != ""){
                $.post('http://localhost:8081/register/',
                {password:$("#password").val()},
                function(res){
                    console.log(res)
                    accounts[res.address] = res.balance;
                    showAccountList();
                })
            }
            $("#password").val("")
        })

        $("#trans_btn").click(function(){

            var address_from = $("#address_from").val();
            var address_to = $("#address_to").val();
            var trans_value = $("#trans_value").val();
            var trans_password = $("#trans_password").val();
            if(address_from != '' && address_to != '' && trans_value != "" && trans_password != ""){
                $.post("http://localhost:8081/sendcoin/",
                {
                    address_from,
                    address_to,
                    trans_value,
                    trans_password
                },function(res){
                    if(res.msg == 'ok'){
                        accounts[address_to] = trans_value;
                        showAccountList();
                        // getBalance(address_from)
                        // getBalance(address_to)
                        
                    }
                })
            }
            // $("#address_from").val("");
            $("#address_to").val("");
            $("#trans_value").val("");
            $("#trans_password").val("");

        })

        function getBalance(address){
            console.log(address)
            if(address){
                $.get('http://localhost:8081/getBalance?address='+ address,function(bal){
                    accounts[address] = bal;
                    showAccountList();
                })
            }
        }
        function showAccountList(){
        
            var str = "";          
            for(k in accounts){
                str += `<div class="row">
                            <div class="col-md-9">
                                地址：${k}
                            </div>
                            <div class="col-md-3">
                                金额：${accounts[k]}
                            </div>
                        </div>`
            }
            $("#account-list").html(str)
        }

    </script>
</body>
</html>
```
完整项目案例可以从github上下载：https://github.com/cooleye/web3_wallet



