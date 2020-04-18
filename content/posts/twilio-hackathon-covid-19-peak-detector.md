---
title: "Twilio Hackathon - Covid 19 Peak Detector"
date: 2020-04-18T17:00:00+01:00
tags: ["hackathon", "software"]
draft: false
---

![image](/images/twilio-hackathon-covid-19-peak-detector.png)

I'm taking part in the Twilio x DEV.to Hackathon and this is my project. I'll show here how I built it and how it works. Keep reading if you want to know more.

<!--more-->

#### The application

COVID-19 Peak Detector is a Java Spring Boot application with a DynamoDB store that displays graph information with stats about COVID-19. You can select the country you are interested in and you can create an alert by entering your phone number.

A scheduled process will check the data periodically and will send the alert when a peak is detected.

For simplicity and to avoid unnecessary spam, after sending the alert the process removes it from the database.

In addition to that, there is an anonymous chat available, in which COVID-19 news are posted every 5 minutes. People can comment on the news or just talk about the weather.

![image](/images/twilio-hackathon-covid-19-peak-detector-2.png)

#### How I built it

In a nutshell, this application uses the following techonologies:

- Spring Boot as the web framework
- DynamoDB as the database
- Thymeleaf for templating
- Chart.js for graphs
- Twilio SMS for alerts
- Twilio Chat for in-app messaging
- Twilio Sync for news
- Gradle as the build tool

For the COVID-19 data and the news, I'm using free APIs that are available via the Postman website here https://covid-19-apis.postman.com/.

The demo application was deployed in AWS using Elastic Beanstalk.

I'll explain some of the most interesting pieces.

#### The Welcome controller

This is where everything starts. The controller is in charge of calling the different services to fetch the necessary data to populate the view.

From here, we call the DataService to fetch the historical data, and we use Faker, a Java library to generate random data sets, to assign a random username to the user landing the page. This username will be important for the chat widget.

Fun fact, all the usernames are characters from Game of Thrones 🤓

It is important to mention that the data about COVID-19 doesn't change very frequently. Therefore, I'm caching the data in memory for 12 hourse before calling the third-party API again.

This is a small optimisation, because those APIs are not very flexible, and they force you to fetch all the historical data in every call. This way, we minimise the amount of data transfered over the wire in exchange for a bit of up-to-dateness.


#### The Data API

We expose our data in an endpoint available at `/countries/{countryAndProvince}`. This endpoint is called using AJAX from the browser everytime the user selects a different country from the dropdown.

When called, it will pass the request to the data service so that we can load the data for the selected country and craft a response for caller.

We then read the information form Javascript and update the graph using Chart.js.

This is a monolithic application, so typically we render the views on the server-side using Thymeleaf. However, it is useful to also call the back-end from Javascript and render changes without refreshing the page. This provides a better user experience.

#### Creating an alert

Another thing we can do here is to create an SMS alert for a given country. When the user signs up, a confirmation SMS will be sent. When a peak is detected, a second SMS will be sent and the phone number will be deleted from the DynamoDB.

But before that, we need to insert the data in Dynamo. For that, we use `spring-data-dynamodb`. This library allows us to set up our repositories like normal JPA repositories. It's very convenient, and it makes it very easy to read and write from your tables.

The key components here are the DyanmoDBConfiguration class, the model and the repository.

```
@Configuration
@EnableDynamoDBRepositories(basePackages = "com.plopcas.twiliohackathon.cpd.repository")
public class DynamoDBConfiguration {
    @Value("${amazon.dynamodb.endpoint}")
    private String amazonDynamoDBEndpoint;

    @Value("${amazon.aws.accesskey}")
    private String amazonAWSAccessKey;

    @Value("${amazon.aws.secretkey}")
    private String amazonAWSSecretKey;

    @Bean
    public AmazonDynamoDB amazonDynamoDB() {
        AmazonDynamoDB dynamoDB = new AmazonDynamoDBClient(amazonAWSCredentials());

        if (!StringUtils.isEmpty(amazonDynamoDBEndpoint)) {
            dynamoDB.setEndpoint(amazonDynamoDBEndpoint);
        }

        // check table and create if it does not exist
        DynamoDBMapper mapper = new DynamoDBMapper(dynamoDB);
        CreateTableRequest request = mapper.generateCreateTableRequest(Alert.class);
        request.setProvisionedThroughput(new ProvisionedThroughput(1L, 1L));
        TableUtils.createTableIfNotExists(dynamoDB, request);

        return dynamoDB;
    }

    @Bean
    public AWSCredentials amazonAWSCredentials() {
        return new BasicAWSCredentials(amazonAWSAccessKey, amazonAWSSecretKey);
    }
}
```

```
@DynamoDBTable(tableName = "Alert")
public class Alert {
    private String id;
    private String country;
    private String phone;

    @DynamoDBHashKey
    @DynamoDBAutoGeneratedKey
    public String getId() {
        return id;
    }

    @DynamoDBAttribute
    public String getCountry() {
        return country;
    }

    @DynamoDBAttribute
    public String getPhone() {
        return phone;
    }

    // setters and constructors

}
```

```
public interface AlertRepository extends CrudRepository<Alert, String> {
    @EnableScan
    public List<Alert> findAll();
}
```

I am creating the table upfront, on app startup, if it doesn't exist already.

Of course, all the sensitive information is externalised as environment variables.

Like usual with Spring JPA repositories, all the CRUD operations are automagically available. I'm explicitely declaring the `findAll` one so that I can annotate it with `@EnableScan`. All methods of the repository are implemented as queries by default. However, if there is no index to query, these will fail. So you need to explicitly enable the scan operations with the annotation. 

More information can be found [here](https://www.baeldung.com/spring-data-dynamodb).

Once these pieces are reading, we can call the `save` method in the repository (it's not there, it's magic), to insert new alerts. This is done in the AlertService as expected.

```
Alert alert = new Alert(country, phone);
alertRepository.save(alert);
```

#### The scheduled peak detector

In order to detect peaks we use a very naive approach. It's a hack project, please bear with me. We check if the number of cases today is lower than yesterday and lower than the day before yesterday.

If that is the case, we consider a peak detected, and we send an alert to all the phone numbers that subscribed to that country.

In order to run the process periodically, we use the `@Scheduled` annotation with a fixedRate of 12 hours.

```
@Scheduled(fixedRate = 12 * 60 * 60 * 1000)
```

#### The SMS service

Twilio makes it very easy to send SMS messages. Once you have your account set up, it's as easy as using their SDK, get an access token and start sending messages. Note that you need to get a phone number first, that will be the number from which the messages will be sent.

```
@Service
public class SmsService {
    private String fromPhone;

    public SmsService(@Value("${twilio.accountSid}") String accountSid,
                      @Value("${twilio.authToken}") String authToken,
                      @Value("${twilio.fromPhone}") String fromPhone) {
        this.fromPhone = fromPhone;
        Twilio.init(accountSid, authToken);
    }

    public void sendConfirmationSms(String toPhone) {
        Message message = Message.creator(
                new com.twilio.type.PhoneNumber(toPhone),
                new com.twilio.type.PhoneNumber(fromPhone),
                "COVID-19 Peak Detector - Your alert has been created, thanks!")
                .create();
    }

    public void sendAlertSms(String toPhone, String country) {
        Message message = Message.creator(
                new com.twilio.type.PhoneNumber(toPhone),
                new com.twilio.type.PhoneNumber(fromPhone),
                "COVID-19 Peak Detector - A peak has been reached in "
                        + country + "! Thanks for using our service :)")
                .create();
    }
}
```

To get the tokens, I created a dedicated TokenService that uses the API key and API secret to generate tokens with the necessary grants depending on the service. In my case I need two types of tokens, one for the Chat service and one for the Sync service.

#### The Chat

The idea behind the chat is to have a forum for people to talk about the situation, ramble and share. I created a very simple UI, and using the Twilio Javascript SDK, I create a channel, and sychronise all the messages between all the users.

In addition to this, to spice things up, I use the Sync service to store a shared state that contains COVID-19 news. Every 5 minutes, a process posts one headline for people to chat about. Using Sync, I can post the same headline to every browser at the same time.

Keeping state in a distributed environment is not easy, and Twilio makes it very straightforward. Again, this requires some configuration in you Twilio console, but it's fairly simple and it's all explained in their documentation.

#### That's it!

I hope you enjoyed this post. All the code is publicly available in GitHub here

https://github.com/plopcas/covid19-peak-detector

and the demo application (at least for a few days) is available here

http://covid19.plopcas.com

Stay safe, stay strong, stay home 👋
