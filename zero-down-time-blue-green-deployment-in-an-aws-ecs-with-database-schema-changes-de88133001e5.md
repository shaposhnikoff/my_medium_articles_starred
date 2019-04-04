
# Zero down time(Blue/Green) deployment in an AWS ECS with database schema changes

On a deployment of a given backend service, one would like to have that service always available, even during deployments. Different roll-out strategies have been developed for that purpose (blue/green deployments, progressive rollouts such as in ECS…). In a Blue/green deployments, there’s an challenging case that’s not easy to handle: deploying a new version with database schema change.

General approach while deploying microservices in ECS as listed below:

* Deploy new version of application in addition to running old version of application

* Run the flyway database migration scripts

* A short while later, ECS will drain old version of apps

However, in this case what happens when something is getting wrong and you wanted to rollback to previous version even though your database schema changed? Yes, your **app got stuck**..

## How to solve database schema issue?

Solution is implementing **backward compatible** approaches to database/code changes.

Use-case scenario( Renaming column name from “**last_name**” to “**surname**”) for backward compatible database changes:

## 1. Initial situation(Creating users table with id, first_name and last_name columns)

* There will be 2 ECS services(Blue and Green) and 2 target groups(Blue and Green) for 1 microservice.

* For a live usage, end users will be accessing to your load balancer with “443” port. In this case, when you try to access [https://](about:blank)[ms1.myapp.com](https://msA1.mindsphere.io:8443))[,](about:blank) it will redirect traffic to your BlueService(v1).

* Microservice1-Blue will create “users” table with 3 columns(id, firstname, surname) in database.

So, following flyway script will create table in an initial bootup:

**Flyway migration script( V1__init.sql):**

    CREATE TABLE users (
     id serial PRIMARY KEY not null,
     first_name varchar(255) not null,
     last_name varchar(255) not null
    );
    insert into users (first_name, last_name) values ('ahmet', 'atalay');
    insert into users (first_name, last_name) values ('mehmet', 'yilmaz');

In this initial case, our app is in **v1** and created “users” table with a flyway scripts in a database. And our code seems as below:

    .
    .
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id", columnDefinition = "serial")
    private Long id;
    
    @Column(name="first_name")
    private String firstName;
    
    @Column(name="last_name")
    private String lastName;
    
    
    public String getFirstName() {
        return this.firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return this.lastName;
    }
    
    public void setLastName(String lastname) {
        this.lastName = lastname;
    }
    .
    .

![After initial deployment, v1 reaches to initial database via Blue TG with 443 port](https://cdn-images-1.medium.com/max/2000/1*rdhW7P7XSLU8JNKBqhHsrg.jpeg)*After initial deployment, v1 reaches to initial database via Blue TG with 443 port*

## 2. Adding surname column in addition to last_name column

Following steps need to be done to add surname column:

* migrate your db to create the new column called surname.

* copy the data from the last_name column to surname.

* write the code to use BOTH the new and the old column.

* Deploy your **v2** app to “Green” service with **8443** port

* Run feature tests to the app’s 8443 port(i.e. [https://ms1.myapp.com:8443)](https://msA1.mindsphere.io:8443))

* If tests are passed, switch target groups between 8443 and 443 listeners.

* Old situation: 443 => Blue-TG, 8443==> Green-TG

* After switching, new situation: 8443 ==> Blue-TG, 443 ==> Green-TG

So, following flyway script will create table in a second deploy:

**Flyway migration script( V2__Add_surname.sql):**

    ALTER TABLE users ADD surname varchar(255);
    UPDATE users SET users.surname = users.last_name;

In this second case, our app is in v2 and deployed to green service and added “**surname**” column and copied **last_name** data to **surname** with a flyway scripts in a database. And our code seems as below:

    .
    .
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id", columnDefinition = "serial")
    private Long id;
    
    @Column(name="first_name")
    private String firstName;
    
    @Column(name="last_name")
    private String lastName;
    
    @Column(name="surname")
    private String surname;
    
    public Long getId() {
        return id;
    }
    
    public User setId(Long id) {
        this.id = id;
        return this;
    }
    
    public String getFirstName() {
        return this.firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getSurname() {
        return this.surname != null ? this.surname : this.lastName;
    }
    
    public User setSurname(String surname) {
        this.lastName = surname;
        this.surname = surname;
        return this;
    }
    
    .
    .

![After second deployment, until switch between TGs, v2 will be reachable from 8443 via GreenTG](https://cdn-images-1.medium.com/max/2000/1*D23o7S5xytbzqfovYamCnw.jpeg)*After second deployment, until switch between TGs, v2 will be reachable from 8443 via GreenTG*

## 3. Removing last_name from code

Following steps need to be done to remove last_name code:

1. remove last_name parameters from code.

1. Deploy your v3 app to “Blue” service with **8443** port

1. Run feature tests to the app’s 8443 port(i.e. [https://](about:blank)[ms1.myapp.com](https://msA1.mindsphere.io:8443))[:8443)](about:blank)

1. If tests are passed, switch target groups between 8443 and 443 listeners.

1. Old situation: 443 => Green-TG , 8443==> Blue-TG

1. After switching, new situation: 8443 ==> Green-TG , 443 ==> Blue-TG

So, following flyway script will make last_name null:

**Flyway migration script( V3__Final_migration.sql):**

    UPDATE users SET users.surname = users.last_name;
    ALTER TABLE users MODIFY COLUMN last_name varchar(255) NULL DEFAULT NULL;

In this third case, our app is in v3 and deployed to blue service. And our code seems as below:

    .
    .
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id", columnDefinition = "serial")
    private Long id;
    
    @Column(name="first_name")
    private String firstName;
    
    @Column(name="surname")
    private String surname;
    
    
    public String getFirstName() {
        return this.firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getSurname() {
        return this.surname;
    }
    
    public User setSurname(String surname) {
        this.surname = surname;
        return this;
    }
    .
    .

![Now, v3 will be reachable from BlueTG via 8443 until make a switch](https://cdn-images-1.medium.com/max/2000/1*MnwwiFn-j-XI5s7JLHMFPg.jpeg)*Now, v3 will be reachable from BlueTG via 8443 until make a switch*

## 4. Removing last_name from database

Following steps need to be done to remove last_name in database:

1. remove last_name parameters from database with flyway scripts.

1. Deploy your v4 app to “Green” service with **8443** port

1. Run feature tests to the app’s 8443 port(i.e. [https://](about:blank)[ms1.myapp.com](https://msA1.mindsphere.io:8443))[:8443)](about:blank)

1. If tests are passed, switch target groups between 8443 and 443 listeners.

1. Old situation: 443 => Blue-TG , 8443==> Green-TG

1. After switching, new situation: 8443 ==> Blue-TG , 443 ==> Green-TG

So, following flyway script will delete last_name:

**Flyway migration script( V4__Remove_lastname.sql):**

    ALTER TABLE users DROP last_name;
    UPDATE users SET surname='' WHERE surname IS NULL;
    ALTER TABLE users MODIFY surname VARCHAR(255) NOT NULL;

![Lastly, v4 will be accessible from GreenTG via 8443 port until make a switch](https://cdn-images-1.medium.com/max/2000/1*2qJCMxIJq8yFA0ysxn-c0g.jpeg)*Lastly, v4 will be accessible from GreenTG via 8443 port until make a switch*

Finally, after these zero-down time deployments, our schema is completely changed as a backward compatible way and code changed.

Blue Green deployments help us to deploy with zero down time. However, it requires a lot of work from code & database perspectives as well in addition to fully automated infrastructure & CI-CD pipeline

In the next post, I will explain how to do these processes in a fully automated Gitlab CI-CD pipeline.

**References:**

[https://aws.amazon.com/blogs/compute/bluegreen-deployments-with-amazon-ecs/](https://aws.amazon.com/blogs/compute/bluegreen-deployments-with-amazon-ecs/)

[https://dzone.com/articles/zero-downtime-deployment-with-a-database-1](https://dzone.com/articles/zero-downtime-deployment-with-a-database-1)
