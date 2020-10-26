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
            out = openFileOutput("data", Context.MODE_PRIVATE);
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



