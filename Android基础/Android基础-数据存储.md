# Android基础-数据存储

Android系统中主要提供了三种方式用于实现数据持久化存储

* 文件存储
* SharedPreference
* 数据库存储

# 文件存储

> 不对数据进行格式化处理，数据原封不动保存在文件里面，默认的存储文件/data/data/com.example.storage/files/data

* Context类里面的openFileOutput()方法

~~~java
    public void Save()
    {
        String data = "Data to save";
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try {
            out = openFileOutput("data", Context.MODE_PRIVATE); //默认的操作模式，表示该文件存在的话就进行覆盖写。
            writer = new BufferedWriter(new OutputStreamWriter(out));
            writer.write(data);

        } catch (IOException e)
        {
            e.printStackTrace();
        }finally {
            try {
                if(writer!=null)
                {
                    writer.close();
                }
            }catch (IOException e)
            {
                e.printStackTrace();
            }
        }
    }
~~~

读取文件内容：

java和android还是有区别的，我用平常IO读文件的方法可能是需要全部的路径的（我没试验哈），但是用以下的方法，我们可以有一个默认的文件夹位置，就是这个app在系统里面存储文件数据的默认位置。

~~~java
 public String load() throws IOException{
        StringBuilder stringBuilder = new StringBuilder();
        FileInputStream inputStream = null;
        BufferedReader bufferedReader = null;
        try {
            inputStream = openFileInput("data");
            bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            String line = "";
            while ((line=bufferedReader.readLine())!=null){
                stringBuilder.append(line);
            }
        }catch (IOException e){
            e.printStackTrace();
        }
        finally {
            if(bufferedReader!=null)
            {
                try {
                    bufferedReader.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
            }
        }
        return stringBuilder.toString();
    }
~~~

学到一个方法：

* EditText的setSelection()方法，移动光标到指定的位置。

# SharedPreferences

文件存储位置：

![image-20201027183200787](Android基础-数据存储.assets/image-20201027183200787.png)

目录下的。

获取SharedPreferences对象：

* Context 类getSharedPreferences()方法

~~~java
 public void onClick(View v) {
                SharedPreferences.Editor editor = getSharedPreferences("data",MODE_PRIVATE).edit();
                editor.putString("name","tom");
                editor.putInt("age",28);
                editor.putBoolean("married",false);
                editor.apply();
            }
~~~

我们以一个onclick事件来测试，这会生成一个xml文件，可以看到里面存储的有我们的数据。	

* Activity类里面的getPreferences()方法

它只接受一个操作模式参数，因为使用这个方法时会自动将当前活动的类名作为SharedPreferences的文件名。

* PreferenceManager 类里面的getDefaultSharedPreferences()方法。

这个是一个静态方法，它接受一个Context参数，并自动使用当前应用的包名作为前缀来命名文件。

`从SharedPreferences`里面读取数据：

`SharedPreferences对象`里面提供了一系列的get方法帮助读取数据，每种方法对应着一个SharedPreferences.Editor里面的一种put方法（参见上述代码），这些get方法都接受两个参数，一个是put传入的键值对里面的键，第二个参数是默认值，表示当传入的键找不到对应的值得时候会以怎么样的默认值进行返回。

~~~java
 public void onClick(View v) {
                SharedPreferences sharedPreferences = getSharedPreferences("data",MODE_PRIVATE);
                String name = sharedPreferences.getString("name","");
                int age = sharedPreferences.getInt("age",0);
                Log.d("MainActivity","name is "+name);
                Log.d("MainActivity","name is "+age);
            }
~~~

### 记住密码

这里实现一个记住密码的功能来帮助理解这一个存储的方式：

~~~java
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
        editText = (EditText)findViewById(R.id.username);
        passwordText = (EditText)findViewById(R.id.password);
        checkBox = (CheckBox) findViewById(R.id.checkBox);
        boolean isRemeber = sharedPreferences.getBoolean("remember_password",false);
        if(isRemeber){
            String username = sharedPreferences.getString("username","");
            String passwd = sharedPreferences.getString("passwd","");
            editText.setText(username);
            passwordText.setText(passwd);
            checkBox.setChecked(true);
        }
        Button button = (Button)findViewById(R.id.button);
        Button button1 = (Button)findViewById(R.id.button2);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String username = editText.getText().toString();
                String passwd = passwordText.getText().toString();
                if(username.equals("admin") && passwd.equals("123456")){
                    editor = sharedPreferences.edit();
                    if(checkBox.isChecked()){
                        editor.putBoolean("remember_password",true);
                        editor.putString("username",username);
                        editor.putString("passwd",passwd);
                        Toast.makeText(MainActivity.this,"存储成功",Toast.LENGTH_LONG);
                    }
                    else {
                        editor.clear();
                    }
                }
                else {
                    Toast.makeText(MainActivity.this,"登陆错误",Toast.LENGTH_LONG);
                }
                editor.apply();
            }
        });
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SharedPreferences sharedPreferences = getSharedPreferences("com.example.sharedpreferencestest_preferences",MODE_PRIVATE);
                String name = sharedPreferences.getString("username","");
                int age = sharedPreferences.getInt("age",0);
                Log.d("MainActivity","name is"+name);
                Log.d("MainActivity","name is"+age);
            }
        });
    }
~~~

# SQLite数据库存储

> android系统内置数据库SQLite,

android提供了`SQLiteOpenHelper`帮助类，这是一个抽象类。其中包含两个抽象方法`onCreate()`和`onUpgrade()`。

其也包含两个实例方法`getRead`