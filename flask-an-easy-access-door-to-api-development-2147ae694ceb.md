
# Flask: An Easy Access Door to API development

Photo by Chris Ried on Unsplash

The world has gone through a huge transition; from separating the piece of code as functions in procedural languages to the development of libraries; from RPC calls to Web Service specifications in Service Oriented Architecture(SOA) like SOAP and REST.

This has paved a way to Web APIs and microservices, in which the piece of code, providing a solution to a specific problem is exposed (privately or publicly) and the code can be accessed as a black box to the API consumers, who just need to know the so-called interface, which defines the input and output of that particular piece of code.

We log in to many systems using Google, Facebook, Twitter Logins directly, without any need to sign up for each and every specific website. Thanks to these giants for providing publicly available Web APIs for authorization, which has made the developer’s life as well as the user’s life very easy.

Don’t get overwhelmed by so many statements made above. I am sure that till the time you finish reading it, you will be having a good grasp on APIs.

**Now, What Exactly is an API?**

I have been talking of this abbreviation API(Application Program Interface). But what exactly it is? In simple words, it is just a small piece of code exposed in such a way, that any application in any language, be it Java, Python, Node.js, can simply send a request to get some output for some specific input. It just acts as an interface between the underlying application and outside world.

![Client and Target Application(Exposed as API) are in different languages, still can communicate](https://cdn-images-1.medium.com/max/2054/1*69-qky-FJKDD0OLH7Sr5YQ.png)*Client and Target Application(Exposed as API) are in different languages, still can communicate*

As shown above, Clients are in different languages, but they call API as a black box as underlying technology is in Dot Net. Still, they are able to communicate. Response for a particular request remains the same for all the clients. It may be json or XML.

**Now coming to the point, why python and then why flask?**

![Python and Flask combination makes API development very easy](https://cdn-images-1.medium.com/max/3080/1*0Chb847xsx1OwqnRbmGdCA.png)*Python and Flask combination makes API development very easy*

In my Job, working on machine learning related tasks, I had to develop backend APIs for some specific tasks, most of which were related to image uploading, downloading and GRPC calling stuff and it was required to be done in a short timespan.

Coming from JAVA background, when I was exposed to python, I found it to be so easy to be a programming language. No boilerplate codes, rich in libraries, and syntax (which may piss you off in beginning, but you will realize how proper indentation leads to the development of beautiful pieces of code). So, it was an obvious affinity for python. And above all, when it comes to machine learning and deep learning, python is the killer.

Next task was to choose a well-defined framework for python which can be used for APIs development. I had two options: Django and Flask (there are many others but these 2 are mostly used with much support). I did some research and the reason I chose flask was that it is way more pythonic than Django. So, I had the answer for this as well. Now was the time to execute it. And so, with Flask, the backend was ready within few days with all the APIs developed as REST calls

So, much of talk now, let's get your hands dirty in the Flask filled with pythons. I know it sounds scary. but pythons are non-venomous and so this is; trust me.
> Caution: Expect a good amount of code in this post. Reason for this is I didn’t want to just show hello world type of code. Because that is already done in many of websites and you don’t need to see one more post for the same thing.
> Here I wanted you to have a good experience in Flask and python so that you can readily start using the same in your apps , which require common functionalities like sending json response after reading a file and uploading a file. So, put on your seat belts and start coding .

Let’s develop two Web-API ’s, one with **GET** and one with **POST**.

## **GET API:**

    **flask_simplified.py**

    from flask import json
    from flask import Response
    from flask_cors import CORS
    from flask_api import FlaskAPI

    APP = FlaskAPI(__name__)
    CORS(APP)

    @APP.route("/getJsonFromFile/<filename>", methods=['GET'])
    def get_json_response(filename):

        labels_dict = {}
        response_dict = {}
        try:
            with open(filename, 'r') as labels:
                labels_dict = json.load(labels)

            ***response_dict[STATUS] = "true***
            ***response_dict["labels_mapping"] = labels_dict
            js_dump = json.dumps(response_dict)
            resp = Response(js_dump,status=200,
                            mimetype='application/json')***

        except FileNotFoundError as err:
            response_dict = {'error': 'file not found in server'}
            js_dump = json.dumps(response_dict)
            resp = Response(js_dump,status=500,
                            mimetype='application/json')

        except RuntimeError as err:
            response_dict = {'error': 'error occured on server side.    
                              Please try again'}
            js_dump = json.dumps(response_dict)
            resp = Response(js_dump, status=500,
                            mimetype='application/json')

        return resp

    if __name__ == '__main__':
        APP.run(host='0.0.0.0', port=5000)
    -------------------------------------------------------------------

Above code shows how to create a simple GET call using flask-API. You might be feeling a lot of code here. But trust me; for writing good code, you should start writing good code to make it as a habit. So, it has all the elements which you should consider while writing the code like proper indentations(which is anyways necessary for python), following PEP8 python conventions and including exceptions.

**Code explained:**

    **APP = FlaskAPI(__name__):**
    sends the __name__ to FlaskApi Constructor to instantiate Flask Api object, which will be used accross the application
    **CORS(APP):
    **this might not be required for now but is required in case your client is browser(eg. javascript), then cross origin requests will be allowed by the server else you might get CORS related errors

Next part explains the actual method which is responsible for serving the requests:

    **@APP.route("/getJsonFromFile/<filename>", methods=['GET']):
    **defines the method type and uri to be hit by client for this method

    **def get_json_response(filename):
    **filename parameter should be kept same which transports the path parameter to the method parameter and is used accross this method

    **labels_dict = {}
    response_dict = {}:
    **these are some local dictionaries created. labels_dict contains the key-value pairs in the labelsFile.json.
    response_dict creates the actual json which contains either the labels_dict key or error key to be returned along with STATUS; depending on response.

    **try:
        with open(filename, 'r') as labels:
            labels_dict = json.load(labels)
    **Above code opens the file. This json file is loaded as key-value pair in labels_dict

    ***response_dict["*LABELS_MAPPING*"] = labels_dict
        js_dump = json.dumps(response_dict)
        resp = Response(js_dump,status=200,
                            mimetype='application/json')
    **This is the most important part of code, which shows the Flask api returning response for the request. Response is the Flask's inbuilt way to sendback data to client.*

    **Then comes Exceptions, which are self Explanatory**

    **if __name__ == '__main__':
        APP.run(host='0.0.0.0', port=5000)
    **When we run application as script, the Flask's inbuilt server starts on port 5000 due to APP.run
    Here we instruct APP(which we defined already as Flask's instance), to run the server at 0.0.0.0(localhost/127.0.0.1) which is Flask's inbuilt WSGI server

**How to test my first Flask application?**

It's really simple to test it. Just save the code and run the file as

    python3 flask_simplified.py

and you will see something like(in case there are no errors in the script) :

![Flask Server running on 5000 port](https://cdn-images-1.medium.com/max/2000/1*9m_m-fEQHdkNe_1vj-oNWQ.png)*Flask Server running on 5000 port*

As it shows, the server which is running is python’s inbuilt Flask server(APP.run starts the server at localhost:5000), which obviously **shouldn’t be used in production**. uwsgi or gunicorn or any other WSGI production server must be used. I will have a separate article on what is WSGI and how to install and deploy your application on uwsgi application server and nginx as a web server on top of it to make it production-ready.

Let's test this using the following cURL command:

    curl localhost:5000/getJsonFromFile/labelsFile.json

where labelFile.json contents can be key-value pairs in a file like:

    {"ironman":"1",
    "deadpool":"2",
    "thor":"3",
    "captainAmerica":"4",
    "angryYoungMan":"5",
    "blackPanther":"6",
    "Wolverine":"7",
    "thanos":"snapItAll"
    }

    """don't mind; keeping ironman at 1 , fan detected. ;)"""

and you should see the response or exception depending on the filename which you have provided.

    Response:
    {"LABELS_MAPPING": {"Wolverine": "7", "angryYoungMan": "5", "blackPanther": "6", "captainAmerica": "4", "deadpool": "2", "ironman": "1", "thanos": "snapItAll", "thor": "3"}, "STATUS": "true"}

## **POST API:**

In case of post request, the JSON body can be extracted from the request using request.form[‘form_data’] and yes, methods = “POST” should be provided

It will be more interesting if we get to do something with a post request, which is more frequently used in real life: **Uploading images to Server**

**Let’s Upload some images to the server (Now you get something interesting to do) :)**

In this part, I will be explaining POST call using upload method, in which the file could be sent along with the request and can be saved in the server.

    **UPLOADING FILES:**
    @APP.route("/uploadFiles", methods=['POST'])

    def upload_file():
    """uploads file to the server. This will save files to the
    directory where server is running"""

        response_dict = {}
        error_files = ""
        new_filename = ""

        try:
            new_filename = request.form['FILE_NAME']
            **recieved_files = request.files.getlist(FILES_RECEIVED_LIST)**
            for each_file in recieved_files:
                each_file_name = each_file.filename
                try:
                    **each_file.save(os.path.join(".", new_filename +
                               each_file.filename.replace(" ", "")))**
                except RuntimeError as err:
                    print("\nError in saving file: %s :: %s",
                          each_file.filename, err)
                    error_files = error_files + "," + each_file.filename

            response_dict[STATUS] = "true"
            response_dict[ERROR_FILES_LIST] = error_files
            js_dump = json.dumps(response_dict)
            resp = Response(js_dump, status=200,   
                             mimetype='application/json')

        except RuntimeError as err:
            response_dict = {'error': 'error occured on server side.
                              Please try again'}
            js_dump = json.dumps(response_dict)
            resp = Response(js_dump, status=500,
                            mimetype='application/json')
        return resp

**code explained:**

    **        new_filename = request.form['FILE_NAME']
    **FILE_NAME contains the filename in the request.(see  the cURL request below)

    **        recieved_files = request.files.getlist(FILES_RECEIVED_LIST)
    **FILES_RECEIVED_LIST contains the files in the request. (see  the cURL request below)

            **for each_file in recieved_files:
                each_file_name = each_file.filename
    **iterate through all the files recieved to save one by one

                try:
                    **each_file.save(os.path.join(".", new_filename +
                               each_file.filename.replace(" ", "")))
     **all files are saved to the path given by os.path(. will save in location where server is running ; you could give some absolute or relative path here. I am appending new_filename to each_file.filename where each_file.filename is actual name of file. You could replace the each_file.filename with new_filename)

                except RuntimeError as err:
                    **error_files = error_files + "," + each_file.filename
    **in case file is not saved, the error_files will contain all the filenames which are not saved which will be sent back to the client in json response

    **response_dict[ERROR_FILES_LIST] = error_files
    **contains ERROR_FILES_LIST key in json response which contains all the error files names seperated by ","
               ***Rest of the code is self explanatory***

And this could be triggered using following cURL request:

    ***curl -F "FILES_LIST=@./file1.jpg" -F "FILES_LIST=@./file2.jpg" -F "FILE_NAME=new_file" localhost:5000/uploadFiles***

    where file1.jpg and file2.jpg are two files to be uploaded and are present in the directory from where cURL request is triggered

and you will see the files being generated in the same location where you are running the Flask server script.

***The full code is available on my GitHub page:***
[**tseth92/flask_simplified**
*Contribute to tseth92/flask_simplified development by creating an account on GitHub.*github.com](https://github.com/tseth92/flask_simplified)

**Conclusion**:

In this short article, I have tried to explain, how Flask could be used for building APIs using a GET request dealing with reading json file and POST request dealing with uploading files in the server. Hence, using Flask, within a short timespan and with little effort in writing some basic code with discipline, you could have your APIs running in a very short span of time for applications with a short time to market in vision.

**Next Steps:**

Read my [another post](https://medium.com/@tusharseth93/is-docker-really-worth-your-time-e86dbdf374d9) on Dockerizing the application (extension to this post) which shows the ins and outs of Docker and Dockerize the Flask application in this post. With Dockerization help, you scale API development with a lot fewer headaches and lot more fun.
[**Is Docker Really Worth Your Time?**
*Docker and Containers have been a buzz word from quite a sometime now. Docker being open source has become so popular…*medium.com](https://medium.com/@tusharseth93/is-docker-really-worth-your-time-e86dbdf374d9)

I am planning to take this forward and explain in another post on **how to build this application on a production-ready server such as uwsgi and running nginx on top of it**

Thanks for reading!!. For any suggestions, doubts or discussions, shoot in the comments section.

Follow me on medium and GitHub for upcoming articles; till then, happy pythoning and good luck :)
