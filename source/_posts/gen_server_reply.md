title: **gen_server:reply/2**
date: 2014-04-29 21:45:51
category:erlang 
tags:otp 
-------------------------------------

##gen_server:reply/2##

    reply(Client, Reply) -> Result
    Types:
    Client - see below
    Reply = term()
    Result = term()

 This function can be used by a gen_server to explicitly send a reply to a client that called call/2,3 or multi_call/2,3,4, when the reply cannot be defined in the return value of Module:handle_call/3.
 Client must be the From argument provided to the callback function. Reply is an arbitrary term, which will be given back to the client as the return value of call/2,3 or multi_call/2,3,4.
 The return value Result is not further defined, and should always be ignored.

![](http://zhongwencool.qiniudn.com/reply1.png)
Result :

![](http://zhongwencool.qiniudn.com/reply2.png)

if exchange the timer:sleep(3000) and timer:sleep(1000)  the Result will be "gen_server:reply"

so the from process only receive the fast reply ,but the gen_server will execute both!

Why?
gen_server:reply/2 design as :

![](http://zhongwencool.qiniudn.com/reply3.png)