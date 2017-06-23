# SMS-with-SharePoint

<b>#Overview</b><br>
This diagram is written for SharePoint developers to explain how to build a solution that sends a SMS to group of people who exist on SharePoint site as list items. This solution sends SMS depends on two ways, the first one is collecting people from contacts list and make group of contacts and send SMS to them, the second way is to send SMS depends on specific criteria ( want to send SMS to all contacts that belong to sales department and class A and from specific city).
<b>#Highlights :</b><br> 
rabbitMQ, HangFire, TopShelf, EventReciever, webApi, sqlServer, SharePoint, WindowsService, CAML Query, CSOM.
![Alt text](https://github.com/FirasOmar/Gallery/blob/master/sms2.png?raw=true "module summary")
<b>#Technical Details</b><br> 
#Part one: Send SMS to Group  
Send sms general Process: 
<ul>
<li>	The user click new button to schedule sms</li>
<li>User choose sms template (sms text body)</li>
<li>	User choose sms group(the group of people that contains number)</li>
<li>User choose the time that want sms to be send in it.</li>
</ul>
<b>Send SMS Technical process:</b><br>  
<p>When user determine the needed date to schedule sms and click save => event receiver works when item added and send data(id, data and time ) to rabbitMQ queue depends on date and time for schedule sms.</p>
 
<p>A windows service keep listening to rabbitMQ Queues and reed data from the queue if it’s not empty=> then it sends the data to web app thru API.</p>
 
<p>The WebApp read data and categories it depending on the date and time and enqueue to hangfire server (item id, send sms function) [some of enqueued process scheduled and some of it send directly depending of the date Time]</p>
<b>#Part two: Send SMS Depends on Query:</b><br> 
Send SMS Process: 
 When add item in SMS definition list an event receiver triggered and send item id and dateTime fields to specified RabbitMQ Queu
The windows service read from RabbitMQ Queue and send the data to asp.net application thru webApi the app enqueu task to hangfire server (with id + sending fuction(item id )). And schedule it
<b>#Environment Setup</b><br>
Prerequisites: 
<ul>
<li>Sharepoint site collection up and running</li> 
<li>SQL server up and running</li>
<b>RabbitMQ setup</b>:<br> 
<li>http://www.erlang.org/downloads  to install  Erlang Windows Binary File.</li> 
<li>https://www.rabbitmq.com/install-windows.html  to install Installer for Windows systems (from rabbitmq.com)        rabbitmq-server-3.6.6.exe </li><br>
<b>Setting up the management portal :<b><br> 
<li>Open rabbitMQ comand line</li>  
<li>Write the following command rabbitmq-plugins enable rabbitmq_management</li><ul> 
<li>If u face problem installing plugins open the rabbitMQ commad line as admin then navigate to C:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.6\sbin></li>  
<li>Then run the following command  rabbitmq-service remove</li>
<li>	Then run the following command  rabbitmq-service install</li> 
<li>Then run the following command rabbitmq-plugins enable rabbitmq_management</li>  
<li>	http://stackoverflow.com/questions/28258392/rabbitmq-has-nodedown-error </li></ul>
<li>	On the browser write localhost:15672 and then enter username and pass (guest / guest)</li>
<li>To Edit config file  navigate to C:\Users\dcadmin\AppData\Roaming\RabbitMQ  and then uninstall and reinstall</li>

<b>#Appendix</b><br>

<b>Event Receiver #1<b> <br> 
This event receiver trigger when item added on list “scheduleSMS” and send the id and sendingTime to rabbitMQ Queue depends on DateTime Field(if time is current or past send enqueu to Queue1 else enqueu to Queue2).
<b>Event Receiver #2</b><br>
This event receiver trigger when item added on list “SMSQueury” and send the id and sendingTime to rabbitMQ Queue depends on DateTime Field(if time is current or past send enqueu to Queue3 else enqueu to Queue4).

<b>Windows service</b><br> 
Windows service keep running in background and listening to rabbitMq Queues and dequeue from the not empty one and send the data to webApp through post request.<br>
<b>Asp.net web app (API)</b><br> 
webApp receive post request data from windows service and schedule tasks to hangfire server (Enqueu tasks with sending function )to be send when Time comes.
