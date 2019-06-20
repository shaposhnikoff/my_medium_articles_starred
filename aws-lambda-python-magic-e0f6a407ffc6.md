
# AWS Lambda Python magic

Tips for creating powerful Lambda functions

![](https://cdn-images-1.medium.com/max/3294/1*FHlfn-qCQLrAlG_b-t-zww.png)

By: [*Vincent Sarago](https://www.mapbox.com/about/team/vincent-sarago/)*

Recently, Sean Gillies talked about how [Rasterio is built for cloud-hosted files](https://blog.mapbox.com/build-for-the-cloud-with-rasterio-3254d5d60289). The python library can also be easily packaged for cloud services like AWS Lambda. The Mapbox Satellite team loves Lambda functions. While they can be powerful (e.g. [landsat-tiler](https://github.com/mapbox/landsat-tiler)), they can also be frustrating when creating the package itself. I’ll share some tips and tricks for making complex Lambda functions:

## AWS Lambda Limits

* 250 Mb size (100Mb zip)

* 3008 Mo (was 1536Mo until Dec 2017)

* 5-minute runtime

## Good to know

* Lambda runs on CentOS 7.0 Amazon Linux AMI ([Docker images](https://docs.docker.com/engine/reference/commandline/images/))

* A python Lambda package is equivalent to the content of your site-packages directory

* .zip packages are uncompressed in /var/task at runtime

* LD_LIBRARY_PATH is set to /var/task/lib

Find more info [here](http://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html).

## Creating a package

If you don’t need complex python modules (e.g. C extension modules), creating a Lambda package is a straightforward process.

    pip install my-module -t ./my-package

    cd ./my-package && zip -r9q my-package.zip *

    zip -r9q my-package.zip my_handler.py

But when you need Numpy, SciPy, Rasterio, or Tensorflow, it can get challenging. In the past, you had to create the package on AWS EC2, using Amazon AMI Centos Linux image. This changed when AWS released their Linux image on DockerHub, meaning that you can now do everything locally in seconds.

**Dockerfile**

    # Dockerfile example

    FROM amazonlinux:latest

    RUN yum install -y gcc gcc-c++ freetype-devel yum-utils findutils openssl-devel groupinstall development

    # Install python3.6

    RUN curl https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tar.xz | tar -xJ \

    && cd Python-3.6.1 \

    && ./configure --prefix=/usr/local --enable-shared \

    && make && make install \

    && cd .. && rm -rf Python-3.6.1

    ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

    # Install Python modules to a /tmp/vendored directory that we will zip

    # up for deployment to Lambda.

    # - We force a build of numpy from source to get a lighter distribution (save ~40Mb).

    # - To skip GDAL compilation we use binary rasterio wheels from PyPI.

    # - The `[s3]` option will install rasterio + boto3 to enable AWS S3 files support.

    # - We use --pre option to force the install of rasterio alpha 1.0 version

    RUN pip3 install rasterio[s3] -t /tmp/vendored --no-binary numpy --pre rasterio

    # Echo the estimated size of the package

    RUN du -sh /tmp/vendored

    # Create the zip file

    RUN cd /tmp/vendored && zip -r9q /tmp/package.zip *

    RUN du -sh /tmp/package.zip

**Shell commands**

    # Run dockerfile

    $ docker build -f Dockerfile --tag lambda:latest .

    # Copying the package locally

    $ docker run --name lambda -itd lambda:latest

    $ docker cp lambda:/tmp/package.zip package.zip

    $ docker stop lambda

    $ docker rm lambda

👆 This creates a 138Mb lambda package (35Mb zip). This is not bad, but there are ways to make it lighter 🏋️‍.

The AWS Lambda environment has some pre-installed python modules like boto3 or botocode that we don’t need to ship with our package ([full list of modules](https://gist.github.com/gene1wood/4a052f39490fae00e0c3#file-all_aws_lambda_modules-txt)). So let’s remove them.

    ...

    ...

    # We can remove all tests/ script and other unused files

    RUN find /tmp/vendored -name "*-info" -type d -exec rm -rdf {} +

    RUN find /tmp/vendored -name "tests" -type d -exec rm -rdf {} +

    # Here we remove package that will be present in AWS Lambda env

    RUN rm -rdf /tmp/vendored/boto3/

    RUN rm -rdf /tmp/vendored/botocore/

    RUN rm -rdf /tmp/vendored/docutils/

    RUN rm -rdf /tmp/vendored/dateutil/

    RUN rm -rdf /tmp/vendored/jmespath/

    RUN rm -rdf /tmp/vendored/s3transfer/

    RUN rm -rdf /tmp/vendored/numpy/doc/

    ...

    ...

👆This package will now weigh only 100Mb (28Mb zip).

This is great, we just saved 28% of space, but what if I tell you we can still make it lighter and even make the code run faster?

## Magic trick

Python is an [interpreted language](https://en.wikipedia.org/wiki/Python_(programming_language)), meaning that we don’t compile our code before sending our instruction to the machine. That said, Python has a concept of compiled files used mainly to speed up processes.

If you’ve ever written a python script, you may have noticed the presence of .pyc files after running it. Those files are “byte-compiled” copy of the script that’s then sent to python’s virtual machine ([source](http://www.network-theory.co.uk/docs/pytut/CompiledPythonfiles.html)).

![](https://cdn-images-1.medium.com/max/2000/0*nws0WKrsZoC4yODH.png)

Python .pyc files can be a couple of octets lighter.

Because our code is never going to change, we don’t need to provide .py files. We can ship the .pyc files directly, making the package lighter and also speeding up the python runtime ([source](https://www.curiousefficiency.org/posts/2011/04/benefits-and-limitations-of-pyc-only.html)).

On python3.6, we need to remove all of the .py files and move them from the __pycache__ directories to their top-level parent.

    ...

    ...

    # Keep byte-code compiled files for faster Lambda startup

    RUN find /tmp/vendored -type f -name '*.pyc' | while read f; do n=$(echo $f | sed 's/__pycache__\///' | sed 's/.cpython-36//'); cp $f $n; done;

    RUN find /tmp/vendored -type d -a -name '__pycache__' -print0 | xargs -0 rm -rf

    RUN find /tmp/vendored -type f -a -name '*.py' -print0 | xargs -0 rm -f

    ...

    ...

Adding this 👆, the package now weighs 93Mb (26Mb zip), which is 33% lighter than the initial package 💪.

We hope these tricks help you build more powerful python lambda functions. Here is the [full Dockerfile and other examples](https://github.com/mapbox/aws-lambda-python-packages). Tweet [@*VincentS](https://twitter.com/_VincentS_)* with any questions.
[**Vincent Sarago**
*Vincent is a geologist working with the satellite team to help us produce beautiful basemaps. Vincent develop new tools…*www.mapbox.com](https://www.mapbox.com/about/team/vincent-sarago/)
