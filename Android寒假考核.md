# 依赖库

使用Okhttp网路通信库

```java
compile 'com.google.code.gson:gson:2.8.0'
compile 'com.squareup.okhttp3:okhttp:3.4.1'
```

# 聚合数据接口申请

https://www.juhe.cn/

申请后获得对应的AppKey

# 数据解析

## 示例

```json
{
    "resultcode":"200",
    "reason":"Return Successd!",
    "result":{
        "province":"浙江",
        "city":"杭州",
        "areacode":"杭州",
        "city":"0571",
        "zip":"310000",
        "company":"中国移动",
        "card":""
    }
}
```

# 界面布局

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.song1.pnsearch.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <EditText
            android:id="@+id/inputNp_et"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="3"
            android:hint="请输入11位手机号或前7位：" />

        <Button
            android:id="@+id/search_bt"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="查询" />

    </LinearLayout>

    <TextView
        android:id="@+id/province_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="10dp"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/city_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="5dp"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/areacode_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="5dp"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/zip_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="5dp"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/company_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="5dp"
        android:textSize="18sp" />
    
</LinearLayout>
```

# 代码



```java
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.net.HttpURLConnection;
import java.net.URL;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class MainActivity extends AppCompatActivity {

    private EditText editText;
    private Button button;
    int resultcode;
    String province, city, areacode, zip, company;
    private TextView show_area, show_areacode, show_zip, show_company;
    public static final String ADRESS = "http://apis.juhe.cn/mobile/get?phone=";
    public static final String KEY = "&key=自己申请的KEY";  //替换成自己申请的KEY
    private static String TAG = "text";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editText = (EditText) findViewById(R.id.inputNp_et);
        show_area = (TextView) findViewById(R.id.area_tv);
        show_areacode = (TextView) findViewById(R.id.areacode_tv);
        show_zip = (TextView) findViewById(R.id.zip_tv);
        show_company = (TextView) findViewById(R.id.company_tv);
        button = (Button) findViewById(R.id.search_bt);
        
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                sendRequestWithOkHttp();
            }
        });
    }
```

## 网络连接获取Json数据

```java
 private void sendRequestWithOkHttp() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                String ID = editText.getText().toString().trim();//获取输入框的手机号码
                Log.e(TAG, "ID: " + ID);
                try {
                    URL url = new URL(ADRESS + ID + KEY);  //拼接成网址
                    Log.e(TAG, "url:" + url);
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                            .url(url)
                            .build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    Log.e(TAG, "response:" + responseData);
                    showResponse(responseData);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

## 解析Json数据

```java
private void showResponse(final String data) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                try {
                    JSONObject jsonObject = new JSONObject(data);
                    resultcode = jsonObject.optInt("resultcode");
                    JSONObject jsonObject1 = jsonObject.getJSONObject("result");
                    province = jsonObject1.optString("province").toString();
                    city = jsonObject1.optString("city").toString();
                    areacode = jsonObject1.optString("areacode").toString();
                    zip = jsonObject1.optString("zip").toString();
                    company = jsonObject1.optString("company").toString();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                if (resultcode == 200) {
                    show_area.setText("号码归属地：" + province + "省" + city + "市");
                    show_areacode.setText("区号：" + areacode);
                    show_zip.setText("邮政编码：" + zip);
                    show_company.setText("运营商：中国" + company);
                } else {
                    Toast.makeText(MainActivity.this, "查询失败，请输入正确的手机号码", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }
}
```

# 添加网络权限

```
 <uses-permission android:name="android.permission.INTERNET"/>

```

