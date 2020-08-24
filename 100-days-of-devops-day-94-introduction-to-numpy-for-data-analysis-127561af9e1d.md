
# 100 Days of DevOps — Day 94-Introduction to Numpy for Data Analysis

Welcome to Day 94 of 100 Days of DevOps, Focus for today is Introduction to Numpy for Data Analysis

*NumPy is a Linear Algebra Library for Python and the reason it’s so important that all libraries in the PyData Ecosystem rely on NumPy as the main building block.*

***Installing Numpy***

    *# pip2 install numpy*

    *Collecting numpy*

    *Using cached numpy-1.12.1-cp27-cp27mu-manylinux1_x86_64.whl*

    *Installing collected packages: numpy*

    *Successfully installed numpy-1.12.1*

*It’s highly recommended to install Python using Anaconda distribution to make sure all underlying dependencies(**such as Linear Algebra libraries**)all sync up with the use of a conda install.*

*In case you have conda install [https://www.continuum.io/downloads](https://www.continuum.io/downloads)*

    *conda install numpy*

*Numpy arrays are the main reason we use Numpy and they come in two flavors*

* ***Vectors (1-d arrays)***

* ***Matrices (2-d arrays)***

    *# 1-D Array*

    *>>> test = [1,2,3]*

    *>>> import numpy as np*

    *# We got the array
    >>> np.array(test)*

    *array([1, 2, 3])*

    *>>> arr = np.array(test)*

*Let’s take a look at **2-D array***

    *>>> test1 = [[1,2,3],[4,5,6],[7,8,9]]*

    *>>> test1*

    *[[1, 2, 3], [4, 5, 6], [7, 8, 9]]*

    *>>> np.array(test1)*

    *array([[1, 2, 3],*

    *[4, 5, 6],*

    *[7, 8, 9]])*

*But the most common way to generate NumPy array is using **arange** function(similar to range in Python)*

    *#Similar to range(start,stop,step),stop not included and indexing start with zero 
    >>> np.arange(0,10)*

    *array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])*

*But if we are looking for a specific type of arrays*

    *>>> np.zeros(3)*

    *array([ 0., 0., 0.])*

    *#We are passing Tuple where **first value represent row** and **second represent column***

    *>>> np.zeros((3,2))*

    *array([[ 0., 0.],*

    *[ 0., 0.],*

    *[ 0., 0.]])*

*Similarly for ones*

    *>>> np.ones(2)*

    *array([ 1., 1.])*

    *>>> np.ones((2,2))*

    *array([[ 1., 1.],*

    *[ 1., 1.]])*

*Now let’s take a look at **linspace***

    *#It will give 9 evenly spaced point between 0 and 3(It return 1D vector)
    >>> np.linspace(0,3,9)*

    *array([ 0. , 0.375, 0.75 , 1.125, 1.5 , 1.875, 2.25 , 2.625, 3. ])*

    *>>> np.linspace(0,10,3)*

    *array([  0.,   5.,  10.])*

*Let’s create an **identity matrix**(2-D square matrix where the number of rows is equal to the number of columns and diagonal of 1)*

![](https://cdn-images-1.medium.com/max/2000/1*f-Wvh5YO1rYItKzB6cpTkQ.png)

*To create an array of **random number***

    *#1-D, it create random sample uniformly distributed between 0 to 1
    >>> np.random.rand(3)*

    *array([ 0.87169008, 0.51446765, 0.65027072])*

    *#2-D
    >>> np.random.rand(3,3)*

    *array([[ 0.4217015 , 0.86314141, 0.14976093],*

    *[ 0.4348433 , 0.68860693, 0.88575823],*

    *[ 0.56613179, 0.56030069, 0.51783999]])*

*Now if I want **random integer***

    *#This will give random integer between 1 and 50
    >>> np.random.randint(1,50)*

    *27*

    *#In case if we need 10 random integer
    >>> np.random.randint(1,50,10)*

    *array([39, 34, 30, 21, 18, 30,  3,  6, 37, 11])*

*We can **reshape** our existing array*

    *>>> np.arange(25)*

    *array([ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,*

    *17, 18, 19, 20, 21, 22, 23, 24])*

    *>>> arr = np.arange(25)*

    *>>> arr*

    *array([ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,*

    *17, 18, 19, 20, 21, 22, 23, 24])*

    *>>> arr.reshape(5,5)*

    *array([[ 0, 1, 2, 3, 4],*

    *[ 5, 6, 7, 8, 9],*

    *[10, 11, 12, 13, 14],*

    *[15, 16, 17, 18, 19],*

    *[20, 21, 22, 23, 24]])*

*Let’s take a look at some other methods*

    *>>> np.random.randint(0,50,10)*

    *array([10, 40, 18, 30, 6, 40, 49, 23, 3, 18])*

    *>>> ranint = np.random.randint(0,50,10)*

    *>>> ranint*

    *array([18, 49, 6, 28, 30, 10, 46, 11, 40, 16])*

    *#It will return **max value** of the array
    >>> ranint.max()*

    *49*

    ***#Minimum value**
    >>> ranint.min()*

    *6*

    *>>> ranint*

    *array([18, 49,  6, 28, 30, 10, 46, 11, 40, 16])*

    *#To find out the position
    >>> ranint.argmin()*

    *2*

    *>>> ranint.argmax()*

    *1*

*To find out the shape of an array*

    *>>> arr.shape*

    *(25,)*

    *>>> arr.reshape(5,5)*

    *array([[ 0, 1, 2, 3, 4],*

    *[ 5, 6, 7, 8, 9],*

    *[10, 11, 12, 13, 14],*

    *[15, 16, 17, 18, 19],*

    *[20, 21, 22, 23, 24]])*

    *>>> arr = arr.reshape(5,5)*

    *>>> arr.shape*

    *(5, 5)*

*To find out the **datatype***

    *>>> arr.dtype*

    *dtype(‘int64’)*

*Indexing in case of NumPy*

    *>>> import numpy as np*

    *>>> arr =np.arange(0,11)*

    *>>> arr*

    *array([ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])*

    *>>> arr[0]*

    *0*

    *>>> arr[0:4]*

    *array([0, 1, 2, 3])*

*How Numpy array is different from the Python list because of there ability to broadcast*

    *>>> arr[:] = 20*

    *>>> arr*

    *array([20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20])*

    *# Now let's try to slice this array*

    *>>> arr1 = arr[0:5]*

    *>>> arr1*

    *array([0, 1, 2, 3, 4])*

    *>>> arr1[:] = 50*

    *>>> arr1*

    *array([50, 50, 50, 50, 50])*

    *#But as you can see the side effect it change the original array too(i.e data is not copied it's just the view of original array)
    **>>> arr***

    ***array([50, 50, 50, 50, 50,  5,  6,  7,  8,  9, 10])***

    *#If we want to avoid this feature, we can copy the array and then perform broadcast on the top of it*

    *>>> arr2 = arr.copy()*

    *>>> arr2*

    *array([50, 50, 50, 50, 50,  5,  6,  7,  8,  9, 10])*

    *>>> arr2[6:10] = 100*

    *>>> arr2*

    *array([ 50,  50,  50,  50,  50,   5, 100, 100, 100, 100,  10])*

    *>>> arr*

    *array([50, 50, 50, 50, 50,  5,  6,  7,  8,  9, 10])*

*Indexing** 2-D Array(Matrices)***

    *>>> arr = ([1,2,3],[4,5,6],[7,8,9])
    >>> arr*

    *([1, 2, 3], [4, 5, 6], [7, 8, 9])*

    *>>> arr1 = np.array(arr)*

    *>>> arr1*

    *array([[1, 2, 3],*

    *[4, 5, 6],*

    *[7, 8, 9]])*

    *>>> arr[1]*

    *[4, 5, 6]*

    *# To grab 5(Indexing Start with zero)*

    *>>> arr1[1][1]*

    *5*

    *#Much shortcut method
    >>> arr1[1,1]*

    *5*

*To grab elements from 2-D array*

    *>>> arr*

    *array([[1, 2, 3],*

    *[4, 5, 6],*

    *[7, 8, 9]])*

    *#This will grab everything from Row 1 except last element(2) and staring from element 1 upto the end from Row 2*

    *>>> arr[:2,1:]*

    *array([[2, 3],*

    *[5, 6]])*

***Conditional Selection**: This will return a boolean value*

    *>>> arr*

    *array([ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])*

    *>>> arr > 5*

    *array([False, False, False, False, False, True, True, True, True, True], dtype=bool)*

    *# We can save this value to an array and perform boolean selection*

    *>>> my_arr = arr > 5*

    *>>> my_arr*

    *array([False, False, False, False, False,  True,  True,  True,  True,  True], dtype=bool)*

    *>>> arr[my_arr]*

    *array([ 6,  7,  8,  9, 10])*

    ***#OR much easier way***

    *>>> arr[arr > 5]*

    *array([ 6,  7,  8,  9, 10])*

    *>>> arr[arr < 5]*

    *array([1, 2, 3, 4])*

***Operations***

    *# It's the same operation as we are doing with Normal Python*

    *>>> arr = np.arange(0,10)*

    *>>> arr*

    *array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])*

    *>>> arr*

    *array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])*

    ***#Addition**
    >>> arr + arr*

    *array([ 0, 2, 4, 6, 8, 10, 12, 14, 16, 18])*

    ***#Substraction**
    >>> arr — arr*

    *array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0])*

    ***#Multiplication**
    >>> arr * arr*

    *array([ 0, 1, 4, 9, 16, 25, 36, 49, 64, 81])*

    ***#Broadcast(It add's/substract/multiply 100 to each element)**
    >>> arr + 100*

    *array([100, 101, 102, 103, 104, 105, 106, 107, 108, 109])*

    *>>> arr - 100*

    *array([-100,  -99,  -98,  -97,  -96,  -95,  -94,  -93,  -92,  -91])*

    *>>> arr * 100*

    *array([  0, 100, 200, 300, 400, 500, 600, 700, 800, 900])*

*In case of Python if we try to divide one with zero we will get division by zero exception*

    *>>> 0/0*

    *Traceback (most recent call last):*

    *File "<stdin>", line 1, in <module>*

    *ZeroDivisionError: division by zero*

    ***OR***

    *>>> 1/0*

    *Traceback (most recent call last):*

    *File “<stdin>”, line 1, in <module>*

    *ZeroDivisionError: division by zero*

*In case of Numpy if we try to divide by zero we will not get any exception but it returns **nan**(not a number)*

    *#Not giving you 
    >>> arr/arr*

    *__main__:1: RuntimeWarning: invalid value encountered in true_divide*

    *array([ **nan**, 1., 1., 1., 1., 1., 1., 1., 1., 1.])*

*and in case of one divide by zero it will return **infinity***

    *>>> 1/arr*

    *array([ **inf**, 1. , 0.5 , 0.33333333, 0.25 ,*

    *0.2 , 0.16666667, 0.14285714, 0.125 , 0.11111111])*

***Universal Array Function***

    *#Square root
    >>> np.sqrt(arr)*

    *array([ 0. , 1. , 1.41421356, 1.73205081, 2. ,*

    *2.23606798, 2.44948974, 2.64575131, 2.82842712, 3. ])*

    *#Exponential
    >>> np.exp(arr)*

    *array([ 1.00000000e+00, 2.71828183e+00, 7.38905610e+00,*

    *2.00855369e+01, 5.45981500e+01, 1.48413159e+02,*

    *4.03428793e+02, 1.09663316e+03, 2.98095799e+03,*

    *8.10308393e+03])*

    *#Maximum
    >>> np.max(arr)*

    *9*

    *#Minimum
    >>> np.min(arr)*

    *0*

    *#Logarithmic
    >>> np.log(arr)*

    *__main__:1: RuntimeWarning: divide by zero encountered in log*

    *array([       -inf,  0.        ,  0.69314718,  1.09861229,  1.38629436,*

    *1.60943791,  1.79175947,  1.94591015,  2.07944154,  2.19722458])*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 93-Python Functions**
*Welcome to Day 93 of 100 Days of DevOps, Focus for today is Python Functions*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-93-python-functions-f7a8f92fb563)
