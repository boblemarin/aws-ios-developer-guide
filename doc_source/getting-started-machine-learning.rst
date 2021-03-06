.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

Amazon Machine Learning
=======================

Amazon Machine Learning (ML) is a service that makes it easy for developers of all skill levels to use machine learning technology.
The SDK for iOS provides a simple, high-level client designed to help you interface with Amazon Machine Learning service. The client enables you to call Amazon ML's real-time API to retrieve predictions from your models and enables you to build mobile applications that request and take actions on predictions. The client also enables you to retrieve the real-time prediction endpoint URLs for your ML models.

Integrate Amazon Machine Learning
---------------------------------

To use the Amazon Machine Learning mobile client, you’ll need to integrate the SDK for iOS into your app and import the necessary libraries. To do so, follow these steps:

1. Download the SDK and unzip it as described in `Setup the SDK for iOS <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/setup.html>`_
2. The instructions direct you to import the headers for the services you’ll be using. For Amazon Machine Learning, you need the following import:

``#import <AWSMachineLearning/AWSMachineLearning.h>``

Configure Credentials
^^^^^^^^^^^^^^^^^^^^^

You can use Amazon Cognito to provide temporary AWS credentials to your application. These credentials let the app access your AWS resources. To create a credentials provider, follow the instructions at `Providing AWS Credentials <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/cognito-auth.html#providing-creds-iOS>`_.

To use Amazon Machine Learning in an application, you must set the proper permissions. The following IAM policy allows the user to perform the actions shown in this tutorial on two actions identified by ARN:
::

   {
      "Statement": [{
      "Effect": "Allow",
      "Action": [
          "machinelearning:GetMLModel",
          "machinelearning:Predict"
      ],
      "Resource": "arn:aws:machinelearning:use-east-1:11122233444:mlmodel/example-model-id"
    }]
   }

This policy should be applied to roles assigned to the Amazon Cognito identity pool, but you will need to replace the Resource value with the correct account ID and ML Model ID. You can apply policies at the `IAM console <https://console.aws.amazon.com/iam/home>`_. To learn more about IAM policies, see `Introduction to IAM <http://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html>`_.

Create an Amazon Machine Learning Client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you've imported the necessary libraries and have your credentials object, you can instantiate AWSMachineLearningGetMLModelInput.
::

	AWSMachineLearningGetMLModelInput *getMLModelInput = [AWSMachineLearningGetMLModelInput new];

Making a Predict Request
^^^^^^^^^^^^^^^^^^^^^^^^

Prior to calling Predict, make sure you have not only a completed ML Model ID but also a created real-time endpoint for that ML Model ID. This cannot be done through the mobile SDK; you will have to use the `Machine Learning Console <https://console.aws.amazon.com/machinelearning>`_ or an alternate `SDK <http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/welcome.html>`_. To validate that this ML can be used for real-time Predictions:
::

   // Use a created model that has a created real-time endpoint
   NSString *MLModelId = @"example-model-id";

   // Call GetMLModel to get the realtime endpoint URL
   AWSMachineLearningGetMLModelInput *getMLModelInput = [AWSMachineLearningGetMLModelInput new];
   getMLModelInput.MLModelId = MLModelId;

   [[[MachineLearning getMLModel:getMLModelInput] continueWithSuccessBlock:^id(AWSTask *task) {
      AWSMachineLearningGetMLModelOutput *getMLModelOutput = task.result;

      // Validate that the ML model is completed
      if (getMLModelOutput.status != AWSMachineLearningEntityStatusCompleted) {
          NSLog(@"ML Model is not completed");
          return nil;
       }

       // Validate that the realtime endpoint is ready
       if (getMLModelOutput.endpointInfo.endpointStatus != AWSMachineLearningRealtimeEndpointStatusReady) {
          NSLog(@"Realtime endpoint is not ready");
          return nil;
       }
   }

Once the real-time endpoint is ready, we can begin calling Predict. Note that you must pass the real-time endpoint through the PredictRequest.
::

	// Create a Predict request with your ML Model id and the appropriate Record mapping.
	AWSMachineLearningPredictInput *predictInput = [AWSMachineLearningPredictInput new];
	predictInput.predictEndpoint = getMLModelOutput.endpointInfo.endpointUrl;
	predictInput.MLModelId = MLModelId;
	predictInput.record = @{};

	// Call and return prediction
	return [MachineLearning predict:predictInput];

Additional Resources

- `Developer Guide <http://docs.aws.amazon.com/machine-learning/latest/dg>`_
- `API Reference <http://docs.aws.amazon.com/machine-learning/latest/APIReference>`_
