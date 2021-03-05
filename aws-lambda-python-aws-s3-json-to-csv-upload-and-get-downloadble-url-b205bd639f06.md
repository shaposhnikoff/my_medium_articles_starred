
# [AWS Lambda+Python+ AWS S3] Json to CSV Upload and Get Downloadble URL

Turn Json to CSV file

1. Upload to S3

1. Return S3 downloadable URL

    **import** **boto3**
    **import** **botocore
    import csv**

    **def** lambda_handler(event, context):

        BUCKET_NAME **=** 'my-bucket' *# replace with your bucket name*
        KEY **=** 'OUTPUT.csv' *# replace with your object key*

        json_data = [{"id":"1","name":"test"},{"id":"2","name":"good"}]

        with open("data.csv", "w") as file:
            csv_file = csv.writer(file)
            csv_file.writerow(['id', 'name'])
            for item in data:
                csv_file.writerow([item.get('id'),item.get('name')])
        
        csv_binary = open('data.csv', 'rb').read()

    **    try**:
            obj = s3.Object(BUCKET_NAME, KEY)
            obj.put(Body=csv_binary)
        **except** botocore**.**exceptions**.**ClientError **as** e:
            **if** e**.**response['Error']['Code'] **==** "404":
                **print**("The object does not exist.")
            **else**:
                **raise**

        s3client = boto3.client('s3')
        **try:**
            download_url = s3client.generate_presigned_url(
                             'get_object',
                              Params={
                                  'Bucket': BUCKET_NAME,
                                  'Key': KEY
                                  },
                              ExpiresIn=3600
            )
            return {"csv_link": download_url}
        **except** Exception as e:
            raise utils_exception.ErrorResponse(400, e, Log)
