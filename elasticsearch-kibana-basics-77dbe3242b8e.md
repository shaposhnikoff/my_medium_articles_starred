Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m174[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m201[39m, end: [33m202[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m430[39m, end: [33m444[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m104[39m }

# ElasticSearch+Kibana basics.



## Overview

In this article, we’re going to focus on two parts of ELK stack — ElasticSearch and Kibana.

ElasticSearch is a distributed analytics and full search engine, highly scalable and easy to use.
ES is open source, RESTful and JSON-based. It can help you manage the huge amount of data, which is why it is so common in the Big Data world.
ElasticSearch is used by such companies as Netflix, Stack Overflow, LinkedIn, etc.
Kibana is an analytic and visualization platform that has a tight integration with ElasticSearch, but we will use only Kibana’s Console to create, update, search and view the data stored in Elasticsearch indices.

## Get started

Docker commands for starting Elasticsearch and Kibana.

docker pull nshou/elasticsearch-kibana docker run -d -p 9200:9200 -p 5601:5601 nshou/elasticsearch-kibana

After a few minutes you can access to Kibana [here](http://localhost:5601)
Then go to Console, which can be found in Dev Tools(just click on the tiny spanner on the left) and let’s create an index.

## Creating an index “books”.

Below you can see a query for creating an index “books” with 15 documents, each of them has predefined id and 5 fields.

    POST /books/_bulk

    {"index": {"_id": 1}}

    {"author": "Isaac Asimov", "genre": "science fiction", "title": "I, Robot", "pages": 224, "description": "The three laws of Robotics:\n1) A robot may not injure a human being or, through inaction, allow a human being to come to harm.\n2) A robot must obey orders given to it by human beings except where such orders would conflict with the First Law.\n3) A robot must protect its own existence as long as such protection does not conflict with the First or Second Law.\nWith these three, simple directives, Isaac Asimov changed our perception of robots forever when he formulated the laws governing their behavior. In I, Robot, Asimov chronicles the development of the robot through a series of interlinked stories: from its primitive origins in the present to its ultimate perfection in the not-so-distant future--a future in which humanity itself may be rendered obsolete.\nHere are stories of robots gone mad, of mind-read robots, and robots with a sense of humor. Of robot politicians, and robots who secretly run the world--all told with the dramatic blend of science fact and science fiction that has become Asimovs trademark.", "created": "2019/09/17"}

    {"index": {"_id": 2}}

    {"author": "Isaac Asimov", "genre": "science fiction", "title": "Foundation", "pages": 244, "description": "For twelve thousand years the Galactic Empire has ruled supreme. Now it is dying. But only Hari Seldon, creator of the revolutionary science of psychohistory, can see into the future -- to a dark age of ignorance, barbarism, and warfare that will last thirty thousand years. To preserve knowledge and save mankind, Seldon gathers the best minds in the Empire -- both scientists and scholars -- and brings them to a bleak planet at the edge of the Galaxy to serve as a beacon of hope for a future generations. He calls his sanctuary the Foundation.\nBut soon the fledgling Foundation finds itself at the mercy of corrupt warlords rising in the wake of the receding Empire. Mankinds last best hope is faced with an agonizing choice: submit to the barbarians and be overrun -- or fight them and be destroyed.", "created": "2019/09/17"}

    {"index": {"_id": 3}}

    {"author": "Isaac Asimov", "genre": "science fiction", "title": "Robot Dreams", "pages": 352, "description": "Robot Dreams collects 21 of Isaac Asimovs short stories spanning the body of his fiction from the 1940s to the 1980s----exploring not only the future of technology, but the future of humanitys maturity and growth.", "created": "2019/09/17"}

    {"index": {"_id": 4}}

    {"author": "Isaac Asimov", "genre": "science fiction", "title": "The Naked Sun", "pages": 208, "description": "A millennium into the future, two advancements have altered the course of human history: the colonization of the Galaxy and the creation of the positronic brain. On the beautiful Outer World planet of Solaria, a handful of human colonists lead a hermit-like existence, their every need attended to by their faithful robot servants. To this strange and provocative planet comes Detective Elijah Baley, sent from the streets of New York with his positronic partner, the robot R. Daneel Olivaw, to solve an incredible murder that has rocked Solaria to its foundations. The victim had been so reclusive that he appeared to his associates only through holographic projection. Yet someone had gotten close enough to bludgeon him to death while robots looked on. Now Baley and Olivaw are faced with two clear impossibilities: Either the Solarian was killed by one of his robots - unthinkable under the laws of Robotics - or he was killed by the woman who loved him so much that she never came into his presence!", "created": "2019/09/17"}

    {"index": {"_id": 5}}

    {"author": "Arkady & Boris Strugatsky", "genre": "science fiction", "title": "Roadside Picnic", "pages": 145, "description": "Red Schuhart is a stalker, one of those young rebels who are compelled, in spite of extreme danger, to venture illegally into the Zone to collect the mysterious artifacts that the alien visitors left scattered around. His life is dominated by the place and the thriving black market in the alien products. But when he and his friend Kirill go into the Zone together to pick up a “full empty,” something goes wrong. And the news he gets from his girlfriend upon his return makes it inevitable that he’ll keep going back to the Zone, again and again, until he finds the answer to all his problems.", "created": "2019/09/17"}

    {"index": {"_id": 6}}

    {"author": "Arkady & Boris Strugatsky", "genre": "science fiction", "title": "Hard to Be a God", "pages": 219, "description": "The novel follows Anton, an undercover operative from the future planet Earth, in his mission on an alien planet, that is populated by human beings, whose society has not advanced beyond the Middle Ages. The novel's core idea is that human progress throughout the centuries is often cruel and bloody, and that religion and blind faith can be an effective tool of oppression, working to destroy the emerging scientific disciplines and enlightenment.", "created": "2019/09/17"}

    {"index": {"_id": 7}}

    {"author": "Arkady & Boris Strugatsky", "genre": "science fiction", "title": "Monday Starts on Saturday", "pages": 257, "description": "Sasha, a young computer programmer from Leningrad, is driving through the forests of Northwest Russia to meet up with some friends for a nature vacation. He picks up a couple of local hitchhikers, who persuade him to come work with them at the National Institute for the Technology of Witchcraft and Thaumaturgy, or NITWiT. The adventures Sasha has in the largely dysfunctional Institute involve all sorts of magical beings and devices-a wish-granting fish, a talking cat who can remember only the beginnings of stories, a sofa that translates fairy tales into reality, a motorcycle that can zoom into the imagined future, a hungry dog-size mosquito-along with a variety of wizards (including Merlin), vampires, and petty bureaucrats.", "created": "2019/09/17"}

    {"index": {"_id": 8}}

    {"author": "Lev Tolstoy", "genre": "historical fiction", "title": "War and Peace", "pages": 1225, "description": "War and Peace broadly focuses on Napoleon’s invasion of Russia in 1812 and follows three of the most well-known characters in literature: Pierre Bezukhov, the illegitimate son of a count who is fighting for his inheritance and yearning for spiritual fulfillment; Prince Andrei Bolkonsky, who leaves his family behind to fight in the war against Napoleon; and Natasha Rostov, the beautiful young daughter of a nobleman who intrigues both men.\nA s Napoleon’s army invades, Tolstoy brilliantly follows characters from diverse backgrounds—peasants and nobility, civilians and soldiers—as they struggle with the problems unique to their era, their history, and their culture. And as the novel progresses, these characters transcend their specificity, becoming some of the most moving—and human—figures in world literature.", "created": "2019/09/17"}

    {"index": {"_id": 9}}

    {"author": "Markus Zusak", "genre": "historical fiction", "title": "The Book Thief", "pages": 552, "description": "A young foster girl Liesel Meminger steals books to fund her meager existence near Munich during WWII. Her accordion-playing foster father teaches her to read, and she shares her books with neighbors during bombing raids. Meanwhile she slowly befriends the Jewish man hidden in their basement.", "created": "2019/09/17"}

    {"index": {"_id": 10}}

    {"author": "Alexandre Dumas", "genre": "historical fiction", "title": "The Three Musketeers", "pages": 625, "description": "This swashbuckling epic of chivalry, honor, and derring-do, set in France during the 1620s, is richly populated with romantic heroes, unattainable heroines, kings, queens, cavaliers, and criminals in a whirl of adventure, espionage, conspiracy, murder, vengeance, love, scandal, and suspense. Dumas transforms minor historical figures into larger- than-life characters: the Comte d’Artagnan, an impetuous young man in pursuit of glory; the beguilingly evil seductress “Milady”; the powerful and devious Cardinal Richelieu; the weak King Louis XIII and his unhappy queen—and, of course, the three musketeers themselves, Athos, Porthos, and Aramis, whose motto “all for one, one for all” has come to epitomize devoted friendship. With a plot that delivers stolen diamonds, masked balls, purloined letters, and, of course, great bouts of swordplay, The Three Musketeers is eternally entertaining.", "created": "2019/09/17"}

    {"index": {"_id": 11}}

    {"author": "Margaret Mitchell", "genre": "romance novel", "title": "Gone with the wind", "pages": 1037, "description": "Set in Georgia during the Civil War, Gone with the Wind follows the fortunes and fate of Scarlett O'Hara, the spoiled daughter of a rich plantation owner. Scarlett uses every means to claw her way out of poverty and back to wealth which she thinks is the epitome of life.", "created": "2019/09/17"}

    {"index": {"_id": 12}}

    {"author": "Jane Austen", "genre": "romance novel", "title": "Pride and Prejudice", "pages": 279, "description": "The story follows the main character, Elizabeth Bennet, as she deals with issues of manners, upbringing, morality, education, and marriage in the society of the landed gentry of the British Regency. Elizabeth is the second of five daughters of a country gentleman living near the fictional town of Meryton in Hertfordshire, near London.Page 2 of a letter from Jane Austen to her sister Cassandra (11 June 1799) in which she first mentions Pride and Prejudice, using its working title First Impressions.Set in England in the early 19th century, Pride and Prejudice tells the story of Mr and Mrs Bennet's five unmarried daughters after the rich and eligible Mr Bingley and his status-conscious friend, Mr Darcy, have moved into their neighbourhood. While Bingley takes an immediate liking to the eldest Bennet daughter, Jane, Darcy has difficulty adapting to local society and repeatedly clashes with the second-eldest Bennet daughter, Elizabeth", "created": "2019/09/17"}

    {"index": {"_id": 13}}

    {"author": "Charlotte Brontë", "genre": "romance novel", "title": "Jane Eyre", "pages": 290, "description": "Jane Eyre follows the emotions and experiences of its title character, including her growth to adulthood, and her love for Mr. Rochester, the byronic master of fictitious Thornfield Hall. In its internalisation of the action — the focus is on the gradual unfolding of Jane's moral and spiritual sensibility and all the events are coloured by a heightened intensity that was previously the domain of poetry — Jane Eyre revolutionised the art of fiction.", "created": "2019/09/17"}

    {"index": {"_id": 14}}

    {"author": "J.R.R. Tolkien", "genre": "fantasy novel", "title": "The Lord of the Rings", "pages": 1216, "description": "One Ring to rule them all, One Ring to find them, One Ring to bring them all and in the darkness bind them.\nIn ancient times the Rings of Power were crafted by the Elven-smiths, and Sauron, the Dark Lord, forged the One Ring, filling it with his own power so that he could rule all others. But the One Ring was taken from him, and though he sought it throughout Middle-earth, it remained lost to him. After many ages it fell by chance into the hands of the hobbit Bilbo Baggins.\nFrom Sauron's fastness in the Dark Tower of Mordor, his power spread far and wide. Sauron gathered all the Great Rings, but always he searched for the One Ring that would complete his dominion.\nWhen Bilbo reached his eleventy-first birthday he disappeared, bequeathing to his young cousin Frodo the Ruling Ring and a perilous quest: to journey across Middle-earth, deep into the shadow of the Dark Lord, and destroy the Ring by casting it into the Cracks of Doom.", "created": "2019/09/17"}

    {"index": {"_id": 15}}

    {"author": "C. S. Lewis", "genre": "fantasy novel", "title": "The Chronicles of Narnia", "pages": 784, "description": "Fantastic creatures, heroic deeds, epic battles in the war between good and evil, and unforgettable adventures come together in this world where magic meets reality, which has been enchanting readers of all ages for over sixty years. The Chronicles of Narnia has transcended the fantasy genre to become a part of the canon of classic literature.", "created": "2019/09/17"}

We just created an index “books”, that consists of 15 documents and fields: “author”, “genre”, “title”, “ pages”, “ description” and “created.
Once you are indexed documents and the index, ES automatically define fields and mapping types for them. It is called “dynamic mapping”.
Let’s take a look how does the mapping created by ES look.

    GET /books/_mapping

    {
      "books" : {
        "mappings" : {
          "properties" : {
            "author" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "created" : {
              "type" : "date",
              "format" : "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
            },
            "description" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "genre" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "pages" : {
              "type" : "long"
            },
            "title" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            }
          }
        }
      }
    }

The dynamic mapping for text fields make them text and keyword by default. “type” : “text”, “fields” : { “keyword” : { “type” : “keyword”, “ignore_above” : 256 } } That makes possible to perform full-text search these fields, and keyword search and aggregations using the FIELD_NAME.keyword field.

## Creating an index, with specified mapping.

If the dynamic mapping doesn’t satisfy you, you can explicit it by yourself.
Let’s delete an index.

    DELETE books

And recreate it with defining a mapping.

    PUT /books
    {
      "mappings": {
          "dynamic": false,
          "properties": {
            "added": {
              "type": "date",
              "format": "year_month_day"
            },
            "author": {
              "type": "keyword"
            },
            "description": {
              "type": "text",
              "analyzer": "my_custom_analyzer"
            },
            "genre": {
              "type": "text"
            },
            "pages": {
              "type": "integer"
            },
            "title": {
              "type": "keyword"
            }
        }
      }
    }

We just have created a mapping for index “books” Where we set “dynamic” to false , that means that dynamical adding fields to document will be disabled. In properties we defined our fields and their types. Field “added” with type “date” and “format” — “format”: “year_month_day”. This means, that in field “added” we can put the date in format “year/month/day”. Also we created the mapping for fields “author, “genre” and “title” , “description” with type “text” and “pages” with type “integer”.

NOTE You can’t update a mapping, for already existing fields.

Now we can add the documents in the “books” like we did it before.

    POST /books/_bulk 
    { ... }

## Adding a new field for an existing mapping

What if we decided to create a new field? For example, we want to add “volumes” into “The Chronicles of Narnia” and maybe in other books too.
We still can do it.

    PUT /books/_mapping/
    {
      "properties": {
        "volumes": {
          "type": "keyword"
        }
      }
    }

We just added mapping for the new field “volumes” with type “text”. By default in ES, you can put a zero or more objects in any type of field, so no need to specify array type for volumes here.

A query for updating the document with id = 15, that contain “The Chronicles of Narnia”.

    POST /books/_doc/15
    {
      "volumes": ["The Lion, the Witch and the Wardrobe", "Prince Caspian: The Return to Narnia",  "The Voyage of the Dawn Treader", "The Silver Chair", "The Horse and His Boy", "The Magician's Nephew", "The Last Battle"]
    }

Let’s take a look at an updated document.

    GET /books/_doc/15

Here is a result of our query.

    {
      "_index" : "books",
      "_type" : "_doc",
      "_id" : "15",
      "_version" : 2,
      "_seq_no" : 15,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "volumes" : [
          "The Lion, the Witch and the Wardrobe",
          "Prince Caspian: The Return to Narnia",
          "The Voyage of the Dawn Treader",
          "The Silver Chair",
          "The Horse and His Boy",
          "The Magician's Nephew",
          "The Last Battle"
        ]
      }
    }

## Analyzers

Analyzer in ES consists of 3 parts: character filter, tokenizer and token filter.
When you add a character filter it receives the original text as a stream of characters and then can transform it by removing, adding or changing characters. A tokenizer receives a stream of characters, breaks it up into individual tokens (usually individual words), and outputs a stream of tokens. So if we have a sentence consisting of 5 words, tokenizer will break it up to 5 individual words.
Token filters receive a stream of tokens and then modify them, delete or add.

ElasticSearch has a wide selection of built-in analyzers, which can be used without any configurations. You can found them [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) 
Also, you can create a custom analyzer choosing from the variety of built-in character filters, tokenizers, and token filters.

### Configuring a standard analyzer.

    PUT /books
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "english_analyzer": {
              "type": "standard",
              "stopwords": "_english_"
            }
          }
        }
      }
    }

Let’s use our analyzer.

    POST /books/_analyze
    {
      "analyzer": "english_analyzer",
      "text": "If you don’t like to read, you haven’t found the right book. – J.K. Rowling"
    }

In “analyzer” we’re specifying the name of configuring earlier analyzer and in “text” — text that we want to analyze.

Result of the “standard” analyzer

    {
      "tokens" : [
        {
          "token" : "you",
          "start_offset" : 3,
          "end_offset" : 6,
          "type" : "<ALPHANUM>",
          "position" : 1
        },
        {
          "token" : "don’t",
          "start_offset" : 7,
          "end_offset" : 12,
          "type" : "<ALPHANUM>",
          "position" : 2
        },
        {
          "token" : "like",
          "start_offset" : 13,
          "end_offset" : 17,
          "type" : "<ALPHANUM>",
          "position" : 3
        },
        {
          "token" : "read",
          "start_offset" : 21,
          "end_offset" : 25,
          "type" : "<ALPHANUM>",
          "position" : 5
        },
        {
          "token" : "you",
          "start_offset" : 27,
          "end_offset" : 30,
          "type" : "<ALPHANUM>",
          "position" : 6
        },
        {
          "token" : "haven’t",
          "start_offset" : 31,
          "end_offset" : 38,
          "type" : "<ALPHANUM>",
          "position" : 7
        },
        {
          "token" : "found",
          "start_offset" : 39,
          "end_offset" : 44,
          "type" : "<ALPHANUM>",
          "position" : 8
        },
        {
          "token" : "right",
          "start_offset" : 49,
          "end_offset" : 54,
          "type" : "<ALPHANUM>",
          "position" : 10
        },
        {
          "token" : "book",
          "start_offset" : 55,
          "end_offset" : 59,
          "type" : "<ALPHANUM>",
          "position" : 11
        },
        {
          "token" : "j.k",
          "start_offset" : 63,
          "end_offset" : 66,
          "type" : "<ALPHANUM>",
          "position" : 12
        },
        {
          "token" : "rowling",
          "start_offset" : 68,
          "end_offset" : 75,
          "type" : "<ALPHANUM>",
          "position" : 13
        }
      ]
    }

### Configuring a custom analyzer.

For custom analyzer we should specify character filters, tokenizers and token filters it will consist of.

    PUT /books
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "my_custom_analyzer": {
              "type": "custom",
              "char_filter": ["html_strip"],
              "tokenizer": "whitespace",
              "filter": ["uppercase", "unique"]
            }
          }
        }
      }
    }

We just created a custom analyzer, by specifying it’s “type” as “custom”. This analyzer has a character filter “html_strip”, that strips HTML elements from the text and replaces HTML character codes with their decoded value. “whitespace” tokenizer breaks the text into terms whenever a whitespace character will be found. “uppercase” token filter will normalize text to uppercase and “unique” will be index only unique tokens during analysis.

Let’s use our analyzer on a simple example and see how it works.

    POST /analyzers_test/_analyze
    {
    "analyzer": "my_custom_analyzer", 
      "text": "<html>If you don&apos;t like to read, you haven&apos;t found the <h3>right book</h3>. – J.K. Rowling</html>"
    }

Result of our query:

    {
      "tokens" : [
        {
          "token" : "IF",
          "start_offset" : 6,
          "end_offset" : 8,
          "type" : "word",
          "position" : 0
        },
        {
          "token" : "YOU",
          "start_offset" : 9,
          "end_offset" : 12,
          "type" : "word",
          "position" : 1
        },
        {
          "token" : "DON'T",
          "start_offset" : 13,
          "end_offset" : 23,
          "type" : "word",
          "position" : 2
        },
        {
          "token" : "LIKE",
          "start_offset" : 24,
          "end_offset" : 28,
          "type" : "word",
          "position" : 3
        },
        {
          "token" : "TO",
          "start_offset" : 29,
          "end_offset" : 31,
          "type" : "word",
          "position" : 4
        },
        {
          "token" : "READ,",
          "start_offset" : 32,
          "end_offset" : 37,
          "type" : "word",
          "position" : 5
        },
        {
          "token" : "HAVEN'T",
          "start_offset" : 42,
          "end_offset" : 54,
          "type" : "word",
          "position" : 6
        },
        {
          "token" : "FOUND",
          "start_offset" : 55,
          "end_offset" : 60,
          "type" : "word",
          "position" : 7
        },
        {
          "token" : "THE",
          "start_offset" : 61,
          "end_offset" : 64,
          "type" : "word",
          "position" : 8
        },
        {
          "token" : "RIGHT",
          "start_offset" : 69,
          "end_offset" : 74,
          "type" : "word",
          "position" : 9
        },
        {
          "token" : "BOOK",
          "start_offset" : 75,
          "end_offset" : 79,
          "type" : "word",
          "position" : 10
        },
        {
          "token" : ".",
          "start_offset" : 84,
          "end_offset" : 85,
          "type" : "word",
          "position" : 11
        },
        {
          "token" : "–",
          "start_offset" : 86,
          "end_offset" : 87,
          "type" : "word",
          "position" : 12
        },
        {
          "token" : "J.K.",
          "start_offset" : 88,
          "end_offset" : 92,
          "type" : "word",
          "position" : 13
        },
        {
          "token" : "ROWLING",
          "start_offset" : 93,
          "end_offset" : 100,
          "type" : "word",
          "position" : 14
        }
      ]
    }

Our result looks like expected. The sentence is split up correctly, all our terms are in uppercase, we see the word “YOU” only once, and instead of HTML character code &apos; we see it's decoded value '.
Suchwise you can customize your analyzer using different built-in tokenizers, character filters and token filters. But where are the results of the analysis stored? They're stored in the special data structure, that is called inverted index. An inverted index consists of the list of all the unique words that appear in any document, and for each word, a list of the documents in which it appears. An inverted index is created for each text field in the document.

In the mapping for the “books” index, we have created 4 “text” fields: “author”, “description”, “genre” and “title”. Also, we added a mapping for “volumes”, which is of the type “text” too, therefore 5 inverted indices should be created.

Inverted indices allow very officiant and fast full-text searches.

## Searching

### URI search

For creating a URI search query we specify

GET NAME_OF_INDEX/_search?q=OUR_QUERY

For example, this is how a request for searching all documents in index “books” looks:

    GET /books/_search?q=*

Let’s also create a request for searching by specific field (q=NAME_OF_FIELD:VALUE)

    GET /books/_search?q=author:Isaac Asimov

Search request by the field “author”, which should contain “Isaac Asimov”

    GET /books/_search?q=genre:science fiction

Search request by the field “genre”, which should contain “science fiction” Also, we can create a searching query using boolean logic.

    GET /books/_search?q=genre:science fiction AND author:Isaac Asimov

This query will search for documents with “genre” = “science fiction” and “author” = “Isaac Asimov”

    GET /books/_search?q=author:Charlotte Brontë OR author:Isaac Asimov

The result of this query will be all documents with author Charlotte Brontë and author Isaac Asimov.

    GET /books/_search/?size=5

The result of this query will be first 5 documents from the “books”

You can find documentation [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/search-uri-request.html) if you want to learn more about URI search.

### Query DSL search with request body.

Structure of the DSL query with search body:

    GET /<NAME_OFINDEX>/_search { "query": { <PARAMETERS> } }

Let’s start from creating a query, that will search for all existing documents and return their amount and some of their values.

    GET /books/_search
    {
      "query": {
        "match_all": {
        }
      }
    }

This query is similar to GET /books/_search?q=* A search query, that will be searching for a specific value in the specific field:

    GET /books/_search
    {
      "query": {
        "match": {
          "genre": " ROMANCE NOVEL "
        }
      }
    }

“match” query is a part of “full text” queries. Full text queries are analyzed with "standard" analyzer before matching. That means that the text provided in the field "genre" will be broken into terms and lowercased. In our case, it will search for words "romance" and "novel" in the "inverted index". So how does that search work? Foremost will be searched for documents which contain two words "romance" and "novel", after that will be searched for all documents, that contain at least one of these words.

Let’s take a look at the result of the query:

    {
      "took" : 576,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 5,
          "relation" : "eq"
        },
        "max_score" : 2.5876663,
        "hits" : [
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "11",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Margaret Mitchell",
              "genre" : "romance novel",
              "title" : "Gone with the wind",
              "pages" : 1037,
              "description" : "Set in Georgia during the Civil War, Gone with the Wind follows the fortunes and fate of Scarlett O'Hara, the spoiled daughter of a rich plantation owner. Scarlett uses every means to claw her way out of poverty and back to wealth which she thinks is the epitome of life.",
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "12",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Jane Austen",
              "genre" : "romance novel",
              "title" : "Pride and Prejudice",
              "pages" : 279,
              "description" : "The story follows the main character, Elizabeth Bennet, as she deals with issues of manners, upbringing, morality, education, and marriage in the society of the landed gentry of the British Regency. Elizabeth is the second of five daughters of a country gentleman living near the fictional town of Meryton in Hertfordshire, near London.Page 2 of a letter from Jane Austen to her sister Cassandra (11 June 1799) in which she first mentions Pride and Prejudice, using its working title First Impressions.Set in England in the early 19th century, Pride and Prejudice tells the story of Mr and Mrs Bennet's five unmarried daughters after the rich and eligible Mr Bingley and his status-conscious friend, Mr Darcy, have moved into their neighbourhood. While Bingley takes an immediate liking to the eldest Bennet daughter, Jane, Darcy has difficulty adapting to local society and repeatedly clashes with the second-eldest Bennet daughter, Elizabeth",
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "13",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Charlotte Brontë",
              "genre" : "romance novel",
              "title" : "Jane Eyre",
              "pages" : 290,
              "description" : "Jane Eyre follows the emotions and experiences of its title character, including her growth to adulthood, and her love for Mr. Rochester, the byronic master of fictitious Thornfield Hall. In its internalisation of the action — the focus is on the gradual unfolding of Jane's moral and spiritual sensibility and all the events are coloured by a heightened intensity that was previously the domain of poetry — Jane Eyre revolutionised the art of fiction.",
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "14",
            "_score" : 1.0678406,
            "_source" : {
              "author" : "J.R.R. Tolkien",
              "genre" : "fantasy novel",
              "title" : "The Lord of the Rings",
              "pages" : 1216,
              "description" : """
    One Ring to rule them all, One Ring to find them, One Ring to bring them all and in the darkness bind them.
    In ancient times the Rings of Power were crafted by the Elven-smiths, and Sauron, the Dark Lord, forged the One Ring, filling it with his own power so that he could rule all others. But the One Ring was taken from him, and though he sought it throughout Middle-earth, it remained lost to him. After many ages it fell by chance into the hands of the hobbit Bilbo Baggins.
    From Sauron's fastness in the Dark Tower of Mordor, his power spread far and wide. Sauron gathered all the Great Rings, but always he searched for the One Ring that would complete his dominion.
    When Bilbo reached his eleventy-first birthday he disappeared, bequeathing to his young cousin Frodo the Ruling Ring and a perilous quest: to journey across Middle-earth, deep into the shadow of the Dark Lord, and destroy the Ring by casting it into the Cracks of Doom.
    """,
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "15",
            "_score" : 1.0678406,
            "_source" : {
              "author" : "C. S. Lewis",
              "genre" : "fantasy novel",
              "title" : "The Chronicles of Narnia",
              "pages" : 784,
              "description" : "Fantastic creatures, heroic deeds, epic battles in the war between good and evil, and unforgettable adventures come together in this world where magic meets reality, which has been enchanting readers of all ages for over sixty years. The Chronicles of Narnia has transcended the fantasy genre to become a part of the canon of classic literature.",
              "created" : "2019/09/17"
            }
          }
        ]
      }
    }

In the hits -> total -> value we can see how many documents satisfy our search query and in the "_score" of each document we can see how well it satisfies it. Therefore in the documents where both words matches we see "_score": 2.5876663, where only one("fantasy novel") we see "_score": 1.0678406. But what if we don't want results with only one matched word? What if we want to see documents that contain exactly "romance novel" genre? That sounds reasonable - let's create another query.

    GET /books/_search
    {
      "query": {
        "match": {
          "genre": {
            "query": "ROMANCE NOVEL",
            "operator": "and"
          }
        }
      }
    }

By default, match query use Boolean operator “or”, but now, when we have specified operator “and”, all words from the query (“romance” and “novel”) must be present in the field “genre”. Result of this query looks like expected:

    {
      "took" : 13,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : 2.5876663,
        "hits" : [
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "11",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Margaret Mitchell",
              "genre" : "romance novel",
              "title" : "Gone with the wind",
              "pages" : 1037,
              "description" : "Set in Georgia during the Civil War, Gone with the Wind follows the fortunes and fate of Scarlett O'Hara, the spoiled daughter of a rich plantation owner. Scarlett uses every means to claw her way out of poverty and back to wealth which she thinks is the epitome of life.",
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "12",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Jane Austen",
              "genre" : "romance novel",
              "title" : "Pride and Prejudice",
              "pages" : 279,
              "description" : "The story follows the main character, Elizabeth Bennet, as she deals with issues of manners, upbringing, morality, education, and marriage in the society of the landed gentry of the British Regency. Elizabeth is the second of five daughters of a country gentleman living near the fictional town of Meryton in Hertfordshire, near London.Page 2 of a letter from Jane Austen to her sister Cassandra (11 June 1799) in which she first mentions Pride and Prejudice, using its working title First Impressions.Set in England in the early 19th century, Pride and Prejudice tells the story of Mr and Mrs Bennet's five unmarried daughters after the rich and eligible Mr Bingley and his status-conscious friend, Mr Darcy, have moved into their neighbourhood. While Bingley takes an immediate liking to the eldest Bennet daughter, Jane, Darcy has difficulty adapting to local society and repeatedly clashes with the second-eldest Bennet daughter, Elizabeth",
              "created" : "2019/09/17"
            }
          },
          {
            "_index" : "books",
            "_type" : "_doc",
            "_id" : "13",
            "_score" : 2.5876663,
            "_source" : {
              "author" : "Charlotte Brontë",
              "genre" : "romance novel",
              "title" : "Jane Eyre",
              "pages" : 290,
              "description" : "Jane Eyre follows the emotions and experiences of its title character, including her growth to adulthood, and her love for Mr. Rochester, the byronic master of fictitious Thornfield Hall. In its internalisation of the action — the focus is on the gradual unfolding of Jane's moral and spiritual sensibility and all the events are coloured by a heightened intensity that was previously the domain of poetry — Jane Eyre revolutionised the art of fiction.",
              "created" : "2019/09/17"
            }
          }
        ]
      }
    }

In some cases, we need a value of the field to match exactly to our query value. We can use “match_prase” query for these needs.

    GET /books/_search
    {
      "query": {
        "match_phrase":{
          "title": "Gone with the wind"
         }
      }
    }

This query will search for the book, that exactly has a name “Gone with the wind”.

    GET /books/_search
    {
      "query": {
        "multi_match":{
          "query": "robot",
          "fields": ["description", "title"]
         }
      }
    }

“multi_match” query search for a given term in more than one field. In our case, we’re searching for the term “robot” in two fields: “description” and “title”.

    GET /books/_search
    {
      "query": {
        "multi_match":{
          "query": "robot",
          "fields": ["description", "title"],
          "operator": "and"
         }
      }
    }

We can specify a boolean operator in the “multi_match” as well. The query above will search documents, that have words “robot” and “dreams” in the “description” or “title” field.

## Summary

Let’s summarize, what was done. We have created an index “books” with dynamic mapping. Then we defined a mapping for our index manually. We saw how built-in “standard” analyzer works and have configured a custom analyzer, using different built-in character filters, tokenizers, and token filters. Also, we got our hands dirty with URI searching and DSL search with the request body and tried different kinds of full text queries.

*Originally published at [https://enfuse.io](https://enfuse.io/elasticsearchkibana-basics/).*
