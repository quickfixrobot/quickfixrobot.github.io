---
layout: default
---

## [](#header-2)fixrobot is software to write test cases and test FIX applications based on FIX Protocol.
### [](#header-3)

FIXRobot can act as intitiator and acceptor/exchange. fixrobot can send and receive FIX messages and will compare the actual messages with the expected messages. For documentation please check the docs/index.html.  
Installation: pip install fixrobot

Usage: import fixrobot  


## [](#header-2)How to test a FIX based application using fixrobot ?


### [](#header-3)1. Setup your connections
Identify and setup the incoming and outgoing FIX connections from your FIX Application to be tested.
Identify the protocol version you need to test and create the .ini file for each of your FIX connection. 

Eg: fixrobot/env/CLIENT-EXECUTOR-FIX42.ini

>[DEFAULT]  
>ConnectionType=initiator  
>BeginString=FIX.4.2  
>SenderCompID=CLIENT  
>TargetCompID=EXECUTOR  
>HeartBtInt=180  
>SocketConnectPort=9991  
>SocketConnectHost=localhost  
>ReconnectInterval=60  
>...

### [](#header-3)2. Define the outgoing and expected incoming message  
Add the FIX message types and messages you want to test.  

Eg: fixrobot/env/DEFAULTMESSAGES.ini  
>NewOrderSinglePass=NewOrderSingle,\|35=D\|$setField( ClOrdID )\|21=1\|55=IBM\|54=1\|$setField( SendingTime )\|$setField(TransactTime)\|40=2\|38=100\|44=10.50\|59=0\|

>ExecutionReportAckPass=ExecutionReport,\|35=8\|$setField( ClOrdID,  IN, ClOrdID )\|$setField( OrderID )\|$setField( ExecID )\|20=0\|150=0\|39=0\|55=IBM\|38=100\|54=1\|44=10.50\|151=100\|14=0\|6=0.0\|

>ExecutionReportFillPass=ExecutionReport,\|35=8\|$setField( ClOrdID,  IN, ClOrdID )\|$setField( OrderID )\|$setField( ExecID )\|20=0\|150=2\|39=2\|55=IBM\|38=100\|54=1\|44=10.50\|32=100\|31=10.50\|151=0\|14=100\|6=10.50\|

### [](#header-3)3. Create test cases using fixrobot
Write test cases which will start initiator and/or acceptor to send and receive FIX messages.  

Eg: fixrobot/fixrobot/tests/FIX42RobotUnitTest_ShouldPass.py  
```python
from fixrobot import *
import unittest
...
    #Positive test case for NewOrderSingle and Filled where fix message arguments are passed as template names.
    def test_NewOrderSingleFilled_AsNames_ShouldPass(self):
        returnValue = self.exch.clearMessageStore()
        self.assertEqual(returnValue, True)
        time.sleep(1)

        returnValue = self.client.clearMessageStore()
        self.assertEqual(returnValue, True)
        time.sleep(1)
        returnMessageInitiator = self.client.sendMessage("NewOrderSinglePass")
        if returnMessageInitiator:
            self.assertEqual(
                returnMessageInitiator.getHeader().getField(35), "D")
        time.sleep(1)
        returnMessageAcceptor = self.exch.receiveMessage("NewOrderSinglePass")
        if returnMessageAcceptor:
            self.assertEqual(
                returnMessageAcceptor.getHeader().getField(35), "D")
        time.sleep(1)
        returnMessageAcceptor = self.exch.sendMessage("ExecutionReportAckPass")
        if returnMessageAcceptor:
            self.assertEqual(
                returnMessageAcceptor.getHeader().getField(35), "8")
        time.sleep(1)
        returnMessageInitiator = self.client.receiveMessage(
            "ExecutionReportAckPass")
        if returnMessageInitiator:

            self.assertEqual(
                returnMessageInitiator.getHeader().getField(35), "8")
        time.sleep(1)
        returnMessageAcceptor = self.exch.sendMessage(
            "ExecutionReportFillPass")
        if returnMessageAcceptor:
            self.assertEqual(
                returnMessageAcceptor.getHeader().getField(35), "8")
        time.sleep(1)
        returnMessageInitiator = self.client.receiveMessage(
            "ExecutionReportFillPass")
        if returnMessageInitiator:
            self.assertEqual(
                returnMessageInitiator.getHeader().getField(35), "8")
        time.sleep(1)
        returnValue = self.exch.logDumpMessageStore()
        self.assertEqual(returnValue, True)
        time.sleep(1)
...
```

### [](#header-3)4. Execute the test cases
Eg: FIX42RobotUnitTest_ShouldPass.py  
```shell
quickfixrobot:~/Python/fixrobot/fixrobot/tests$ python FIX42RobotUnitTest_ShouldPass.py
/home/quickfixrobot/Python/fixrobot
test_NewOrderSingleFilled_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_NewOrderSingleFilled_AsStrings_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_NewOrderSingleOrderCancelReplaceRequest_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_NewOrderSingleOrderCancelRequest_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_OrderListAckFillReverse_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_OrderListAckFill_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_ResendRequest_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_TestRequestHeartBeatReverse_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_TestRequestHeartBeat_AsNames_ShouldPass (__main__.FIXRobotUnitTest) ... ok
test_getSenderAndTargetMsgSeqNum_ShouldPass (__main__.FIXRobotUnitTest) ... ok

----------------------------------------------------------------------
Ran 10 tests in 100.439s

OK
quickfixrobot:~/Python/FIXRobot/fixrobot/tests$ 

```

### [](#header-3)5. Investigate test case failures
In case if test case fails, investigate in fixrobot logs.  
Eg: fixrobot/fixrobot/fixrobot.log  
```log
...
2018-01-07 11:12:51,167 root         DEBUG    Difference Found True
2018-01-07 11:12:51,167 root         DEBUG    MessageType: Received D == D Expecting
2018-01-07 11:12:51,167 root         DEBUG    FIXTag-35: Received D == D Expecting
2018-01-07 11:12:51,167 root         DEBUG    FIXTag-11: Present
2018-01-07 11:12:51,167 root         DEBUG    FIXTag-21: Received 1 == 1 Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-55: Received IBM != AAPL Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-54: Received 1 == 1 Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-52: Present
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-60: Present
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-40: Received 2 == 2 Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-38: Received 100 != 200 Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-44: Received 10.50 == 10.50 Expecting
2018-01-07 11:12:51,168 root         DEBUG    FIXTag-59: Received 0 == 0 Expecting
2018-01-07 11:12:51,169 root         DEBUG    Received message is not same as expected message
...
```
Medium article: https://medium.com/@quickfixrobot/testing-fix-applications-using-fixrobot-python-module-1e227eb01c78
fixrobot License: GGPLv3+ GNU GPL version 3 or later as mentioned in fixrobot/LICENSE file.  
GGPLv3+ GNU GPL version 3 or later license online: <http://gnu.org/licenses/gpl.html>.  
Download: <https://github.com/quickfixrobot/FIXRobot/>  
Reporting bugs: <https://github.com/quickfixrobot/FIXRobot/issues>  
For enhancements, contributions, collabaration or any questions please contact at <quickfixrobot@gmail.com>  
For any issues or enhacements you can also enter the issues in github  
Author: Written by Anand P. Subramanian.  
Copyright: FIXRobot  Copyright (C) 2016  Anand P. Subramanian.  

