# Admin Bot

## Overview

The "Admin Bot" is a Telegram bot designed to gather information about users from the 
[**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider) microservice and provide it to the 
administrator of the [**"English Level Tester"**](https://github.com/delibre/tg-english-test-bot) bot once a day in a 
csv format.

### Technologies Used

* **Spring Boot Starter** v3.0.5
* **Spring Boot Starter Web** v3.0.5
* **Telegram Bots Spring Boot Starter** v6.5.0
* **Telegram Bots** v6.5.0
* **Aspose Cells** v23.3
* **Lombok** v1.18.26
* **OpenCSV** v5.5.2
* **Lombok Annotation Processor** v1.18.20
* **Spring Boot Starter Test** v3.0.5
* **Mockito Core** v5.0.0


## Guidelines

How to run the application locally:

1. Download and launch the ["User Data Provider"](https://github.com/delibre/tg-user-data-provider) application
2. Download and launch the ["English Level Tester"](https://github.com/delibre/tg-english-test-bot) application
3. Download the most recent release of this project
4. Modify the application.properties file to meet your requirements and set the springboot.active.profile variable in
environment variables of the application
```properties
admin_id = <Telegram user id of administrator>
userdata_endpoint = http://<host>:<port>/api/v1/users/last-day
bot_username = <your Telegram bot's username>
bot_token = <your Telegram bot's token>
data_sending_rate=0 1 0 * * *
server.port=<port>
```
5. Launch the application
6. Proceed according to the installation guidelines of the ["User Data Provider"](https://github.com/delibre/tg-user-data-provider)
and then launch it
7. Proceed according to the installation guidelines of the ["English Level Tester"](https://github.com/delibre/tg-english-test-bot)
   bot and then launch it 

## Source Code Review

### Bot's Class Implementation

The bot's main functionalities are implemented in a class called **"AdminBot"**. In this class, there is a method called 
"callScheduledDataSending" that is scheduled to run every day at 00:01. This method is responsible for sending CSV files
with users' information to the admin. The implementation of this scheduled task uses Spring's @Scheduled annotation and 
sets the cron expression using the value of the data_sending_rate property.
```java
@Scheduled(cron = "${data_sending_rate}")
public void callScheduledDataSending() throws IOException {
    sendUserDataToAdmin();
}
```
When the **"callScheduledDataSending"** method is executed, it calls the **"sendUserDataToAdmin"** method, which in turn 
calls **"generateUsersCSV"** from the **"BotService"** class. The **"generateUsersCSV"** method is overloaded and can take
a list of **User** objects or use the getUsers method to retrieve the list.
```java
public SendDocument generateUsersCSV() throws IOException {
  return generateUsersCSV(getUsers());
}

public SendDocument generateUsersCSV(List<User> users) throws IOException {
  StringWriter writer = new StringWriter();
  CSVWriter csvWriter = new CSVWriter(writer);
  Date now = new Date();
  SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");

  try {
      fillWriterFromUserClass(users, csvWriter);
  } catch (IllegalAccessException e) {
      log.error(e.getMessage(), e);
  }

  File mediaFile = File.createTempFile("users " + formatter.format(now), ".csv");
  FileUtils.writeStringToFile(mediaFile, writer.toString(), Charset.defaultCharset());

  InputFile csvFile = new InputFile(mediaFile, "users " + formatter.format(now) + ".csv");

  SendDocument sendDocument = new SendDocument(adminId, csvFile);
  sendDocument.setCaption("Here is the CSV file with users.");

  return sendDocument;
}
```

### Other Essentials

* The **"RestService"** class is used to retrieve user data from the endpoint of the [**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider)
microservice.
* The **"User"** class represents the user info which is retrieved from the [**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider)
microservice.


## License

MIT

The code in this repository is covered by the included license.