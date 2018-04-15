# web3 钱包案例

* web3 + express
* 前端使用bootstrap制作页面。使用express做接口。
* 后端调用web3 实现账户查询，转账操作

### 核心代码

#### 注册账户
```
//注册用户
app.get("/register", function(req, res){

    var password = req.query.password;
    console.log('password:');
    console.log(password);
    web3.eth.personal.newAccount(password)
    .then(function(addr){
        console.log('新增账户:',addr)
        res.send({address:addr,balance:0})
    });

})
```

#### 查询余额
```
//查询余额
app.get("/getBalance", function(req, res){

    var address = req.query.address;
    web3.eth.getBalance(address).then(function(balance){
        var ether = web3.utils.fromWei(balance, 'ether');
        res.send(ether);
   })
})
```

#### 转账
```
//发送以太币
app.get("/sendcoin", function(req, res){

    var address_from = req.query.address_from;  
    var address_to = req.query.address_to;
    var trans_value = req.query.trans_value;
    var password = req.query.trans_password;
    
    console.log('address_from:',address_from)
    console.log('address_to:',address_to)
    console.log('trans_value:',trans_value)
    
    var nonce = Math.floor(Math.random()*10000).toString(16);

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