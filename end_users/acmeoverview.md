---
layout: writing_samples
comments: false
title:  "ACME Connector Overview"
description: "<div width=50>This is my writing sample that I wrote for my writing bonk <b>bonk</b></div>"
---


# ACME Connector Overview
Surveyland has a Connector that you can use to integrate your Surveyland system with ACME, a customer relationship management (CRM) software.

 -	Easy nomination of your contacts for surveys via automated event triggers, campaigns, and manual nomination.
 - A full history of all contact feedback on each contact's page
 - Real-time Average Scores, Account Health, and Customer Journey charts on each Account page
 -	Bidirectional closed loop follow-up via cases in ACME or Surveyland.

By integrating Surveyland with ACME, your team can get up-to-date analytics and information about your contacts and have everything they need to drive customer satisfaction, promote customer loyalty, and manage account planning to optimize renewal and account growth.

## How the ACME Connector Integrates with your System

The Surveyland Connector is secure because it is loosely coupled, meaning there's no custom code, so there's no additional application to install. The Connector facilitates data transfer between Surveyland and only the objects in your system that you want to integrate with. Rather than deploying custom code in your application, the ACME Connector uses the best in class Informatica Cloud Platform as a middle layer. Informatica uses the existing ACME Connector to transfer data to and from ACME, and uses the Surveyland Connector to transfer the data to and from the corresponding objects in Surveyland.

[See the Surveyland ACME Setup Guide for more information.](#)

## Trigger Surveys Automatically Based on Events
You can set your survey invitations to send when certain events happen in your ACME application. This is a great feature you can use to ensure you are automatically triggering surveys at a key lifecycle stage or for key events in your customer journey. For example, because onboarding is such a critical lifecycle stage, you might want to start your relationship survey based on the onboarding date. Or you can initiate touchpoint surveys for key interactions in the customer journey.

### How do Automatic Triggers work?
Here's an example of how you can set survey invitations to trigger after a support case closes in ACME.

1.	A customer logs a service ticket that raises a case in your ACME application.
2.	A member of your support staff responds to the customer and resolves the issue.
3.	The support staff then closes the case in your ACME application.
4.	The case closure event initiates a survey invitation from Surveyland for the customer who logged the ticket.
5.	Surveyland sends survey invitation status updates to ACME so you can check the customer’s survey response status and click a link to see the survey response.
