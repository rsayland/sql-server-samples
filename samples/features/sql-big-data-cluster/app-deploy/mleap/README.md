# Deploying a Spark model in SQL Server big data cluster using MLeap bundles

### Contents

[About this sample](#about-this-sample)<br/>
[Before you begin](#before-you-begin)<br/>
[Run this sample](#run-this-sample)<br/>
[Sample details](#sample-details)<br/>
[Related links](#related-links)<br/>

<a name=about-this-sample></a>

## About this sample

This sample assumes you are familiar with MLeap. MLeap provides simple interfaces to execute entire ML pipelines. For additional information on MLeap [see the documentation](http://mleap-docs.combust.ml/). MLeap allows you to take a trained Spark model and use it outside of Spark, for example for scoring. After you have created and trained your model in Spark, you can serialize the model as an MLeap bundle for use outside of Spark. This allows model portability and using the model for scoring outside of Spark. This example demonstrates how to serialize a trained model as MLeap bundle and how to deploy it a RESTful web service with a single line of code in SQL Server big data cluster. Also, this example shows how to use an sample input in MLeap frame format to test it.

### Inputs
|Parameter|Description|
|-|-|
|`bundleFileName`|The file name of the MLeap model to run.|

### Outputs
There are no outputs defined as it is dependent on what the model returns.

<a name=before-you-begin></a>

## Before you begin

To run this sample, you need the following prerequisites.

**Software prerequisites:**

1. SQL Server big data cluster CTP 2.3 or later.
2. `mssqlctl`. Refer to [installing mssqlctl](https://docs.microsoft.com/en-us/sql/big-data-cluster/deploy-install-mssqlctl?view=sqlallproducts-allversions) document on setting up the `mssqlctl` and connecting to a SQL Server 2019 big data cluster.

<a name=run-this-sample></a>

## Run this sample

1. Clone or download this sample on your computer.
2. Log in to the SQL Server big data cluster using the command below using the IP address of the `endpoint-service-proxy` in your cluster. If you are not familiar with `mssqltctl` you can refer to the [documentation](https://docs.microsoft.com/en-us/sql/big-data-cluster/big-data-cluster-create-apps?view=sqlallproducts-allversions) and then return to this sample.

   ```bash
   mssqlctl login -e https://<ip-address-of-endpoint-service-proxy>:30777 -u <user-name> -p <password>
   ```
3. This example uses a Machine Learning Model that predicts the price per square foot of an Airbnb property based on various parameters such as the location and type of dwelling. [More details and information on the example are here](http://mleap-docs.combust.ml/mleap-serving/#load-model). The application you will be deploying as part of this sample is a Random Forest Model that was built in Spark and has been serialized as an MLeap bundle. 

   Deploy the app using the `create` command and pass the location of the spec file. In the example below, the spec file is expected to be in the `mleap` folder: 
   ```bash
   mssqlctl app create --spec ./mleap
   ```
4. Check the deployment by running the following command:
   ```bash
   mssqlctl app list -n mleap -v [version]
   ```
   Once the app is listed as `Ready` you can continue to the next step.
5. Now that the app has been deployed you can test if the app works correctly by passing in a sample input that is available in the mleap folder. The deployed app is a RESTful webservice that is [Swagger](swagger.io) compliant. For this sample we will show you how you can test this using the CLI.

   To test the app, run the command below. The input parameter is a `LeapFrame`, a `json` file that describes the parameters and the values provided to the model for predicting the cost per square feet. Note that the input parameter has a special character '@' to indicate that a `json` file is being passed. This command needs to be run within the `mleap` folder. 

   ```bash
   mssqlctl app run --name mleap --version [version] --inputs mleap-frame=@frame.json
   ```

   The result will be a json output that includes the prediction along with additional data, including the predicted price per square feet.

6. You can clean up the sample by running the following commands:
   ```bash
   # delete app
   mssqlctl app delete --name mleap --version [version]
   ```

<a name=sample-details></a>

## Sample details

### Spec file
The spec file serves as the data required for deploying the app and it contains basic information about the app such as the name, version, inputs, outputs, replicas and poolsize you need for this app. As you can see this example uses the `Mleap` runtime and specifies the `bundleFileName` to point to the Spark model serialized as MLeap bundle.

Here is the spec file for this application:

```yaml
name: mleap
version: v1
runtime: Mleap
bundleFileName: model.lr.zip
replicas: 2
poolsize: 2
```

# Next Steps
Please refer to [Train and Create machine learning models with Spark](https://docs.microsoft.com/en-us/sql/big-data-cluster/train-and-create-machinelearning-models-with-spark?view=sqlallproducts-allversions) on how to train your model in Spark within SQL Server big data clusters and export it to MLeap.