---
title: "Android应用安全防护和逆向分析 基础篇①"
date: 2018-07-02T17:33:05+08:00
lastmod: 2018-07-02T17:33:05+08:00
draft: false
description: "Android中锁屏密码加密算法分析"
tags: [
    "android安全",
    "读书笔记"
]
categories: ["读书笔记","Android安全"]
author: "Vorblock"

---

## 第一章 Android中锁屏密码加密算法分析  
#### 1. 锁屏密码方式：  

   - 手势
   - 九宫格连线
   - 输入密码
   - 指纹、人脸、虹膜
   - 可穿戴设备

#### 2. 这儿分析手势密码和输入密码  
   找到android源代码中的LockPatternUtils,java 这个工具类   
   路径：Android-5.1.1\frameworks\base\core\java\com\android\internal\widget  

   > 2.1 输入密码算法分析    (5.1版本的源代码 和书上细微差异)  

   ```java
   public byte[] passwordToHash(String password, int userId) {//参数为密码和对应用户ID 默认0
           if (password == null) {
               return null;
           }
   
           try {
               byte[] saltedPassword = (password + getSalt(userId)).getBytes();  
               byte[] sha1 = MessageDigest.getInstance("SHA-1").digest(saltedPassword);
               byte[] md5 = MessageDigest.getInstance("MD5").digest(saltedPassword);
   //首先让 password+salt值 再SHA-1和MD5
               byte[] combined = new byte[sha1.length + md5.length];
               System.arraycopy(sha1, 0, combined, 0, sha1.length);
               System.arraycopy(md5, 0, combined, sha1.length, md5.length);
   //装换成hex值 再拼接起来
               final char[] hexEncoded = HexEncoding.encode(combined);
               return new String(hexEncoded).getBytes(StandardCharsets.UTF_8);
           } catch (NoSuchAlgorithmException e) {
               throw new AssertionError("Missing digest algorithm: ", e);
           }
       }
   ```

   如何获取设备对应的salt值：

   ```java
       private String getSalt(int userId) {
           long salt = getLong(LOCK_PASSWORD_SALT_KEY, 0, userId);
           if (salt == 0) {   //值为0  重新生成
               try {
                   salt = SecureRandom.getInstance("SHA1PRNG").nextLong();
                   setLong(LOCK_PASSWORD_SALT_KEY, salt, userId);  //保存值
                   Log.v(TAG, "Initialized lock password salt for user: " + userId);
               } catch (NoSuchAlgorithmException e) {
                   throw new IllegalStateException("Couldn't get SecureRandom number", e);
               }
           }
           return Long.toHexString(salt);       //  hex之后返回
       }
   ```

   继续跟踪 看保存的地方

   ```java
   private long getLong(String secureSettingKey, long defaultValue, int userHandle) {
           try {
               return getLockSettings().getLong(secureSettingKey, defaultValue, userHandle);
           } catch (RemoteException re) {
               return defaultValue;
           }
       }
   ```

   继续跟踪代码

   ```java
   @VisibleForTesting
       public ILockSettings getLockSettings() {
           if (mLockSettingsService == null) {
               ILockSettings service = ILockSettings.Stub.asInterface(
                       ServiceManager.getService("lock_settings"));   //获取服务来操作
               mLockSettingsService = service;
           }
           return mLockSettingsService;
       }
   ```

   在android中 这种获取服务的方式最终实现逻辑都是在XXXService类中  

   这里在LockSettingService.java中  找到这个类的getLong方法

   ```java
   public long getLong(String key, long defaultValue, int userId) {
           checkReadPermission(key, userId);
           String value = getStringUnchecked(key, null, userId);
           return TextUtils.isEmpty(value) ? defaultValue : Long.parseLong(value);
       }
   ```

   保存在数据库？

   继续跟踪

   ```java
   static class Injector {
   
           protected Context mContext;
   
           public Injector(Context context) {
               mContext = context;
           }
   
           public Context getContext() {
               return mContext;
           }
   
           public Handler getHandler() {
               return new Handler();
           }
   
           public LockSettingsStorage getStorage() {
               final LockSettingsStorage storage = new LockSettingsStorage(mContext);
               storage.setDatabaseOnCreateCallback(new LockSettingsStorage.Callback() {
                   @Override
                   public void initialize(SQLiteDatabase db) {
                       // Get the lockscreen default from a system property, if available
                       boolean lockScreenDisable = SystemProperties.getBoolean(
                               "ro.lockscreen.disable.default", false);
                       if (lockScreenDisable) {
                           storage.writeKeyValue(db, LockPatternUtils.DISABLE_LOCKSCREEN_KEY, "1", 0);
                       }
                   }
               });
               return storage;
           }
   
   public LockSettingsService(Context context) {
           this(new Injector(context));
       }
   
   
   ```

   继续  查看LockSettingsStorage.java 类中   存在数据库中

   ```java
   static class DatabaseHelper extends SQLiteOpenHelper {
           private static final String TAG = "LockSettingsDB";
           private static final String DATABASE_NAME = "locksettings.db";
   
           private static final int DATABASE_VERSION = 2;
           private static final int IDLE_CONNECTION_TIMEOUT_MS = 30000;
   
           private Callback mCallback;
   
           public DatabaseHelper(Context context) {
               super(context, DATABASE_NAME, null, DATABASE_VERSION);
               setWriteAheadLoggingEnabled(true);
               // Memory optimization - close idle connections after 30s of inactivity
               setIdleConnectionTimeout(IDLE_CONNECTION_TIMEOUT_MS);
           }
   
           public void setCallback(Callback callback) {
               mCallback = callback;
           }
   
           private void createTable(SQLiteDatabase db) {
               db.execSQL("CREATE TABLE " + TABLE + " (" +
                       "_id INTEGER PRIMARY KEY AUTOINCREMENT," +
                       COLUMN_KEY + " TEXT," +
                       COLUMN_USERID + " INTEGER," +
                       COLUMN_VALUE + " TEXT" +
                       ");");
           }
   
           @Override
           public void onCreate(SQLiteDatabase db) {
               createTable(db);
               if (mCallback != null) {
                   mCallback.initialize(db);
               }
           }
   
           @Override
           public void onUpgrade(SQLiteDatabase db, int oldVersion, int currentVersion) {
               int upgradeVersion = oldVersion;
               if (upgradeVersion == 1) {
                   // Previously migrated lock screen widget settings. Now defunct.
                   upgradeVersion = 2;
               }
   
               if (upgradeVersion != DATABASE_VERSION) {
                   Log.w(TAG, "Failed to upgrade database!");
               }
           }
       }
   ```

   看到了数据库的名字叫作：locksettings.db  保存在了：

   ```java
       private static final String SYSTEM_DIRECTORY = "/system/";    //目录
       private static final String LOCK_PATTERN_FILE = "gatekeeper.pattern.key";   
       private static final String BASE_ZERO_LOCK_PATTERN_FILE = "gatekeeper.gesture.key";
       private static final String LEGACY_LOCK_PATTERN_FILE = "gesture.key";    //key1
       private static final String LOCK_PASSWORD_FILE = "gatekeeper.password.key";
       private static final String LEGACY_LOCK_PASSWORD_FILE = "password.key";    //key2
       private static final String CHILD_PROFILE_LOCK_FILE = "gatekeeper.profile.key";
       private static final String SYNTHETIC_PASSWORD_DIRECTORY = "spblob/";
   ```

   数据库文件存在/data/system/locksetting.db    

   测试  在/data/system/下看到password.key   

   ![](http://my-md-1253484710.coscd.myqcloud.com/android-note-1-1.png)

   打开看看：![](http://my-md-1253484710.coscd.myqcloud.com/android-note-1-2.png)

   手动简单实现加密算法：

   ```java
   public byte[] passwordToHash(String password) {
           if (password == null) {
               return null;
           }
           byte [] hashed = null;
           try {
               byte[] saltedPassword = (password + SALT).getBytes();    //SALT 值从数据库中得到 拿到之后进行hex转换
               byte[] sha1 = MessageDigest.getInstance("SHA-1").digest(saltedPassword);
               byte[] md5 = MessageDigest.getInstance("MD5").digest(saltedPassword);
               hashed = (toHex(sha1)+toHex(md5)).getBytes();
           } catch(Exception e){
               
           }
           return hashed;
       }
       private static String toHex(byte[] ary){
           final String hex = "102031398sjdfklaj";
           String ret = "";
           for(int i=0;i<ary.length;i++){
               ret += hex.charAt((ary[i]>> 4)& 0xf);
               ret += hex.charAt(ary[i]& 0xf);
           }
           return ret;
       }
   ```

   SALT 的值可以从数据库中拿到 也可以利用反射获取

   总结：

   ​	MD5(输的明文密码+设备的salt).hex+ SHA1(输的的明文密码+设备的salt值).hex

   

   > 2.2 手势密码分析   

大致同上

#### 3. 简要：

   ​	九宫格团装化成字节数组->sha1 加密  即可

   ​	 其实大致流程和分析输入密码差不多   保存到本地的目录、/data/system/gesture.key 文件

