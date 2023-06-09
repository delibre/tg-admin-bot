# Admin Bot

## Overview

The "Admin Bot" is a Telegram bot designed to gather information about users from the 
[**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider) microservice and provide it to the 
administrator of the [**"English Level Tester"**](https://github.com/delibre/tg-english-test-bot) bot in a 
csv file.

### Technologies Used

* **Spring Boot Starter** v3.0.5
* **Spring Boot Starter Web**
* **Spring Boot Starter Test**
* **Telegram Bots Spring Boot Starter** v6.5.0
* **Telegram Bots** v6.5.0
* **Aspose Cells** v23.3
* **Lombok** v1.18.26
* **OpenCSV** v5.5.2
* **Lombok Annotation Processor** v1.18.20
* **Mockito Core** v5.0.0

### TODO

* Parameterizing Scheduling: Retrieval of all Users for a Specific Timeframe
* "Request All Users" Button for Database Retrieval


## Guidelines

How to run the application locally:

1. Download and launch the ["User Data Provider"](https://github.com/delibre/tg-user-data-provider) application
2. Download and launch the ["English Level Tester"](https://github.com/delibre/tg-english-test-bot) application
3. Download the most recent release of this project
4. Create your own application.properties file with below variables
```properties
admin_id = <Telegram user id of administrator>
userdata_endpoint = http://<IP>:<port>/api/v1/users/last-day - the endpoint of the "User Data Provider" microservice
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

When the **"callScheduledDataSending"** method is executed, it calls the **"sendUserDataToAdmin"** method, which in 
turn calls **"generateUsersCSV"** from the **"BotService"** class. The **"generateUsersCSV"** method is overloaded and 
can take a list of **User** objects or use the getUsers method to retrieve the list.
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

The **fillWriterFromUserClass** method uses the **fillWriterWithData** method, which utilizes **reflection** to fill 
the CSV writer with data from the list of **User** objects. It takes in a list of users, an array of **Field** objects
representing the fields of the **User** class, and a CSVWriter object. For each user in the list, a new string array is 
created, and the value of each field of the user object is obtained using **reflection**. If a field is null, an empty 
string is stored in the string array.

```java
 public void fillWriterWithData(List<User> users, Field[] fields, CSVWriter csvWriter) throws IllegalAccessException {
     for (User user : users) {
         String[] row = new String[fields.length];
         for (int i = 0; i < fields.length; i++) {
             fields[i].setAccessible(true);
             Object value = fields[i].get(user);
             row[i] = value != null ? value.toString() : "";
         }
         csvWriter.writeNext(row);
     }
 }
```

### Other Essentials

* The **"RestService"** class is used to retrieve user data from the endpoint of the [**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider)
microservice.

* The **"User"** class represents the user info which is retrieved from the [**"User Data Provider"**](https://github.com/delibre/tg-user-data-provider)
microservice.

* The application was set up to utilize distinct configuration profiles for its production and local environments.


## License

MIT

The code in this repository is covered by the included license.