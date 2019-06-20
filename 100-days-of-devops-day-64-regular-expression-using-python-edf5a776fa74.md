
# 100 Days of DevOps — Day 64- Regular Expression using Python

Welcome to Day 64 of 100 Days of DevOps, Focus for today is Regular Expression using Python

*On Day53, I blogged about Regular Expression let extend that concept further and see how we can use Regular Expression using Python*
[**100 Days of DevOps — Day 53-Introduction to Regular Expression — Part 1**
*Welcome to Day 53 of 100 Days of DevOps, Focus for today is Introduction to Regular Expression*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-53-introduction-to-regular-expression-part-1-c6218f1670b7)

*Regular expressions can think like a mini-language for specifying text pattern*

    ***re.compile()**: To create a regex object*

    ***re.search()**: find a pattern in a string*

    ***re.match()**: does this entire string conform to this pattern*

    ***re.findall()**: find all patterns in this string and returns all the matches in it not just the first match*

    ***re.group()**: to get the matched string*

***Searching with Regex***

    ***match = re.search(pattern,string)***

***Pattern type(Character Classes)***

    ***\w :** sequence of word-like characters [a-zA-Z0–9_] that are not space*

    ***\d:** Any numeric digit[0–9]*

    ***\s:** whitespace characters(space,newline,tab)*

    ***\D:** match characters that are NOT numeric digits*

    ***\W:** match characters that are NOT words,digit or underscore*

    ***\S:** match characters that are NOT spaces,tab or newline*

***Repetition Group***

    ***+ :** 1 or more*

    **** :** 0 or more*

    ***?:** 0 or 1*

    ***{k}:** exactly integer K occurence*

    ***{m,n}:** m to n occurence inclusive*

    ***. **:matches any character except the newline(\n)*

    ***^:** start of the string*

    ***$:** end of string*

    ***\:** escape character*

***Example***

    *# Re module has all regular expression function in it*

    *>>> import re*

    *>>> example = “Welcome to the world of Python”*

    *>>> pattern = r’Python’*

    *>>> match = re.search(pattern,example)*

    *>>> print(match)*

    *<_sre.SRE_Match object; span=(24, 30), match=’Python’>*

    *>>> if match:*

    *… print(“found”, match.group())*

    *… else:*

    *… print(“No match found”)*

    *…*

    *found Python*

***NOTE**: r is for the raw string as Regex often uses \ backslashes(\w), so they are often raw strings(r’\d’)*

*A most popular example is finding phone number :-)*

    *>>> import re*

    *>>> message = “my number is 510–123–4567”*

    *# Here we are creating regex object,which define the pattern we are looking for 
    >>> myregex = re.compile(r’\d\d\d-\d\d\d-\d\d\d\d’)*

    *# Then we are trying to find a pattern in the string
    >>> match = myregex.search(message)*

    *# This will tell us the actual text
    >>> print(match.group())*

    *510–123–4567*

*In case we have multiple phone number, use **findall***

    *>>> import re*

    *>>> message = “my number is 510–123–4567 and my office number is 510–987–1234”*

    *>>> myregex = re.compile(r’\d\d\d-\d\d\d-\d\d\d\d’)*

    *# Find all pattern of the string and return a list objects
    >>> print(myregex.findall(message))*

    *[‘510–123–4567’, ‘510–987–1234’]*

*Let's use the **group** to separate area code with phone number. Here parenthesis has special meaning where group starts and where group end.*

    *import re*

    *myregex = re.compile(r’(\d\d\d)-(\d\d\d-\d\d\d\d)’)*

    *>>> match = myregex.search(“My number is 510–123–4567”)*

    *>>> match*

    *<_sre.SRE_Match object; span=(13, 25), match=’510–123–4567'>*

    *# This will return the full matching string
    >>> match.group()*

    *‘510–123–4567’*

    *# Only return the first matching group(area code)
    >>> match.group(1)*

    *‘510’*

    *#Second matching group(Return the whole phone number)
    >>> match.group(2)*

    *‘123–4567’*

*To find out parentheses literally in string, we need to **escape** parentheses using **backslash \(***

    *>>> myregex = re.compile(r’\(\d\d\d\)-(\d\d\d-\d\d\d\d)’)*

    *>>> match = myregex.search(“My number is (510)-123–4567”)*

    *>>> match.group()*

    *‘(510)-123–4567’*

***Pipe Character(|**) match one of many possible groups*

    *>>> lang = re.compile(r’Pyt(hon|con|mon)’)*

    *>>> match = lang.search(“Pyt**hon** is a wonderful language”)*

    *>>> match.group()*

    *‘Python’*

    *>>> match = lang.search(“Pyt**con** is a wonderful language”)*

    *>>> match.group()*

    *‘Pytcon’*

    *>>> match = lang.search(“Pyt**mon** is a wonderful language”)*

    *>>> match.group()*

    *‘Pytmon’*

*If regular expression not able to find that pattern it will return None, to verify that*

    *>>> match = lang.search(“Pytut is a wonderful language”)*

    *>>> match == None*

    *True*

### *?: zero or one time*

    *>>> import re*

    *# Here ho is optional it might occur zero time or one time
    >>> myexpr = re.compile(r’Pyt(ho)?n’)*

    *>>> match = myexpr.search(“Python a wonderful language”)*

    *>>> match.group()*

    *‘Python’*

    *>>> match = myexpr.search(“Pytn a wonderful language”)*

    *>>> match.group()*

    *‘Pytn’*

*So if we try to match this expression it will fail*

    *>>> match = myexpr.search(“Pythohon a wonderful language”)*

    *>>> match.group()*

    *Traceback (most recent call last):*

    *File “<stdin>”, line 1, in <module>*

    *AttributeError: ‘**NoneType**’ object has no attribute ‘group’*

    *>>> match ==None*

    *True*

*Same way as with our previous example of Phone Number we can make area code optional*

    *>>> myphone = re.compile(r’(\d\d\d-)**?**\d\d\d-\d\d\d\d’)*

    *>>> match = myphone.search(“My phone number is 123–4567”)*

    *>>> match.group()*

    *‘123–4567’*

### *“*” zero or more time*

    *>>> import re*

    *>>> myexpr = re.compile(r’Pyth(on)*’)*

    *>>> match = myexpr.search(“Welcome to the world of Pythononon”)*

    *>>> match.group()*

    *‘Pythononon’*

### *“+” must appear at least 1 or more time*

    *>>> myexpr = re.compile(r’Pyth(on)+’)*

    *>>> match = myexpr.search(“Welcome to the world of Pyth”)*

    *>>> match.group()*

    *Traceback (most recent call last):*

    *File "<stdin>", line 1, in <module>*

    *AttributeError: 'NoneType' object has no attribute 'group'*

    *>>> match = myexpr.search(“Welcome to the world of Python”)*

    *>>> match.group()*

    *‘Python’*

    *>>> match = myexpr.search(“Welcome to the world of Pythonononon”)*

    *>>> match.group()*

    *‘Pythonononon’*

*Now if we want to match a **specific number of times***

    *>>> myregex = re.compile(r’(Re){3}’)*

    *>>> match = myregex.search(“My matching string is ReReRe”)*

    *>>> match.group()*

    *‘ReReRe’*

    *# Range of repetitions
    >>> myregex = re.compile(r'(Re){3,5}')
    >>> match = myregex.search("My matching string is ReReReRe")*

    *>>> match.group()*

    *'ReReReRe'*

*The regular expression in Python do **greedy matches** i.e it try to match longest possible string*

    *# Instead of searching for min i.e first 3 it matches first 5*

    *>>> mydigit = re.compile(r’(\d){3,5}’)*

    *>>> match = mydigit.search(‘123456789’)*

    *>>> match.group()*

    *‘12345’*

*To do a non-greedy match **add ?** (then it matches the shortest string possible), Putting a question mark after the curly braces make it do a non-greedy match*

    *>>> mydigit = re.compile(r’(\d){3,5}?’)*

    *>>> match = mydigit.search(‘123456789’)*

    *>>> match.group()*

    *‘123’*

*Let’s take a look at a few more example which involves **character classes***

    *\w : sequence of word-like characters [a-zA-Z0–9_] that are not space*

    *\d: Any numeric digit[0–9]*

    *\s: whitespace characters(space,newline,tab)*

*Let say I need to match this address*

    *>>> import re*

    *>>> address = “123 fremont street”*

    *>>> match = re.compile(r’\d+\s\w+\s\w+’)*

    *>>>match.findall( match.finditer( match.flags match.fullmatch(*

    *>>> match.findall(address)*

    *[‘123 fremont street’]*

*We can create our **own character class***

    *#Let's create our own character class which matches all lower case vowel
    >>> myregex = re.compile(r’[aeiou]’) #To match even upper case **r'[aeiouAEIOU]'***

    *>>> mypat = “Welcome to the world of Python”*

    *>>> myregex.findall(mypat)*

    *[‘e’, ‘o’, ‘e’, ‘o’, ‘e’, ‘o’, ‘o’, ‘o’]*

*Now if we want to match **two vowels in a row***

    *>>> myregex = re.compile(r’[aeiouAEIOU]{2}’)*

    *>>> mypat = “Welcome to the world of Python ae”*

    *>>> myregex.findall(mypat)*

    *[‘ae’]*

*Negative Character Class(Use of **^** means search everything except vowel)*

    *>>> myregex = re.compile(r’[^aeiouAEIOU]’)*

    *>>> mypat = “Welcome to the world of Python ae”*

    *>>> myregex.findall(mypat)*

    *[‘W’, ‘l’, ‘c’, ‘m’, ‘ ‘, ‘t’, ‘ ‘, ‘t’, ‘h’, ‘ ‘, ‘w’, ‘r’, ‘l’, ‘d’, ‘ ‘, ‘f’, ‘ ‘, ‘P’, ‘y’, ‘t’, ‘h’, ’n’, ‘ ‘]*

*Let take look at the dot (**. : matches** any character except the newline(\n))*

    *>>> myregex = re.compile(r’.x’)*

    *>>> mypat = “Linux Unix Minix”*

    *>>> myregex.findall(mypat)*

    *[‘ux’, ‘ix’, ‘ix’]*

*Dot is majorly used with ******

    **** : 0 or more***

*Now if we change our regex to include both*

    *>>> myregex = re.compile(r’.*x’)*

    *>>> mypat = “Linux Unix Minix”*

    *>>> myregex.findall(mypat)*

    *[‘Linux Unix Minix’]*

***NOTE***

    *.*: always perform greedy match(except newline)*

    *.*?: To make it non-greedy add ?*

*Let take a look at this with the help of this example*

    *>>> mystr = ‘“Welcome to the world of Python” great language to learn”’*

    *>>> mypat = re.compile(r’”(.*?)”’)*

    *#Because of non-greedy nature it will search till first " is encountered
    >>> mypat.findall(mystr)*

    *[‘Welcome to the world of Python’]*

*But in case of greedy match*

    *>>> mypat = re.compile(r’”(.*)”’)*

    *# It will return the whole string
    >>> mypat.findall(mystr)*

    *[‘Welcome to the world of Python” great language to learn’]*

*Now as we mentioned above .* matches everything except newline*

    *>>> myexpr = “Welcome to the \n world of \n Python”*

    *>>> print(myexpr)*

    *Welcome to the*

    *world of*

    *Python*

    *>>> mypat = re.compile(r’(.*)’)*

    *>>> mypat.search(myexpr)*

    *<_sre.SRE_Match object; span=(0, 15), match=’Welcome to the ‘>*

*Now even in this case if we want to perform a greedy match add **re.DOTALL(then it will match newlines as well)***

    *>>> mypat = re.compile(r’.*’,re.DOTALL)*

    *>>> mypat.search(myexpr)*

    *<_sre.SRE_Match object; span=(0, 34), match=’Welcome to the \n world of \n Python’>*

*The second argument is really useful, especially if we want to perform a case-insensitive search**(re.I)***

    *>>> import re*

    *>>> mystr = “Why Linux Is Such An Awesome Platform”*

    *>>> mypat = re.compile(r’[aeiou]’,re.I)*

    *>>> mypat.findall(mystr)*

    *[‘i’, ‘u’, ‘I’, ‘u’, ‘A’, ‘A’, ‘e’, ‘o’, ‘e’, ‘a’, ‘o’]*

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
