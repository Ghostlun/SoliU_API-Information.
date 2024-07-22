# MentalHealthApp API

## Overview

This project provides a set of APIs to manage user information, survey results, diary entries, and mood lists for a mental health application. The APIs interact with a Firestore database to store and retrieve data.

## Implementation Approach

I have created a REST API using Djangos. The database used is Google Firebase Firestore, which is a NoSQL database. The reason for choosing Firebase is its cloud capabilities, allowing for scalable and real-time data handling. 

Currently, we are about to deploy the iOS version of the MentalHealth app. In the future, we plan to release web and Android versions as well. Firestore's NoSQL nature offers several benefits:
- **Scalability**: It can handle large amounts of data and high traffic efficiently.
- **Flexibility**: NoSQL databases allow for more flexible data models, which can evolve over time.
- **Real-time Updates**: Firestore provides real-time data synchronization, making it ideal for applications that require live updates.

All servers are deployed on AWS EC2 instances. The benefits of using EC2 include:
- **Scalability**: Easily scalable to handle increased load.
- **Flexibility**: Wide range of instance types to optimize cost and performance.
- **Reliability**: High availability and redundancy options.
- **Security**: Robust security features and compliance certifications.

### Gunicorn Setup for MentalHealth App
Gunicorn (Green Unicorn) is a Python WSGI HTTP server for UNIX. It is a pre-fork worker model, which means it forks multiple worker processes to handle requests. It is widely used in production environments due to its simplicity, performance, and compatibility with various web frameworks.
- **Scalability**: Gunicorn can handle a large number of concurrent requests by forking multiple worker processes.
- **Flexibility**: It supports various worker types and configurations to optimize performance based on application needs.
- **Performance**: Efficiently uses server resources and provides a robust solution for serving Python web applications.
- **Compatibility**: Works seamlessly with WSGI applications and integrates well with web frameworks like Django.

### Request Flow
The flow of a request in the deployed application is as follows:
1. **Request** -> **Web Server** -> **WSGI Server** -> **WSGI-supported Web Application**

#### 1. Configuration

The Gunicorn server is configured to bind to all available IP addresses on port 8000. This allows Nginx, acting as a reverse proxy, to forward requests to Gunicorn.

```bash
gunicorn --bind 0.0.0.0:8000 mentalhealthapi.wsgi
```

#### 2. Running Gunicorn in the Background

To ensure the server continues running even after logging out from the terminal, `nohup` is used:

```bash
nohup gunicorn --bind 0.0.0.0:8000 mentalhealthapi.wsgi &
```
This command runs Gunicorn in the background, ignoring hangup signals and appending output to `nohup.out`.

---


## Nginx

Nginx is a web server similar to Apache, but it is known for being lightweight and efficient. It handles multiple responses simultaneously and utilizes an event-driven architecture to transfer all outputs, making it highly responsive and fast.

### Key Features of Nginx

- **Lightweight**: Designed to be a more efficient alternative to traditional web servers.
- **Event-Driven**: Utilizes event listeners to manage outputs, rather than relying on a thread-per-request model.
- **High Performance**: Capable of handling a large number of concurrent connections with minimal resource usage.

### Configuration

- **Location**: Nginx configuration files are typically located in the `/etc/nginx` directory.
- **Settings**: Configuration files include settings for the number of users, performance optimizations, and other server parameters.

### Example Nginx Configuration File for AWS EC2

Here's an example of what an Nginx configuration file might look like:

```nginx
server {
    listen 80;
    server_name EC2 instance's public DNS

    location / {
        proxy_pass http://unix:/home/ubuntu/mentalhealthapp-backend/gunicorn.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Benefits

- **Scalability**: Easily handles increased load by efficiently managing resources.
- **Flexibility**: Wide range of configuration options to optimize performance based on specific needs.
- **Reliability**: Provides fast and reliable responses, making it suitable for high-traffic applications.

---

## Base URL

```
https://api.yourproject.com
```

## Endpoints

### User Management

#### Get All Users

- **Endpoint**: `GET /users`
- **Description**: Retrieve a list of all users.
- **Response**:

```json
[
  {
    "demographicInformation": {
      "email": "john.doe@example.com",
      "gender": "Male",
      "workStatus": "Employed",
      "ethnicity": "Asian",
      "age": 30,
      "password": "hashedpassword",
      "nickname": "John"
    },
    "surveyResultsList": [],
    "dailyMoodList": [],
    "diaryEntriesList": []
  }
]
```

#### Get User by Email

- **Endpoint**: `GET /users/email/{email}`
- **Description**: Retrieve user information by email.
- **Parameters**:
  - **email** (path): The email of the user.
- **Response**:

```json
{
  "demographicInformation": {
    "email": "john.doe@example.com",
    "gender": "Male",
    "workStatus": "Employed",
    "ethnicity": "Asian",
    "age": 30,
    "password": "hashedpassword",
    "nickname": "John"
  },
  "surveyResultsList": [],
  "dailyMoodList": [],
  "diaryEntriesList": []
}
```

- **Explanation**: This endpoint retrieves user information by email. It will be used once the user successfully logs in to fetch their data into the application.

#### Create User Information

- **Endpoint**: `POST /users`
- **Description**: Create a new user.
- **Explanation**: Once a user creates an account, their information will be saved in the `Users` table in Firebase (a NoSQL database). By setting up `surveyResultsList`, `dailyMoodList`, and `diaryEntriesList` arrays as empty, the user can insert or delete entries later.
- **Request Body**:

```json
{
  "email": "john.doe@example.com",
  "gender": "Male",
  "workStatus": "Employed",
  "ethnicity": "Asian",
  "age": 30,
  "password": "hashedpassword",
  "nickname": "John"
}
```

- **Response**:

```json
{
  "message": "User information created successfully"
}
```

### Survey Management

#### Get Average Scores

- **Endpoint**: `GET /survey/average-score`
- **Description**: Retrieve average scores for various survey metrics.
- **Response**:

```json
{
  "Depression": 2.5,
  "Anxiety": 3.0,
  "Stress": 2.8,
  "Social Media Addiction": 1.5,
  "Loneliness": 2.2,
  "HRQOL": 4.0
}
```

#### Add Survey Result

- **Endpoint**: `POST /survey`
- **Description**: Add a new survey result for a user.
- **Request Body**:

```json
{
  "email": "john.doe@example.com",
  "surveyDate": "07/15/2024",
  "surveyAnswer": [1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5]
}
```

- **Response**:

```json
{
  "message": "Survey results updated successfully"
}
```

### Diary Management

#### Add Diary Entry

- **Endpoint**: `POST /diary`
- **Description**: Add a new diary entry for a user.
- **Request Body**:

```json
{
  "email": "john.doe@example.com",
  "diaryEntryData": {
    "date": "07/15/2024",
    "entry": "Today was a good day."
  }
}
```

- **Response**:

```json
{
  "message": "Diary entry added successfully"
}
```

### Mood Management

#### Add Mood Entry

- **Endpoint**: `POST /mood`
- **Description**: Add a new mood entry for a user.
- **Request Body**:

```json
{
  "email": "john.doe@example.com",
  "moodEntry": {
    "date": "07/15/2024",
    "mood": "Happy"
  }
}
```

- **Response**:

```json
{
  "message": "Mood entry added successfully"
}
```

### Email Management

#### Check if Email Exists

- **Endpoint**: `GET /email/check/{email}`
- **Description**: Check if an email exists in the database.
- **Parameters**:
  - **email** (path): The email to check.
- **Response**:

```json
{
  "exists": true
}
```

## Data Structure

### User Data

```json
{
  "demographicInformation": {
    "email": "john.doe@example.com",
    "gender": "Male",
    "workStatus": "Employed",
    "ethnicity": "Asian",
    "age": 30,
    "password": "hashedpassword",
    "nickname": "John"
  },
  "surveyResultsList": [],
  "dailyMoodList": [],
  "diaryEntriesList": []
}
```

### Variable Structure

- **surveyResultsList**: An array of survey results, each containing the survey date and the answers provided.
  ```json
  [
    {
      "surveyDate": "07/15/2024",
      "surveyAnswer": [1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5]
    }
  ]
  ```

- **dailyMoodList**: An array of daily mood entries, each containing the date and the mood recorded.
  ```json
  [
    {
      "date": "07/15/2024",
      "mood": "Happy"
    }
  ]
  ```

- **diaryEntriesList**: An array of diary entries, each containing the date and the diary entry content.
  ```json
  [
    {
      "date": "07/15/2024",
      "entry": "Today was a good day."
    }
  ]
  ```

## Contributions

Feel free to contribute to this project by creating issues or submitting pull requests. Please ensure that your contributions follow the project's guidelines and code of conduct.

## License

This project is licensed under the MIT License.
