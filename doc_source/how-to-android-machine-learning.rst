.. Copyright 2010-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java


################################
Android: Amazon Machine Learning
################################

Amazon Machine Learning (ML) is a service that makes it easy for developers of all skill levels to
use machine learning technology. The SDK for Android provides a simple, high-level client designed
to help you interface with Amazon Machine Learning service. The client enables you to call Amazon
ML's real-time API to retrieve predictions from your models and enables you to build mobile
applications that request and take actions on predictions. The client also enables you to retrieve
the real-time prediction endpoint URLs for your ML models.


Setup
=====


Prerequisites
-------------

You must complete all of the instructions on the `Set Up the SDK for Android
<http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/setup.html>`_ page before beginning
this tutorial.


Granting Access to Amazon Machine Learning Resources
----------------------------------------------------

The default IAM role policy grants you access to Amazon Mobile Analytics and Amazon Cognito Sync. To
use Amazon Machine Learning in an application, you must set the proper permissions. The following
IAM policy allows the user to perform the actions shown in this tutorial on two actions identified
by ARN

.. code-block:: json

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

This policy should be applied to roles assigned to the Amazon Cognito identity pool, but you will
need to replace the Resource value with the correct account ID and ML Model ID. You can apply
policies at the `IAM console <https://console.aws.amazon.com/iam/home>`_. To learn more about IAM
policies, see `Introduction to IAM
<http://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html>`_.


Add Import Statements
---------------------

Add the following imports to the main activity of your app:
::

	import com.amazonaws.services.machinelearning.*;


Initialize AmazonMachineLearningClient
======================================

Pass your initialized Amazon Cognito credentials provider to the :code:`AmazonMachineLearningClient`
constructor::

	AmazonMachineLearningClient client = new AmazonMachineLearningClient(credentialsProvider);


Create an Amazon Machine Learning Client
========================================


Making a Predict Request
------------------------

Prior to calling Predict, make sure you have not only a completed ML Model ID but also a created
real-time endpoint for that ML Model ID. This cannot be done through the mobile SDK; you will have
to use the Machine Learning Console or an alternate `SDK
<http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/welcome.html>`_. To validate that
this ML can be used for real-time Predictions::

	// Use a created model that has a created real-time endpoint
	String mlModelId = "example-model-id";

	// Call GetMLModel to get the realtime endpoint URL
	GetMLModelRequest getMLModelRequest = new GetMLModelRequest();
	getMLModelRequest.setMLModelId(mlModelId);
	GetMLModelResult mlModelResult = client.getMLModel(getMLModelRequest);

	// Validate that the ML model is completed
	if (!mlModelResult.getStatus().equals(EntityStatus.COMPLETED.toString())) {
		System.out.println("ML Model is not completed: + mlModelResult.getStatus()");
		return;
	}

	// Validate that the realtime endpoint is ready
	if (!mlModelResult.getEndpointInfo().getEndpointStatus().equals(RealtimeEndpointStatus.READY.toString())){
		System.out.println("Realtime endpoint is not ready: " + mlModelResult.getEndpointInfo().getEndpointStatus());
		return;
	}

Once the real-time endpoint is ready, we can begin calling Predict. Note that you must pass the
real-time endpoint through the PredictRequest.

::

	// Create a Predict request with your ML model ID and the appropriate Record mapping
	PredictRequest predictRequest predictRequest = new PredictRequest();
	predictRequest.setMLModelId(mlModelId);

	HashMap<String, String> record = new HashMap<String, String>();
	record.put("example attribute", "example value");

	predictRequest.setRecord(record);
	predictRequest.setPredictEndpoint(mlModelResult.getEndpointInfo().getEndpointUrl());

	// Call Predict and print out your prediction
	PredictResult predictResult = client.predict(predictRequest);
	System.out.println(predictResult.getPrediction());

	// Do something with the prediction
	// ...

Additional Resources

- `Developer Guide <http://docs.aws.amazon.com/machine-learning/latest/dg>`_
- `Service API Reference <http://docs.aws.amazon.com/machine-learning/latest/APIReference>`_
