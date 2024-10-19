пятница, 28 декабря 2007 г. в 15:46:22

Соединение с базой данных в java само по себе простое. Проблема возникла из-за того, что поскольку я использую недавно вышедшую NetBeans IDE 6.0, то при исполнении кода постоянно возникала ошибка **ClassNotFoundException: postgresql.Driver**, из-за того что в Java идёт подгрузка библиотек (по аналогии с php и функицей dl), благодаря системной переменной CLASSPATH. Получается что такой умный IDE, который включает в себя уйму автоматизаций, который может сам соединиться с базой данных (закладка Services/Databases) не может втихую подключить драйвера. Трудно быть новичком - нашёл в контекстном меню Project Properties/Libraries/Compile ручное добавление.

```jsp
<%@ page language="java" import="java.sql.*" %>
<%@ page import="java.util.*" %>

<%  
Connection db;  
Statement st;  
Class.forName("org.postgresql.Driver");  
  
db = DriverManager.getConnection("jdbc:postgresql://localhost/mydatabase", "mylogin","mypassword");  
st = db.createStatement();  
  
String auto_kood = "" ;  
String auto_mark = "" ;  
String auto_hind = "" ;  
  
ResultSet rs = st.executeQuery("select auto, mark, hind from auto");  
while(rs.next()) {  
auto_kood = rs.getString("auto") ;  
auto_mark = rs.getString("mark") ;  
auto_hind = rs.getString("hind") ;  
out.println(auto_kood + "|"+ auto_mark + "|"+ auto_hind + "  
");  
}  
db.close();  
%>
```

Читайте также:

- [JDBC - средство общение](http://www.javaportal.ru/java/articles/JDBC_java_BD.html)