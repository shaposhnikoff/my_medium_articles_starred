
# AWS S3 on the command line

As per AWS Official documentation
> *Amazon Simple Storage Service (Amazon S3) is storage for the Internet. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere on the web*

*To setup AWS CLI*
[**AWS CLI(Command Line Interface)**
*As per AWS Official documentation*medium.com](https://medium.com/@devopslearning/aws-cli-command-line-interface-a48dc3123a25)

* *To create a bucket*

    *# aws s3 mb s3://100daysofdevopsbucket*

    *make_bucket: 100daysofdevopsbucket*

*where mb stands for make bucket*

* *To list out this bucket from the command line*

    *# aws s3 ls*

    *2018-05-20 11:52:38 100daysofdevopsbucket*

* *OR we can check it from UI*

![](https://cdn-images-1.medium.com/max/5284/1*b8rz3GufvYEdfp8ivYti_A.png)

* *If you want this bucket to be created in some specific region*

    *aws s3 mb s3://100daysofdevopsbucket --region us-west-1*

*NOTE: Bucket name must be unique, you can think bucket namespace as global just like the domain name*

* *Now copy some files to this newly created bucket, to upload a file, we are going to use cp command*

    *# aws s3 cp index.html s3://100daysofdevopsbucket*

    *upload: ./index.html to s3://100daysofdevopsbucket/index.html*

* *To verify file is uploaded sucessfully*

    *# aws s3 ls s3://100daysofdevopsbucket*

    *2018-05-20 12:03:33         20 index.html*

* *To Download the file from s3 to local disk*

    *# aws s3 cp s3://100daysofdevopsbucket/index.html .*

    *download: s3://100daysofdevopsbucket/index.html to ./index.html*

* *We can also use features like a recursive copy to the local directory. This recursively copies all objects under mybucket to a specified directory*

    *# aws s3 cp s3://mybucket . --recursive*

* *cp also support include as well as exclude options(This will copy all the files minus jpg file)*

    *# aws s3 cp . s3://100daysofdevopsbucket --recursive --exclude "*.jpg"*

    *upload: ./test1.txt to s3://100daysofdevopsbucket/test1.txt*

    *upload: ./index.html to s3://100daysofdevopsbucket/index.html*

* *We can also specify acl on the command line*

    *# aws s3 cp test3.txt s3://100daysofdevopsbucket --acl public-read-write*

    *upload: ./test3.txt to s3://100daysofdevopsbucket/test3.txt*

* *We can also sync file from local drive to S3 or vice versa*

    *# aws s3 sync . s3://100daysofdevopsbucket*

    *upload: ./test1.jpg to s3://100daysofdevopsbucket/test1.jpg*

    *upload: ./test2.txt to s3://100daysofdevopsbucket/test2.txt*

* *To delete a particular file from s3 bucket*

    *# aws s3 rm s3://100daysofdevopsbucket/test1.txt*

    *delete: s3://100daysofdevopsbucket/test1.txt*

* *To remove a bucket use rb command*

    *# aws s3 rb s3://100daysofdevopsbucket*

    *remove_bucket failed: s3://100daysofdevopsbucket An error occurred (BucketNotEmpty) when calling the DeleteBucket operation: The bucket you tried to delete is not empty*

*NOTE: You cannot delete a bucket which contains the object(unless you are using force command).*

*First remove all the files inside the bucket and then to try to execute this command again*

    *# aws s3 rb s3://100daysofdevopsbucket*

    *remove_bucket: 100daysofdevopsbucket*

*OR*

    *aws s3 rb s3://100daysofdevopsbucket --force*
