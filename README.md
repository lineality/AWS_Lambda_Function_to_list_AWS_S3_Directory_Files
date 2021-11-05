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
    S3_BUCKET_NAME = "sample-bucket-01"

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

# Filtering
If you need to "filter" results, for example exluding some items from your actionable item list, you will need some extra features. This version separates files that do and do not have an accompanying metadata_ file.

```
"""
workds!

starting with working file-name list code:

Adding: 
- make dynamoDB table
(check for type)
- 

Added: 
- check if file has meta-data file already
- make final list of unmached non-metadata files

"""

"""
## workflow
1. input a folder
2. lambda scans folder for .csv files
3. labmda makes a list of .csv files NOT including metadata at start
4. metadata_datatypes: lambda looks for a metadata_ file and if it does not fine one, the lambda uses pandas to create an auto-made table for AWS datatypes.
5. lambda creates a new dynamoDB table with a name the same as the .csv file
6. input: lambda uses meta-data file to load data into dynamoDB
7. output: list of tables created OR error message
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

def no_metadata_file_list(input_list, my_directory):

    new_list = []

    for i in input_list:

        # file only name
        length = len(my_directory) + 1
        name_only = i[length:]
        print("name_only", name_only)

        # make comparison-test name
        new_name = my_directory + "/" + "metadata_" + name_only 

        # inspection
        print("new_name", new_name)

        # make new measure for offsetting the length of 'metadata_' and the folder
        length_plus = length + 9

        # inspection
        print(i[length:length_plus])

        # check if file is a meta-data file
        if "metadata_" == i[length:length_plus]:
            # do not add file to list if it is a metadata file
            pass

        elif new_name not in input_list:
            # add file to list if it does not already have a metadata file
            new_list.append(i)

    return new_list

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

    #########################
    # Get file names from s3
    #########################
    # run functions to make list of names
    s3_file_names_list = make_file_list( s3_bucket, my_directory )

    ########################################################################
    # Clean file list: remove metadata_files and paired with metadata files
    ########################################################################
    clean_s3_file_names_list = no_metadata_file_list( s3_file_names_list, my_directory )


    return {
        'statusCode': 200,
        'body': json.dumps({'all_files':s3_file_names_list, 'clean':clean_s3_file_names_list})
    }

```
