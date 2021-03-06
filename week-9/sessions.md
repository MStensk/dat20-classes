# Sessions

When developing web applications sometimes we need to store data for a user. That could fx be to save data for a logged in user. It could also be to store previous searches from a user like Google maps does. 

Now the problem with that is that HTTP is stateless. This means that there is no state stored in a request. Each request is a completely new request.



This problem is solved using sessions. 



## Learning objectives

- What is a session and why do we use it?
- Implementing session for spring boot
- Rendering sessions using Thymeleaf



## Sessions

In Spring boot sessions is implemented using cookies. Cookies is a small text file saved in the browser. This small text file gives the user an id so it knows who you are! Now every time you request a page from a specific domain, the `cookie` is sent wth the request. Now the server know who requested a specific page.



Here is a screenshot of how the cookie is stored in the browser

![Screenshot 2021-02-23 at 14.56.11](./assets/cookie-browser.png)



Here is a screenshot of how the cookie id is sent in the request

![Screenshot 2021-02-23 at 14.58.04](/Users/benjamin-hughes/Documents/projects/dat20-classes/week-9/assets/cookie-sent-on-request.png)



![Screenshot 2021-02-24 at 10.18.38](./assets/session-server.png)



### Creating a session in Spring boot

Firstly add the dependency that makes the sessions work:

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-core</artifactId>
</dependency>
```



For creating the session in Spring boot we will be working with two specific classes:

- `HttpServletRequest` - Represents the request. It lives from the controller receives the request to a response is sent to the browser. Once the `request` object has been responded, the `request` object will be deleted. Said another way: The request scope ends when the associated response is finished.
- `HttpSession` - The session lives across multiple requests. The session scope ends when the session has been timed out by the client or server. 



Let's get to a real example:

```java
@GetMapping("/set-session")
@ResponseBody
public String setSession(HttpServletRequest request) {
    HttpSession session = request.getSession();
    session.setAttribute("username", "CookieMonster42");
  
    return "Username saved in the session";
}

@GetMapping("/get-session")
@ResponseBody
public String getSession(HttpServletRequest request) {
    HttpSession session = request.getSession();
    String username = (String) session.getAttribute("username");
  
    return username;
}

@GetMapping("/invalidate-session")
@ResponseBody
public String invalidateSession(HttpServletRequest request) {
    HttpSession session = request.getSession();
  	session.invalidate();

  	return "Session invalidated";
}
```

Lets disect what happens here:

There are 3 endpoints

1. `/set-session` - Here we save the value (`"CookieMonster42"`) that is stored under the key `"username"`
2. `/get-session` - Here we get the value stored under the `"username"` key
3. `/invalidate-session` - Here we invalidate (delete) the session



Let's dive into some of the code

- `request.getSession() ` to get the session we use the `getSession` method on the `request`

- `session.setAttribute("key", "value")` - Will save some data in the session. It will be in a key value format. Save this value at this key

- `session.getAttribute("key")` - Will get some value at the specified key
- `session.invalidate()` will invalidate/delete the session



Lets try and manually delete the cookie and see what happens.



### Session storage

For now the session is stored in memory in the spring boot application. It is possible to store it in a database aswell. 



### Session timeout

To decide how long the session should be kept alive add the following to the `application.properties` file

```java
server.servlet.session.timeout=60s
```

Minimum timeout is 60 seconds



https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-jdbc.html

https://attacomsian.com/blog/thymeleaf-get-session-attributes



### What can i save?

Anything you would like! `String`, `Integer`, `ArrayList`, some class it does not matter!



```java
@GetMapping("/set-users")
@ResponseBody
public String setSession(HttpServletRequest request) {
    HttpSession session = request.getSession();
    ArrayList<Session> users = new ArrayList<>();
    User user1 = new Session("benjamin", 23);
    User user2 = new Session("peter", 45);
    users.add(user1);
    users.add(user2);
    session.setAttribute("users", users);

    return "two users saved in the session";
}

@GetMapping("/get-session")
@ResponseBody
public String getSession(HttpServletRequest request) {
  HttpSession session = request.getSession();
  List<User> users = (List<User>)  session.getAttribute("users");

  return username.get(0).name;
}
```



## Sessions in Thymeleaf

To access the session in Thymeleaf simply use the `session`. We dont need to send the session from the `@Controller` to the view. 



```java
@GetMapping("/get-session-rendered")
public String getSessionRendered() {
    return "session.html";
}
```



**session.html**

```html
<div th:text="${session.username}">John Doe</div>
```



Since the sessions might not be there, its a good idea to check if they session data is available first:

```html
<div th:if="${session.containsKey('username')}" th:text="${session.email}"></div>
```



We can also use `session.size()` and `session.isEmpty()`



## Exercise time!



Lav den sådan at det hele handler om vejret! Men bare med de forksellige niveauer. Så et er at de får en random by. Derefter kan brugeren gemme den by han er i. Derefter gemme en liste af byer. Derefter finde vejret for de byer.



### Get fake name - level 1

Lets create a website where a user can get a fake name! It should have 3 endpoints:



| Endpoint            | Description                                                  | Method |
| ------------------- | ------------------------------------------------------------ | ------ |
| `/assign-fake-name` | At this endpoint the user gets a fake name assigned.  We do three things:<br />1. We generate the fake name using the code below. <br />2. We save that name in the session!<br />3. Return a string sayng something like `Fake name has been assigned. Go to /get-fake-name to see the name` | `GET`  |
| `/get-fake-name`    | Here the user can see the name that was assigned (and saved in the session) at the endpoint `/assign-fake-name` | `GET`  |
| `/delete-fake-name` | Here the user can delete the fake name. This happens with us invalidating the whole session | `GET`  |



#### Generate fake name

Add this dependency:

```xml
<dependency>
    <groupId>com.github.javafaker</groupId>
    <artifactId>javafaker</artifactId>
    <version>0.12</version>
</dependency>
```



Code to generate a fake name:

```java
Faker faker = new Faker();
String fakeName = faker.name().fullName();
```



### Notes website - level 2

Lets create a website where users can create notes

| Endpoint    | Description                    | Method |
| ----------- | ------------------------------ | ------ |
| `/notes`    | See a list of your saved notes | `GET`  |
| `/add-note` | Create                         |        |



### Weather site - level 3

Lets create a website where users can see the weather for different cities!

It should be possible for a user to add a new city. For each city added the user should see the weather for all of those cities

| url         | Description                                                  | Method |
| ----------- | ------------------------------------------------------------ | ------ |
| `/add-city` | A user should be able to add a city name and the users firstname using a `form` | `GET`  |
| `/`         | Should show the weather for the cities that have been added  | `GET`  |
| `save-city` | Should save the city name to the list of cities              | `POST` |

**Remember the GET, POST, REDIRECT pattern**



### Technicalities

For getting the weather data we will be using this service: https://home.openweathermap.org

To get an api so we can get weather, go here: https://openweathermap.org/api click on `Subscribe` for `Current Weather Data`. Follow the signup flow. When you are done you should have an API key.







## Glossary

Contains words and their explanation