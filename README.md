## Deploying large python packages on AWS Lambda using EFS + Serverless REST API using API Gateway 

### In our previous lesson, we made a simple example on Lambda via EFS.
To continue from the previous article:
https://github.com/sedaatalay/Deploying-large-python-packages-on-AWS-Lambda-using-EFS 🔙 

### Now let's see how we can use it with API Gateway by adding more different libraries.

### What is API Gateway?

#### Amazon API Gateway is an AWS service for creating REST, HTTP, and WebSocket APIs at any scale. API Gateway acts as the HTTPS endpoint URL when Amazon API Gateway, along with AWS Lambda, is used to build APIs.

#### Amazon API Gateway endpoints have this format:

👉🏽 https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/

![6250d33edf71977f11b17888_Untitled](https://user-images.githubusercontent.com/91700155/175812025-77d8204f-8325-4f5e-9cbc-4b619a828ddb.png)

#### 1. Lambda Function with example libraries from EFS
🔸 Install the Python libraries in my access folder. We can use these libraries in the example we will do with the help of API Gateway.
```console      
pip3 install --upgrade --target mnt/access/ pandas
pip3 install --upgrade --target mnt/access/ scikit-learn==1.0.2
```
🔸 Let’s go back to the Lambda. Deploy the example code.
```console
#normal imports
import json
import os
import sys
#new library import
sys.path.append('/mnt/access')
import pickle
import numpy as np
#import sklearn
# connecting to the folder with the libraries

def lambda_handler(event, context):
    loaded_model = pickle.load(open("finalized_model.pkl","rb"))
    print('Done1')
    pred = loaded_model.predict([event['X']])
    
    return {
        'statusCode': 200,
        'pred':pred[0]
    }
``` 
<img width="484" alt="Ekran Resmi 2022-06-26 15 49 30" src="https://user-images.githubusercontent.com/91700155/175815006-37b675c0-a2be-4922-8030-e8f35ddbb13c.png">

#### 2. Create API Gateway
Search and select API Gateway. Go to the REST API card and click build.

<img width="898" alt="Ekran Resmi 2022-06-25 18 22 44" src="https://user-images.githubusercontent.com/91700155/175811850-ab66910e-2760-4e79-bae5-d48512da2e8d.png">

Provide all the required information as shown in the image below and click Create API.

<img width="714" alt="Ekran Resmi 2022-06-25 18 23 25" src="https://user-images.githubusercontent.com/91700155/175812216-48066900-49ad-48ce-9bdb-c7d9b1a66b85.png">

In the steps above we created an API. But an API usually has endpoint(s). An endpoint usually specifies a path and the HTTP method it supports. For example GET /get-user. Here, we call the path resource, and the HTTP verb tied to a path a method. Thus, resource + method = REST endpoint.

From the "Action" dropdown select "Create Method". Choose "Get Function". Provide all the other info shown in the image below and click save. Lambda function is the name of the function we create in one of the previous chapter.

<img width="728" alt="Ekran Resmi 2022-06-25 18 23 54" src="https://user-images.githubusercontent.com/91700155/175812256-3a2f8bd4-b040-4216-aea6-d46524db193a.png">

Great! API has been created.

<img width="1254" alt="Ekran Resmi 2022-06-25 18 24 25" src="https://user-images.githubusercontent.com/91700155/175812342-c35b1f26-921a-4f22-be71-39d59599abb8.png">

#### 3. Deploy the API

🔸 In the Actions drop-down list, select Deploy API.
🔸 Select [New Stage] in the Deployment stage drop-down list.
🔸 You can fill in the other parts as you wish.

<img width="479" alt="Ekran Resmi 2022-06-25 18 24 45" src="https://user-images.githubusercontent.com/91700155/175812394-675d520f-0a04-4fdc-b03f-c1ba78ee5755.png">

🔸 Stages has been ready.
Note the invoke URL is your API's base URL.
Append the path to your endpoint to the end of the invoke URL like: https://wrl34unbe0.execute-api.eu-central-1.amazonaws.com/{stage name}/paraphrase. And of course the request method should be GET.

<img width="851" alt="Ekran Resmi 2022-06-25 18 25 44" src="https://user-images.githubusercontent.com/91700155/175812507-3670c5e5-0665-49e4-a969-6ecf237814b2.png">


#### 4. API Gateway test example
Run the request through VS Code to get the result of our sample code that we have uploaded to Lambda.

Let's take the Invoke URL in the API Gateway stage section that was just created and write it in the base section here.

```console
import requests
import json
import pandas as pd
from pprint import pprint
import warnings
warnings.filterwarnings('ignore')

#http://ec2-3-66-191-179.eu-central-1.compute.amazonaws.com:5000/

BASE = 'https://2hkpt557s6.execute-api.eu-central-1.amazonaws.com/lambda-efs-deploy/'
request_list = [0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 1., 1., 1., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
                0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 1.]
response = requests.get(BASE , data = json.dumps({'X': request_list}))
pprint(response.json())
```
<img width="567" alt="Ekran Resmi 2022-06-25 18 32 16" src="https://user-images.githubusercontent.com/91700155/175812587-b7298312-c3fb-4c9d-8dc2-bcfc21ec7c0b.png">

That's it. Your response is here!

<p> </br>
<p>
Thank for timing :)



