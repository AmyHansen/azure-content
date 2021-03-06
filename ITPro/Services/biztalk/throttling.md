<properties linkid="manage-services-biztalk-services-throttling" urlDisplayName="Throttling" pageTitle="Throttling thresholds in BizTalk Services | Windows Azure" metaKeywords="BizTalk Services, throttling, Windows Azure" description="Learn about throttling thresholds and resulting runtime behaviors for BizTalk Services. Throttling is based on memory usage and number of simultaneous messages." metaCanonical="" services="biztalk-services" documentationCenter="" title="BizTalk Services: Throttling" authors=""  solutions="" writer="mandia" manager="paulettm" editor="susanjo"  />





# BizTalk Services: Throttling

Windows Azure BizTalk Services implements Service throttling based on two conditions: memory usage and the number of simultaneous messages processing. This topic lists the throttling thresholds and describes the Runtime behavior when a throttling condition occurs.


## Throttling Thresholds

The following table lists the throttling source and thresholds:


<table border="1">

<tr bgcolor="FAF9F9">
        <th> </th>
        <td><strong>Description</strong></td>
        <td><strong>Low Threshold</strong></td>
        <td><strong>High Threshold</strong></td>
</tr>
    <tr>
        <td>Memory</td>
        <td>% of total system memory available/PageFileBytes. 
<br/><br/>
Total available PageFileBytes is approximately 2 times the RAM of the system.</td>
        <td>60%</td>
        <td>70%</td>
    </tr>
    <tr>
        <td>Message Processing</td>
        <td>Number of messages processing simultaneously</td>
        <td>40 * number of cores</td>
        <td>100 * number of cores</td>
    </tr>
</table>

When a high threshold is reached, Windows Azure BizTalk Services starts to throttle. Throttling stops when the low threshold is reached. For example, your service is using 65% system memory. In this situation, the service does not throttle. Your service starts using 70% system memory. In this situation, the service throttles and continues to throttle until the service uses 60% (low threshold) system memory.

Windows Azure BizTalk Services tracks the throttling status (normal state vs. throttled state) and the throttling duration.


## Runtime Behavior

When Azure BizTalk Services enters a throttling state, the following occurs:

- Throttling is per role instance. For example:<br/>
RoleInstanceA is throttling. RoleInstanceB is not throttling. In this situation, messages in RoleInstanceB are processed as expected. Messages in RoleInstanceA are discarded and fail with the following error:<br/><br/>
Server is busy. Please try again.<br/><br/>
- Any pull sources do not poll or download a message. For example:<br/>
A pipeline pulls messages from an external FTP source. The role instance doing the pull gets into a throttling state. In this situation, the pipeline stops downloading additional messages until the role instance stops throttling.
- A response is sent to the client so the client can resubmit the message.
- You must wait until the throttling is resolved. Specifically, you must wait until the low threshold is reached.

## Important notes
- Throttling cannot be disabled.
- Throttling thresholds cannot be modified.
- Throttling is implemented system-wide.
- The Azure SQL Database Server also has built-in throttling.

## Additional Windows Azure BizTalk Services topics

-  [Installing the Windows Azure BizTalk Services SDK](http://go.microsoft.com/fwlink/p/?LinkID=241589)<br/>
-  [Tutorials: Windows Azure BizTalk Services](http://go.microsoft.com/fwlink/p/?LinkID=236944)<br/>
-  [How do I Start Using the Windows Azure BizTalk Services SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)<br/>
-  [Windows Azure BizTalk Services](http://go.microsoft.com/fwlink/p/?LinkID=303664)<br/>

## See Also
- [BizTalk Services: Developer, Basic, Standard and Premium Editions Chart](http://go.microsoft.com/fwlink/p/?LinkID=302279)<br/>
- [BizTalk Services: Provisioning Using Windows Azure Management Portal](http://go.microsoft.com/fwlink/p/?LinkID=302280)<br/>
- [BizTalk Services: Provisioning Status Chart](http://go.microsoft.com/fwlink/p/?LinkID=329870)<br/>
- [BizTalk Services: Dashboard, Monitor and Scale tabs](http://go.microsoft.com/fwlink/p/?LinkID=302281)<br/>
- [BizTalk Services: Backup and Restore](http://go.microsoft.com/fwlink/p/?LinkID=329873)<br/>
- [BizTalk Services: Issuer Name and Issuer Key](http://go.microsoft.com/fwlink/p/?LinkID=303941)<br/>