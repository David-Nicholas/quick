# quick
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as api from 'aws-cdk-lib/aws-apigateway';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as dynamoDB from 'aws-cdk-lib/aws-dynamodb'
import * as apigateway from "aws-cdk-lib/aws-apigateway";
import path = require('path');
import { table } from 'console';
import * as iam from 'aws-cdk-lib/aws-iam';

export class CdkBuild11Stack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const dynamoDB_table = new dynamoDB.Table(this, "myDynamoDb_table", 
      {partitionKey: { name: 'pk', type: dynamoDB.AttributeType.STRING }, 
      sortKey:{name:"sk", type:dynamoDB.AttributeType.STRING},
      removalPolicy:cdk.RemovalPolicy.DESTROY
      });

    const lambda_function = new lambda.Function(this, "myLambda_function", 
      { handler: 'index.handler', 
      runtime: lambda.Runtime.PYTHON_3_12, 
      code: lambda.Code.fromAsset(path.join(__dirname, "src")),
      environment: {DYNAMODB: dynamoDB_table.tableName}
      });

    dynamoDB_table.grantFullAccess(lambda_function.role!);

    ///////////////////////////////////////////////////////////////////////////

    const restApi_gateway = new api.RestApi(this, "myRestApi_gateway");

    const method = restApi_gateway.root.addResource('scan');
    const methodIntegration = new apigateway.LambdaIntegration(lambda_function);
    method.addMethod('GET', methodIntegration);

    ///////////////////////////////////////////////////////////////////////////

    const role_iam = new iam.Role(this, "myRole_iam", 
    { assumedBy:

    });

    
    
    }
  }

import boto3
import os
import json


def handler():
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table(os.environ['DYNAMODB'])
    try:
        response = table.scan()
        print(json.dumps(response['Items']))
    except Exception as e:
        print(f"Error: {e}")
    
    
