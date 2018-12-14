## Functional Requirements

1.
	
In case mobile application sends valid SetMediacClockTimer request  
and gets UI.SetMediacClockTimer("SUCCESS") response from HMI  

SDL must  
respond with (resultCode: SUCCESS, success:true) to mobile application

SDL Note1:   
SDL transfers `startTime` together with modes: "COUNTUP", "COUNTDOWN", "PAUSE" to HMI  
SDL ignores `startTime` for modes: "RESUME", and "CLEAR"


SDL Note2: SDL transfers `endTime` together with modes: "COUNTUP", "COUNTDOWN", "PAUSE" to HMI.  
To be used to calculate any visual progress bar (if not provided, this feature is ignored).  
SDL ignores `endTime` for modes: "RESUME", and "CLEAR"  

HMI Information: it is HMI's responsibility to ignore `startTime` for RESUME, CLEAR and display the values correctly for COUNTUP and COUNTDOWN updateMode values.  
It is HMI's responsibility to track whether the requested `updateMode` is already in effect or not.

HMI Note1: If UI.SetMediaClockTimer contains `startTime` and `updateMode` is COUNTUP, HMI starts updating `mediaClock` field by counting up from `startTime`.  
HMI Note2: If UI.SetMediaClockTimer contains `endTime` and `updateMode` is COUNTDOWN, HMI starts updating `mediaClock` field by counting down from `endTime`.  
HMI Note3: If UI.SetMediaClockTimer contains `updateMode` PAUSE, HMI stops updating "mediaClock" field.  
HMI Note4: If UI.SetMediaClockTimer contains `updateMode` RESUME, HMI starts updating `mediaClock` field according to previously active updateMode (COUNTUP/COUNTDOWN).  
HMI Note5: If UI.SetMediaClockTimer contains `updateMode` CLEAR, HMI clears `mediaClock` field.  
HMI Note6: If UI.SetMediaClockTimer contains equal `<startTime>` and `<endTime>`, HMI returns "SUCCESS" result.

2.	
In case  
mob app sends SetMediaClockTimer_request the RPC is NOT included (omitted) in the PolicyTable group(s) assigned to the application

SDL must
return DISALLOWED, success:false to this mobile app

3.	
In case  
Policy Table doesn't contain current application's HMILevel defined in Policy Table "functional_groupings" section for a specified RPC  

SDL must  

return DISALLOWED resultCode and success:"false" to this RPC requested by the application 

Exception: There must be no defined RegisterAppInterface request HMILevel information in "functional_groupings" section. HMILevel check isn't applicable to RegisterAppInterface request and it must be processed with SUCCESS resultCode (with the precondition that RegisterAppInterface is valid and no erroneous resultCodes are triggered).

4.
In case  
PolicyTable has `<appID>`: "null" in the Local PolicyTable for the specified application with `appID`,  

PoliciesManager must  
return DISALLOWED resultCode and success:"false" to any RPC requested by such `<appID>` app.

Exception: PoliciesManager must only process RegisterAppInterface request with SUCCESS resultCode (with the precondition that RegisterAppInterface is valid and no erroneous resultCodes are triggered)

5.
In case  
the request comes to SDL with one of the following:  

wrong json syntax  
wrong type parameters (including parameters of the structures)  
without parameters defined as mandatory  
with out-of-bounds array ranges or out-of-bounds parameters values (including parameters of the structures) of any type

SDL must  
respond with resultCode "INVALID_DATA" and success:"false" to mobile app

6.
In case  
an application sends a request and HMI responds with REJECTED code to at least one of the HMI-portions,  

SDL must  
re-send REJECTED resultCode in a response and success:"false" to mobile application.

Information: HMI is expected to return REJECTED result code in case HMI is currently busy with a higher-priority event or other reasons defined for the mentioned resultCode by HMI.

7.
In case 

an application sends a request and:  
-> either unknown issue happened  
-> or something went wrong 

SDL must  
respond with "GENERIC_ERROR, success:"false" to mobile application

8.  
In case
SDL receives SetMediaClockTimer(enableSeek=false) from mob app
and HMI sends OnSeekMediaClockTimer notification to SDL

SDL must  
NOT transfer OnSeekMediaClockTimer notification from HMI to mobile app


## Non-functional Requirements
Mobile_API
```
    <function name="SetMediaClockTimer" functionID="SetMediaClockTimerID" messagetype="request" since="1.0">
        <description>Sets the initial media clock value and automatic update method.</description>
        
        <param name="startTime" type="StartTime" mandatory="false">
            <description>
                See StartTime.
                startTime must be provided for "COUNTUP" and "COUNTDOWN".
                startTime will be ignored for "RESUME", and "CLEAR"
                startTime can be sent for "PAUSE", in which case it will update the paused startTime
            </description>
        </param>
        
        <param name="endTime" type="StartTime" mandatory="false" since="3.0">
            <description>
                See StartTime.
                endTime can be provided for "COUNTUP" and "COUNTDOWN"; to be used to calculate any visual progress bar (if not provided, this feature is ignored)
                If endTime is greater then startTime for COUNTDOWN or less than startTime for COUNTUP, then the request will return an INVALID_DATA.
                endTime will be ignored for "RESUME", and "CLEAR"
                endTime can be sent for "PAUSE", in which case it will update the paused endTime
            </description>
        </param>
        
        <param name="updateMode" type="UpdateMode" mandatory="true">
            <description>
                Enumeration to control the media clock.
                In case of pause, resume, or clear, the start time value is ignored and shall be left out.  For resume, the time continues with the same value as it was when paused.
            </description>
        </param>

        <param name="audioStreamingIndicator" type="AudioStreamingIndicator" mandatory="false" since="5.0">
            <description>
                Enumeration for the indicator icon on a play/pause button. see AudioStreamingIndicator.
            </description>
        </param>
        <param name="enableSeek" type="Boolean" mandatory="false">
            <description>
       Defines if seek media clock timer functionality will be available. If omitted, the value is set to false. The value is retained until the next SetMediaClockTimer is sent.
            </description>
        </param>
    </function>
    
    <function name="SetMediaClockTimer" functionID="SetMediaClockTimerID" messagetype="response" since="1.0">
        <param name="success" type="Boolean" platform="documentation" mandatory="true">
            <description> true if successful; false, if failed </description>
        </param>
        
        <param name="resultCode" type="Result" platform="documentation" mandatory="true">
            <description>See Result</description>
            <element name="SUCCESS"/>
            <element name="INVALID_DATA"/>
            <element name="OUT_OF_MEMORY"/>
            <element name="TOO_MANY_PENDING_REQUESTS"/>
            <element name="APPLICATION_NOT_REGISTERED"/>
            <element name="GENERIC_ERROR"/>
            <element name="REJECTED"/>
            <element name="IGNORED"/>
        </param>
        
        <param name="info" type="String" maxlength="1000" mandatory="false" platform="documentation">
            <description>Provides additional human readable info regarding the result.</description>
        </param>
    </function>
```

HMI_API

```
 <function name="SetMediaClockTimer" messagetype="request">
    <description>Sets the initial media clock value and automatic update method.</description>
    <param name="startTime" type="StartTime" mandatory="false">
      <description>
        See StartTime.
        startTime must be provided for "COUNTUP" and "COUNTDOWN".
        startTime will be ignored for "RESUME", and "CLEAR"
        startTime can be sent for "PAUSE", in which case it will update the paused startTime
      </description>
    </param>
    <param name="endTime" type="Common.TimeFormat" mandatory="false">
      <description>
        SDL transfers endTime together with modes: "COUNTUP", "COUNTDOWN", "PAUSE" to HMI. To be used to calculate any visual progress bar (if not provided, this feature is ignored).
SDL ignores endTime for modes: "RESUME", and "CLEAR"
      </description>
    </param>
    <param name="updateMode" type="Common.ClockUpdateMode" mandatory="true">
      <description>The update method of the media clock.</description>
      <description>In case of pause, resume, or clear, the start time value is ignored and shall be left out.  For resume, the time continues with the same value as it was when paused.</description>
    </param>
    <param name="appID" type="Integer" mandatory="true">
      <description>ID of application that requested this RPC.</description>
    </param>
 <param name="enableSeek" type="Boolean" mandatory="false">
      <description>
        Defines if seek media clock timer functionality will be available (when DD is off) allowing for touch input on the media clock timer from the user.
        If omitted, the value is set to false. The value is retained until the next SetMediaClockTimer is sent.
      </description>
    </param>
    <param name="enableSeek" type="Boolean" mandatory="false">
     <description>
       Defines if seek media clock timer functionality will be available. If omitted, the value is set to false. The value is retained until the next SetMediaClockTimer is sent.
     </description>
    </param>	
  </function>

 <function name="SetMediaClockTimer" messagetype="response">
 </function>
 ```

## Diagram
SetMediacClockTimer enableSeek=true
![SetMediacClockTimer](./accessories/OnSeekMediaClockTimer.png)