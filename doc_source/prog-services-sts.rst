.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

########################################
Getting Temporary Credentials with |STS|
########################################

You can use |STSlong| (|STS|_) to get temporary, limited-privilege credentials that can be used to
access AWS services.

There are three steps involved in using |STS|:

1.  Activate a region (optional).

2.  Retrieve temporary security credentials from |STS|.

3.  Use the credentials to access AWS resources.

.. note:: Activating a region is :emphasis:`optional`; by default, temporary security credentials
   are obtained from the global endpoint :emphasis:`sts.amazonaws.com`. However, to reduce latency
   and to enable you to build redundancy into your requests by using additional endpoints if an
   |STS| request to the first endpoint fails, you can activate regions that are geographically
   closer to your services or applications that use the credentials.

.. _optional-activate-and-use-an-sts-region:

(Optional) Activate and use an |STS| region
===========================================

To activate a region for use with |STS|, use the AWS Management Console to select and activate the
region.

.. topic:: To activate additional STS regions

    #. Sign in as an IAM user with permissions to perform IAM administration tasks :code:`"iam:*"`
       for the account for which you want to activate AWS STS in a new region.

    #. Open the IAM console and in the navigation pane click :guilabel:`Account Settings`.

    #. Expand the :guilabel:`STS Regions` list, find the region that you want to use, and then click
       :guilabel:`Activate`.

After this, you can direct calls to the STS endpoint that is associated with that region.

.. note:: For more information about activating STS regions and for a list of the available AWS STS
   endpoints, see `Activating AWS STS in a New Region
   <http://docs.aws.amazon.com/STS/latest/UsingSTS/sts-enableregions.html>`_ in the |STS-ug|.


.. _retrieving-an-sts-token:

Retrieve temporary security credentials from |STS|
==================================================

.. topic:: To retrieve temporary security credentials using the AWS SDK for Java

    #.  Create an :java-api:`AWSSecurityTokenServiceClient` object:

        .. code-block:: java

            AWSSecurityTokenServiceClient sts_client = new AWSSecurityTokenServiceClient();

        When creating the client with no arguments, the default credential provider chain is used to
        retrieve credentials. You can provide a specific credential provider if you want. For more
        information, see Providing AWS Credentials in the AWS SDK for Java.

    #.  :emphasis:`Optional`; requires that you have activated the region) Set the endpoint for the STS
        client:

        .. code-block:: java

            sts_client.setEndpoint("sts-endpoint.amazonaws.com");

        where :emphasis:`sts-endpoint` represents the STS endpoint for your region.

        .. important:: Do not use the :methodname:`setRegion` method to set a regional endpoint |mdash|
            for backwards compatibility, that method continues to use the single global endpoint of
            sts.amazonaws.com.

    #.  Create a :java-api:`GetSessionTokenRequest </model/GetSessionTokenRequest>` object, and
        optionally set the duration in seconds for which the temporary credentials are valid:

        .. code-block:: java

            GetSessionTokenRequest session_token_request = new GetSessionTokenRequest();
            session_token_request.setDurationSeconds(7200); // optional.

        The duration of temporary credentials can range from 900 seconds (15 minutes) to 129600 seconds
        (36 hours) for IAM users. If a duration isn't specified, then 43200 seconds (12 hours) is used
        by default.

        For a root AWS account, the valid range of temporary credentials is from 900 to 3600 seconds (1
        hour), with a default value of 3600 seconds if no duration is specified.

        .. important:: It is :emphasis:`strongly recommended`, from a security standpoint, that you
            :emphasis:`use IAM users` instead of the root account for AWS access. For more information, see
            IAM Best Practices in the |iam-ug|.

    #.  Call :java-api:`getSessionToken
        </AWSSecurityTokenService.html#getSessionToken(com.amazonaws.services.securitytoken.model.GetSessionTokenRequest)>`
        on the STS client to get a session token, using the :classname:`GetSessionTokenRequest` object:

        .. code-block:: java

            GetSessionTokenResult session_token_result =
                sts_client.getSessionToken(session_token_request);

    #.  Get session credentials using the result of the call to :methodname:`getSessionToken`:

        .. code-block:: java

            Credentials session_creds = session_token_result.getCredentials();

The session credentials provide access only for the duration that was specified by the
:classname:`GetSessionTokenRequest` object. Once the credentials expire, you will need to call
:methodname:`getSessionToken` again to obtain a new session token for continued access to AWS.


.. _use-the-token-to-access-aws-resources:

Use the temporary credentials to access AWS resources
=====================================================

Once you have temporary security credentials, you can use them to initialize an AWS service client
to use its resources, using the technique described in :ref:`credentials-explicit`.

For example, to create an S3 client using temporary service credentials:

.. literalinclude:: snippets/sts_basic_session_creds.java
    :language: java
    :lines: 14-

You can now use the :classname:`AmazonS3` object to make Amazon S3 requests.

.. _for-more-information:

For more information
====================

For more information about how to use temporary security credentials to access AWS resources, visit
the following sections in the |STS-ug|:

*   `Creating Temporary Security Credentials
    <http://docs.aws.amazon.com/STS/latest/UsingSTS/CreatingAuthTokens.html>`_

*   `Controlling Permissions for Temporary Security Credentials
    <http://docs.aws.amazon.com/STS/latest/UsingSTS/TokenPermissions.html>`_

*   `Requesting AWS Resources Using Temporary Security Credentials
    <http://docs.aws.amazon.com/STS/latest/UsingSTS/using-temp-creds.html>`_

*   `Activating STS in a New Region
    <http://docs.aws.amazon.com/STS/latest/UsingSTS/sts-enableregions.html>`_

