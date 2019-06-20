
# Automatically invalidate CloudFront cache for site hosted on S3

Have you hosting static content on s3 and using CloudFront as CDN or https endpoint? This lambda can help you.

Create a python lambda, and use this code

<iframe src="https://medium.com/media/5274a2102b490c63ffb1065e5d25f661" frameborder=0></iframe>

Add this policy to lambda role:

<iframe src="https://medium.com/media/49f09b981d7e043ef19fb728a525c5cd" frameborder=0></iframe>

Now, go to you bucket and configure a event like this, don’t forget to select you lambda function.

![Lambda Event](https://cdn-images-1.medium.com/max/2000/1*QSD2icRWzYNlftkKRDAhzw.png)*Lambda Event*

After create the event add a **distribution_id** tag in you bucket with the id of you cloud front distribution.

Now every time that you site get updated, the lambda will invalidate all file changed in CloudFront cache.
