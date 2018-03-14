ResultType和ResultMap两种返回结果的区别：



ResultType: 指定返回的类型名，类型的属性值和select选择的列名一一对应。



select的选择结果会以(列名，值)形成HashMap然后对应的ResultType中指定的类的对应属性获取相应的属性值

```
package com.someapp.model;
public class User {
  private int id;
  private String username;
  private String hashedPassword;
  
  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getUsername() {
    return username;
  }
  public void setUsername(String username) {
    this.username = username;
  }
  public String getHashedPassword() {
    return hashedPassword;
  }
  public void setHashedPassword(String hashedPassword) {
    this.hashedPassword = hashedPassword;
  }
}
```

```
<select id="selectUsers" resultType="User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

处理列名字不匹配的方法：

```
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```





ResultMap相当于是自定义了Map的映射关系，这个就没有严格的要求列名和属性名相同



```
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```



```
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```