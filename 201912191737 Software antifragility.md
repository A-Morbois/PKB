# 201912191737 Software antifragility
tags = #SofwareDevelopment #Antifragile #Article

[Source](https://hackernoon.com/site-antifragility-engineering-kr3c3njp)

When releasing a software:
the more control we put on the release, the more fragile is the system.
-> Less release, more feature in one shoot, more interdependance, more stress, more difficult to fix

this is an non ending circle :

Prod error -> more control -> less release -> deployment more fragile -> prod error

Software can be antifragile. 
many deployment improve your system by having small issue which can be corrected rapidly. Small changes that will stress your application but not break it.
>Antifragility is evolution



On the contrary, big changes / deployment is risky and can have an enormous impact if something goes wrong : It can even break the whole system.

catastrophic risks are not predictable by default.
>Don't try and change the world. Make the world more robust towards defects or even exploit the errors.

Instead of trying to predict the risk, ask you the question of how your system will react while facing that risk.
>It is a lot easier to asses how fragile your system is than predicting the next disaster




