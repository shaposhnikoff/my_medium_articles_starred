
# Adding and Updating data  in csv file via Flask API



This article will take you through how to add and update data in a csv file via flask API. I experienced this while I was doing a group project building a recommendation system, where I had to add and update ratings of a user for a particular event in a csv file. This was done in order to make sure that the latest rating of the user is used to recommend events.(The system is trained every 24 hours).

The following steps was taken by me to create the API mentioned above

1. **Create an Anaconda virtual environment using the anaconda navigator and install flask (You can do it without anaconda, I used anaconda in my project).**

Before getting started anaconda should be installed. Visit the anaconda website and follow the guidelines given to install. After that open the anaconda navigator and select environments from the side bar. Afterwards click on create at the bottom to create a new environment and name it.(I named it RecSys). Select the latest version of python and click OK.

![anaconda navigator](https://cdn-images-1.medium.com/max/3830/1*GdT7BgJ5Rhh2oUTBEOPx3g.png)*anaconda navigator*

Select the new environment(which was just created). The green play logo will appear in the RecSys environment to denote that it is the virtual environment currently activated.

Now click on the green play button and select open terminal in the drop down which appears as you click the button. A conda terminal will appear. (In the terminal RecSys will appear in parenthesis which denotes that the terminal is open in the current virtual environment followed by a path).

![conda terminal](https://cdn-images-1.medium.com/max/2000/1*XCmgu6ERKvr_BhxgiiBSqA.png)*conda terminal*

Then type the below commands to install flask and pandas(module used for adding and updating) respectively .

    pip install flask
    pip install pandas

and then close the terminal.

**2.Write function for adding and updating the data in the csv file**

In my project I had a csv file named rating.csv. It had 3 columns namely user-id, event-id and rating as show in the image below

![An extract of the rating.csv file](https://cdn-images-1.medium.com/max/2420/1*DlJ6IXSdE9KZrGPo95rr2A.png)*An extract of the rating.csv file*

Now, move to home in the anaconda navigator and check weather the current environment is activated and install spyder and then launch it(the image below shows launch cause I have already installed it).

![Home screen in Anaconda navigator](https://cdn-images-1.medium.com/max/2560/1*5cVsFHxJv9wZB3RWsTDWrw.png)*Home screen in Anaconda navigator*

Now create a python file and name it(I have named it Rating.py).

**Note: Put the csv file into the directory of where the Rating.py exist.**

Copy paste the below code in the python file just created and save it.

    import pandas as pd 
     
    def addRate(userId,eventId,rating):

    # Creating the first Dataframe using dictionary 
     isThere = False
     
    # Creating the first Dataframe using dictionary 
     df1 = pd.read_csv(‘./rating.csv’)
     
     try:
     userInt = int(userId)
     eventInt = int(eventId)
     ratingInt = int(rating)
     print(userInt,eventInt,ratingInt)
     
     if( 0 > ratingInt or ratingInt > 5):
     invalid = “invalid”
     return invalid
     
     except ValueError:
     error = “error”
     return error
     
     for index, row in df1.iterrows(): 
     print(row[‘event-id’], row[‘user-id’]) 
     if str(row[‘event-id’]) == str(eventId) and str(row[‘user-id’]) == str(userId):
     row[‘rating’] = rating
     isThere = True

    # Creating the Second Dataframe using dictionary 
     
     if isThere != True: 
     
     df2 = pd.DataFrame({“user-id”:[userId], 
     “event-id”:[eventId], 
     “rating”:[rating]}) 
     
     dff = df1.append(df2, ignore_index = True)
     dff.to_csv(r’./rating.csv’, index = False)
     
     add = “added”
     return add
     
     else:
     df1.to_csv(r’./rating.csv’, index = False) 
     update = “updated”
     return update

Where ever the path is put in the code above it refers to the relative path of the data-set file(rating.csv) from app.py which we will be creating later. (I created the app.py in the same directory there for the path is mentioned as ‘./rating.csv’).

**Note : user-id, event-id and rating is the column names in the rating.csv file.**

The above code has validation for

1. Ratings in the range 1 to 5.

1. If there is already a rating for a particular event by a particular user then it will update the rating (Simply one user can rate one event only once).

1. If there is any error in the data sent to the API an error message is displayed.

3. **Create the Flask API route and run the flask API in localhost.**

Now create a file named app.py in the same directory where the csv file and the Rating.py file exist and copy and paste the code

    from flask import Flask, request, jsonify
    from Rating import addRate

    app = Flask(__name__)

    [@app](http://twitter.com/app).route(“/rating/add”,methods=[‘POST’])
    def addRating():
     eventId = request.form[‘eventId’] 
     userId = request.form[‘userId’]
     rating = request.form[‘rating’]
     print(userId,” “,eventId,” “,rating)
     status = addRate(userId, eventId, rating)
     
     return status

    # Running the server in localhost:5000 
    if __name__ == ‘__main__’:
     app.run(debug=True, host=’0.0.0.0', port=5000)

This code receives the userId, eventId and the rating in post method and invokes the addRate method in Rating.py.

Now we have to run the application created. Type the below commands in the terminal opened under the 1st guideline(**Create an Anaconda virtual environment using the anaconda navigator and install flask**).

    set FLASK_APP=app.py
    set FLASK_ENV=development
    flask run

**4. Test whether every thing works fine**

For testing the application you have to install Postman (You can try any HTTP API testing tool, This is just my preference). After installing launch it and enter the URL for the flask route we just created and change the request type to POST. Then click on body and select x-www-form-urlencoded and enter the keys as show in the image below and click on send. You will get responses such as “added”, “updated”, “error”, “invalid”.

![postman GUI for post request](https://cdn-images-1.medium.com/max/2930/1*Juh-hczP6tQLmlMDjSQpOw.png)*postman GUI for post request*

Change the values and test whether the expected outputs are received .

That’s it 
Cheers!!!
