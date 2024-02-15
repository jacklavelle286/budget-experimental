## Budget Restrictor 

# Overview

The budget restrictor solution invovles two paralell solutions which are designed to control budgets in two ways, by being triggered in the following order:

1. Member Account Solution (deployed on an account by account basis):

- A Budget is exceeded within a member account.
- This invokes an SNS Topic.
- The SNS topic is the trigger for a Lambda function which stops all EC2 instances and revokes all access keys in the account.

2. Multi-Account SCP solution (deployed both in the management account and in all member accounts)

There are two cloudformation stacks, one for the management account and one for the member account, and the workflow is as follows:

- Within any member account in the organization, a budget is exceeded
- This invokes an SNS topic
- The SNS topic is the trigger for a Lambda function which retrives context about the account and the Organizational unit in which the budget is exceeded.
- The function then assumes a role within the management account and passes this information to an Amazon Evenbridge EventBus and and EventBridge rule in the management account.
- The rule then triggers a Lambda function in the management account which applies a predefined Serivce Control Policy to the Organizational Unit from which the spend came from - shutting down the ability to carry out any actions within the OU.

If these two solutions are triggered in order, we will achieve the following:

The Member account function will shutdown resources causing excessive spend (EC2 in used so far but we can include any compute / Database service if we want to) and revokes access keys to remove any bad actors from the account who have spun up resources programatically. Then, the Multi-Account SCP solution will apply a restrictive SCP to the originating OU which will prevent any more service being used, access to the accounts and remove the ability to incur any more spend. 
