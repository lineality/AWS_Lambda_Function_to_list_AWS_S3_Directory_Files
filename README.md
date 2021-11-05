# AWS_Lambda_Function_to_list_AWS_S3_Directory_Files

```

"""

2021.11.05
AWS Lambda Function to return list of file names

sample input:
  my_directory = "xyz_folder"

"""

# Import Libraries and Packages for Python
import json
import io
import boto3
import re


# helper function to strip out name
def get_file_name(text_input):

    # 'after this' choice of characters
    after_this = 'key='

    # get item after your choice of characters:
    pattern = f'(?<={after_this}).*$'
    
    # use regex
    target_file_name = re.findall(pattern,text_input) 

    # strip out extra characters with a string-slice
    target_file_name = str(target_file_name[0][1:-2])
    
    return target_file_name



# helper function to make a list of items
def make_file_list(s3_bucket, my_directory):

    # make list for output
    output_list = []

    # iterate through AWS bucket objects
    for obj in s3_bucket.objects.filter(Prefix= my_directory ):

        # make a list of just stripped-down file names
        output_list.append( get_file_name( str(obj) ) )
    
    # remove empty item from the front of the list
    output_list.pop(0)

    # return the list of file names from the AWS S3 directory
    return output_list



# Main AWS Lambda Function
def lambda_handler(event, context):

    ############
    # Get Input
    ############
    my_directory = event["directory_name"]

    # Set Constants
    AWS_REGION = "us-east-1"
    S3_BUCKET_NAME = "api-sample-bucket1"

    ##############
    # Call AWS S3
    ##############
    s3_resource = boto3.resource("s3", region_name=AWS_REGION)
    s3_bucket = s3_resource.Bucket(S3_BUCKET_NAME)

    # run functions to make list of names
    output = make_file_list( s3_bucket, my_directory )

    return {
        'statusCode': 200,
        'body': output
    }
```
