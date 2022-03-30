增强的理解



通过代理对象，将具体的实现多封装几层



动态代理  cglib



public UserServiceProxy extends UserService{

    UserService target;

    public void test(){

    //代理想要时间的方法

    target.test();

    //在前后进行多种代码处理，实现自己需要实现的功能

    //比如spring事务，就是在这自己新建一个连接，修改了 commit属性，要先判断是否有异常再提交，可以实现事务的特性

}

}
