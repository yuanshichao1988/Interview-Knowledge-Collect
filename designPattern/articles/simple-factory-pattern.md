# 设计模式--简单工厂模式
***
> 简单工厂模式是类的创建模式，又叫做静态工厂方法（Static Factory Method）模式。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。

#### 简单工厂模式的优点
模式的核心是工厂类。这个类含有必要的逻辑判断，可以决定在什么时候创建哪一个登录验证类的实例，而调用者则可以免除直接创建对象的责任。简单工厂模式通过这种做法实现了对责任的分割，当系统引入新的登录方式的时候无需修改调用者。

#### 简单工厂模式的缺点
这个工厂类集中了所以的创建逻辑，当有复杂的多层次等级结构时，所有的业务逻辑都在这个工厂类中实现。什么时候它不能工作了，整个系统都会受到影响。

那么简单工厂模式是在什么场景下使用呢，下面举例说明:
就拿登录功能来说，假如应用系统需要支持多种登录方式如：口令认证、域认证（口令认证通常是去数据库中验证用户，而域认证则是需要到微软的域中验证用户）。那么自然的做法就是建立一个各种登录方式都适用的接口，如下图所示：

![image](https://github.com/yuanshichao1988/Interview-Knowledge-Collect/blob/master/designPattern/articles/imgs/simple-factory-01.png)

#### 代码实例
``` java
public interface Login {
    //登录验证
    public boolean verify(String name , String password);
}
```
``` java
public class DomainLogin implements Login {

    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        /**
         * 业务逻辑
         */
        return true;
    }

}
```
``` java
public class PasswordLogin implements Login {

    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        /**
         * 业务逻辑
         */
        return true;
    }

}
```
我们还需要一个工厂类LoginManager，根据调用者不同的要求，创建出不同的登录对象并返回。而如果碰到不合法的要求，会返回一个Runtime异常。
``` java
public class LoginManager {
    public static Login factory(String type){
        if(type.equals("password")){
            
            return new PasswordLogin();
            
        }else if(type.equals("passcode")){
            
            return new DomainLogin();
            
        }else{
            /**
             * 这里抛出一个自定义异常会更恰当
             */
            throw new RuntimeException("没有找到登录类型");
        }
    }
}
```

测试类：
``` java
public class Test {
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        String loginType = "password";
        String name = "name";
        String password = "password";
        Login login = LoginManager.factory(loginType);
        boolean bool = login.verify(name, password);
        if (bool) {
            /**
             * 业务逻辑
             */
        } else {
            /**
             * 业务逻辑
             */
        }
    }
}
```
