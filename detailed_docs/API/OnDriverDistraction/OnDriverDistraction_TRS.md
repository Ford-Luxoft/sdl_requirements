## Functional requirements

1. 
Driver distraction rules (if active) are expected to be configurable on HMI for each platform.  
HMI must inform SDL via OnDriverDistraction whenever the driver distraction mode is activated/deactivated on HMI.  

Important Note: DD rules are not applicable to SDL functionality, but describes platform (head unit) configuration and driver distraction restrictions to display or operate some functionality on HMI.  
Driver distraction rules may be specific to contry/area, so it depends on HMI when to trigger activate/deactivate states.

For current HMI configuration, sub-menu opening and and ScrollableMessage operating are not available duringh DD ON mode. HMI may notify user about DD mode enabling-disabling.

2. 
In case  
HMI notifies SDL about current DriverDistraction mode ON/OFF via OnDriverDistraction notification  

SDL must  

transfer OnDriverDistraction(state) notification to all applications being connected to SDL at the moment on any device

_Information:_
_Whether the app receives the notification in current HMILevel is defined by app's assigned Policies. Often, OnDriverDistraction is disallowed for apps in NONE._

3.  
In case
mobile app registers and gets NONE HMILevel
and SDL receives OnDriverDistraction (`<state>`) fom HMI

SDL must  

transfer the last known (actual) OnDriverDistraction (`<state>`) to this mobile app right after this mobile app changes HMILevel to any other than NONE

4. 
In case 
HMI sends OnDriverDistraction (ON) to SDL 
and mobile app successfully connects on SDL and gets any HMILevel

SDL must  

transfer OnDriverDistraction (ON) to this connected app 

5.  
In case HMI sends invalid notification that SDL should transfer to mobile app  

SDL must 

log the issue and ignore this notification

Information:  
1. Invalid notifications means the notification contains:  
a. params out of bounds or incorrect symbols (/t, /n, `<whitespace>`)  
b. mandatory params are missing  
c. params of wrong type  
d. invalid json  
e. incorrect combination of params

## Non-functional requirements
HMI API

```
  <function name="OnDriverDistraction" messagetype="notification">
    <description>Notification must be sent from HMI to SDL when driver distraction state is changed. Driver distraction rules are defined by the platform.</description>
    <param name="state" type="Common.DriverDistractionState" mandatory="true">
      <description>See DriverDistractionState. </description>
    </param>
    <param name="lockScreenDismissalEnabled" type="Boolean" mandatory="false">
    <description>
      If enabled, the lock screen will be able to be dismissed while connected to SDL, allowing users 
      the ability to interact with the app. Dismissals should include a warning to the user and ensure 
      that they are not the driver.
    </description>
  </param>
  </function>
```

## Diagram
OnDriverDistraction
![OnDriverDistraction](./accessories/OnDriverDistraction_lockScreenDismissalEnabled.png)