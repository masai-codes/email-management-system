<h1>Email Management System (EMS)</h1>
<p>Through EMS we want to send and track all the emails that are going from masai-one-backend without hitting AWS SES rate limits. There are two types of emails we send,
<ol>
  <li><p>Transactional emails, to update the user with the result of every stage. <b>These are single templated emails.</b></p></li>
  <li><p>Campaign emails, to make users take action if they are inactive in any stage. <b>These are bulk templated emails.</b></p></li>
</ol>
</p>
  
<p>We wanted to send campaing emails from masai-one-backend where we are already sending transactional emails. Tracking campaing emails is crucial. Also, we didn't implement the tracking of transactional emails. So, in this case we had two options. Either use some third party APIs (Sendy or Listmonk) for both transactional and campaingn emails or build EMS.</p>

<p>EMS is built on AWS cloud architecture. Which consist of the following services.</p>
<ul>
  <li>AWS Lambda</li>
  <li>Amazon Simple Queue Service (SQS)</li>
  <li>Amazon Simple Notification Service (SNS)</li>
  <li>Amazon Simple Email Service (SES)</li>
  <li>Amazon API Gateway</li>
  <li>Amazon DynamoDB</li>
</ul>

<h3>The EMS flow</h3>
<p>We have an API Gateway which has send-email and send-bulk-email endpoints. The lambda takes in the request and send messages (request body) to email-trigger and email-wait queues. All campaign emails are sent to email-wait queue first and wait untill email-trigger queue can accommodate a bulk email. Which depends on how many messages are there in email-trigger queue and SES rate limit ( which is 100 mails/second ) </p>

<p>We have a template for every email and created a campaign for that template in email-campaign table. The email-trigger lambda uses the campaign Id to send that particular template email. When it sends an email it stores that messageID which is the response of email sent in email-event-tracker. We get an events like 'send', 'delivered', 'click', etc for every email. The status is updated in the table email-event-tracker. </p>

