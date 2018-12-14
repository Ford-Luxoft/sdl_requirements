## Functional requirements

1. 
In case  
SDL receives OnSeekMediaClockTimer notification from HMI  
and this notification is valid and allowed by Policies

SDL must  
transfer OnSeekMediaClockTimer notification from HMI to mobile app

2.   
In case  
SDL receives OnSeekMediaClockTimer notification from HMI  
and this notification is valid and NOT allowed by Policies

SDL must  
log corresponding error internally  
NOT transfer OnSeekMediaClockTimer notification from HMI to mobile app

3.  
In case  
HMI sends invalid notification that SDL should transfer to mobile app 

SDL must 

log the issue and ignore this notification

Information:  
Invalid notifications means the notification contains:  
a. params out of bounds or incorrect symbols (/t, /n, `<whitespace>`)  
b. mandatory params are missing  
c. params of wrong type  
d. invalid json  
e. incorrect combination of params

4. 
In case  
SDL receives SetMediaClockTimer(enableSeek=false) from mobile app
and receives a response of SUCCESS
and HMI sends valid and allowed by Policies OnSeekMediaClockTimer notification

SDL must  
NOT transfer OnSeekMediaClockTimer notification from HMI to mobile app


## Non-functional requirements

1. New OnSeekMediaClockTimer notification must be added to Mobile_API
```
  <function name="OnSeekMediaClockTimer" functionID="OnSeekMediaClockTimerID" messagetype="notification">
     <description> Callback for the seek media clock timer notification. Notifies the application of progress bar seek event on the mediaclock timer. System will automatically update the media clock timer position based on the seek notification location. </description> 
     <param name="seekTime" type="StartTime" mandatory="true">
      <description>See StartTime.</description>
     </param>
  </function>
```

2. New OnSeekMediaClockTimer notification must be added to HMI_API
```
<interface name="UI" version="n.n" date="nnnn-nn-nn">
<function name="OnSeekMediaClockTimer" messagetype="notification">
        <param name="seekTime" type="Common.TimeFormat" mandatory="true"/>
            <description>See TimeFormat.</description>
        </param>
        <param name="appID" type="Integer" mandatory="true">
            <description>
		The ID of application that relates to this media clock status change.
            </description>
        </param>
    </function>
```

3. PolicyTable must support new OnSeekMediaClockTimer notification at "Notifications" section

```
"Notifications": {
          "rpcs": {
"OnSeekMediaClockTimer": {
              "hmi_levels": [
                "<level_1>",
                "<level_2>",
                "<level_3>"
              ],
}
```

## Diagram
OnSeekMediaClockTimer
![OnSeekMediaClockTimer](././OnSeekMediaClockTimer.png)