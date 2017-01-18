# RN_Java
react-native调用Native:回调（Android）
这样在JavaScript层就可以直接调用Android中的Native方法.
但在大部分的方法调用中，都需要知道调用方法之后的处理结果是什么，有没有出现异常等情况。JavaScript本身是事件驱动的语言，需在JavaScript中可以使用回调方法来处理函数返回的结果。同样地在react-native中定义了Callback和Promise的接口，用来处理JavaScript调用Java方法的回调。
Callback
Callback是react.bridge中的一个接口，它作为ReactMethod的一个传参，用来映射JavaScript的回调函数（function）。Callback接口只定义了一个方法invoke，invoke接受多个参数，这个参数必须是react.bridge中支持的参数。
首先我们定义一个类用来给JavaScript调用:

public class StoreModule extends ReactContextBaseJavaModule {

    @Override
    public String getName() {
        return "StoreModule";
    }

}
指定其名称为StoreModule。
下面我们一定个方法，用来保存userName和password到ShardPreferences中，在这个方法中，我们需要定义两个Callback参数，一个用来处理成功的情况，一个用来处理异常的情况。


    @ReactMethod
    public void addUser(String userName, String password, Callback successCallback, Callback errorCallback) {
        try {
            if (TextUtils.isEmpty(userName)) {
                errorCallback.invoke("user name is empty");
                return;
            }
            if (TextUtils.isEmpty(password)) {
                errorCallback.invoke("password is empty");
                return;
            }
            preferences.edit().putString(USER_NAME, userName).commit();
            preferences.edit().putString(PASSWORD, password).commit();
            successCallback.invoke("add user success");
        } catch (Exception e) {
            e.printStackTrace();
            errorCallback.invoke(e.getMessage());
        }
    }
现在我们有了一个带有Callback作为参数的StoreModule类，把这个类的实例加入ReactPackage的createNativeModules中，就可以在JavaScript层调用该方法。
在JavaScript中，调用这个带有Callback参数的方法:

var {NativeModules}=require('react-native');
var storeModule=NativeModules.StoreModule;
storeModule.addUser("jjz","123456",(msg)=>{
    alert(msg);
  },(errorMsg)=>{
    alert(errorMsg)
});
这里的方法回调方法我们都是使用的匿名函数，在JavaScript调用Java之后，处理结果会以Callback的形式回到JavaScript中,在JavaScript中再对相应的结果进行处理。

Promise
Promise是ES6中增加的对于异步编程和回调更加友好的API(https://segmentfault.com/a/1190000004505028),使用Promise可以更简洁，更灵活地处理回调。
在react.briage中定义的Promise接口,实现了resolve和reject的方法,resolve用来处理正确处理结果的情况，reject用来处理异常的情况。
在StoreModule定义一个支持Promise作为参数的方法:


    @ReactMethod
    public void login(String userName, String password, Promise promise) {
        String storeUserName = preferences.getString(USER_NAME, "");
        String storePassword = preferences.getString(PASSWORD, "");
        if (!equalsString(userName, storeUserName)) {
            promise.reject("0", "user name is wrong");
            return;

        }
        if (!equalsString(password, storePassword)) {
            promise.reject("1", "password is wrong");
            return;

        }
        WritableMap map = Arguments.createMap();
        map.putDouble("user_id", 1);
        promise.resolve(map);

    }
这里的WritableMap继承了ReadableMap,定义了Java和JavaScript中的参数转换.
调用这个方法的时候，在JavaScript中会返回一个Promise对象，这意味着你可以JavaScript直接使用。
在JavaScript中调用：

storeModule.login('jjz','123456').then((map)=>{
    alert(map['user_id']);
  },(code,message)=>{
    alert(message);
})
使用Promise比使用Callback更加的简洁，还能更加灵活的在多线程之间进行切换。
