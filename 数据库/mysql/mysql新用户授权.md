```sql
create user 'salary'@'%' identified by 'salary';

grant all privileges on *.* to 'salary'@'%' identified by 'salary' ;

grant all privileges on *.* to 'salary'@'%' identified by 'salary' WITH GRANT OPTION;
```





https://www.cnblogs.com/ccw869476711/p/11856620.html