---
layout: post
title:  "[Android]Android_Fragment详解"
date:   2015-06-11 11:17:10
categories: 17---【Android】#########
comments: true
---

### 一、概述：
Fragment代表了Activity的子模块，因此可以把Fragment理解成Activity片段。Fragment必须被"嵌入"Activity中使用。
![图片](http://owk5gjdrg.bkt.clouddn.com/0019Android_Fragment%E8%AF%A6%E8%A7%A3.png)

### 二、简介：
Fragment也就是UI片段，开发者实现Fragment必须继承Fragment基类。

1.Fragment总是作为Activity界面的组成部分。

2.在Activiy运行过程中，可调用FragmentManager的add()、remove()、replace()方法动态的添加、删除和替换Fragment。

3.Fragment可以响应自己的输入事件，拥有自己的生命周期，但他们的生命周期直接被其所属的Activity的生命周期控制。

#### ①创建Fragment：
创建Fragment通常需要实现如下三个方法：

1.onCreate()：系统创建Fragment对象后回调该方法，实现代码中只初始化想要在Fragment中保持的必要组件。

2.onCreateView()：当Fragment绘制界面组件时会回调该方法。

3.onPause()：当用户离开该Fragment时将回调该方法。
```java

public class HelloFragment extends Fragment {  
  
    private Button button;  
  
    // 初始化Fragment，实例化在Fragment中的成员变量  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    // 给Fragment 加载UI的布局  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.hello, null);  
        button = (Button) view.findViewById(R.id.button1);  
        button.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View arg0) {  
                // TODO Auto-generated method stub  
                Toast.makeText(getActivity(), "hello world", 1).show();  
            }  
        });  
        return view;  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
}  
```
```java

public class MainActivity extends Activity {  
  
    // 如何加载Fragment到Activity中  
    private FragmentManager manager;  
    private FragmentTransaction transaction;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        manager = getFragmentManager();  
        transaction = manager.beginTransaction();  
        HelloFragment helloFragment = new HelloFragment();  
        transaction.add(R.id.line, helloFragment);  
        transaction.commit();  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
}  
```
#### ②Fragment的生命周期：
![图片](http://owk5gjdrg.bkt.clouddn.com/0020Android_Fragment%E8%AF%A6%E8%A7%A3.png)
1.onAttach()

当碎片和活动建立关联的时候调用。

2.onCreaView()

当碎片创建视图(加载布局)时调用。

3.onActivityCreated()

确保与碎片相关联的活动一定已经创建完毕的时候调用。

4.onDestroyView()

当碎片关联的视图被移除的时候调用。

5.onDetach()

当与碎片关联的视图被移除的时候调用。

下面通过一个实例进行说明：
```java

public class MyFragment extends Fragment {  
    private final String TAG = "MyFragment";  
    private Button button;  
  
    @Override  
    public void onAttach(Activity activity) {  
        // TODO Auto-generated method stub  
        super.onAttach(activity);  
        Log.i(TAG, "-MyFragment-->>onAttach");  
    }  
    /** 
     * 进行本地成员变量的初始化 
     */  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
        Log.i(TAG, "-MyFragment-->>onCreate");  
    }  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        Log.i(TAG, "-MyFragment-->>onCreateView");  
        View view = inflater.inflate(R.layout.f1, null);  
        //如果要是更新UI的操作，必须要在View view被加载成功，才能更新  
        button = (Button) view.findViewById(R.id.button1);  
        button.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View arg0) {  
                // TODO Auto-generated method stub  
                Toast.makeText(getActivity(), "Click me", 1).show();  
            }  
        });  
        return view;  
    }  
    @Override  
    public void onActivityCreated(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onActivityCreated(savedInstanceState);  
        Log.i(TAG, "-MyFragment-->>onActivityCreated");  
    }  
    @Override  
    public void onStart() {  
        // TODO Auto-generated method stub  
        super.onStart();  
        Log.i(TAG, "-MyFragment-->>onStart");  
    }  
    @Override  
    public void onResume() {  
        // TODO Auto-generated method stub  
        super.onResume();  
        Log.i(TAG, "-MyFragment-->>onResume");  
    }  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
        Log.i(TAG, "-MyFragment-->>onPause");  
    }  
    @Override  
    public void onStop() {  
        // TODO Auto-generated method stub  
        super.onStop();  
        Log.i(TAG, "-MyFragment-->>onStop");  
    }  
    @Override  
    public void onDestroyView() {  
        // TODO Auto-generated method stub  
        super.onDestroyView();  
        Log.i(TAG, "-MyFragment-->>onDestroyView");  
    }  
    @Override  
    public void onDestroy() {  
        // TODO Auto-generated method stub  
        super.onDestroy();  
        Log.i(TAG, "-MyFragment-->>onDestroy");  
    }  
    @Override  
    public void onDetach() {  
        // TODO Auto-generated method stub  
        super.onDetach();  
        Log.i(TAG, "-MyFragment-->>onDetach");  
    }  
}  
```
```java

public class MainActivity extends Activity {  
    private final String TAG = "MainActivity";  
    private FragmentManager manager;  
    private FragmentTransaction transaction;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        Log.i(TAG, "-MainActivity-->>onCreate");  
        manager = getFragmentManager();  
        transaction = manager.beginTransaction();  
        MyFragment myFragment = new MyFragment();  
        transaction.add(R.id.line, myFragment);  
        transaction.commit();  
    }  
    @Override  
    protected void onStart() {  
        // TODO Auto-generated method stub  
        super.onStart();  
        Log.i(TAG, "-MainActivity-->>onStart");  
    }  
    @Override  
    protected void onResume() {  
        // TODO Auto-generated method stub  
        super.onResume();  
        Log.i(TAG, "-MainActivity-->>onResume");  
    }  
    @Override  
    protected void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
        Log.i(TAG, "-MainActivity-->>onPause");  
    }  
    @Override  
    protected void onStop() {  
        // TODO Auto-generated method stub  
        super.onStop();  
        Log.i(TAG, "-MainActivity-->>onStop");  
    }  
    @Override  
    protected void onDestroy() {  
        // TODO Auto-generated method stub  
        super.onDestroy();  
        Log.i(TAG, "-MainActivity-->>onDestroy");  
    }  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
}  
```
### 三、深入：
#### ①Fragment管理与Fragment事务：
Activity管理Fragment主要依靠FragmentManger：

1.使用findFragmentById()或findFragmentByTag()方法来获取指定Fragment。

2.调用popBackStack()方法将Fragment从后台栈中弹出(模拟用户按下BACK按键)。

3.调用addOnBackStackChangeListener()注册一个监听器，用于监听后台栈的变化。

下面通过一个实例进行说明：

Fragment1：
```java

public class Fragment1 extends Fragment {  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.f1, null);  
        return view;  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
  
}  
```
Fragment2：
```java

public class Fragment2 extends Fragment {  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.f2, null);  
        return view;  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
  
}  
```
Fragment3：
```java
public class Fragment3 extends Fragment {  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.f3, null);  
        return view;  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
}  


```
MainActivity：
```java
public class MainActivity extends Activity implements OnClickListener {  
    private Button button1;  
    private Button button2;  
    private Button button3;  
    private FragmentManager manager;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button1 = (Button) this.findViewById(R.id.button1);  
        button2 = (Button) this.findViewById(R.id.button2);  
        button3 = (Button) this.findViewById(R.id.button3);  
        button1.setOnClickListener(this);  
        button2.setOnClickListener(this);  
        button3.setOnClickListener(this);  
        manager = getFragmentManager();  
  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
    @Override  
    public void onClick(View v) {  
        // TODO Auto-generated method stub  
        FragmentTransaction transaction = manager.beginTransaction();  
        switch (v.getId()) {  
        case R.id.button1:  
  
            Fragment1 fragment1 = new Fragment1();  
            // 加入Fragment回退栈的标记  
            transaction.replace(R.id.main, fragment1, "fragment1");  
            transaction.addToBackStack("fragment1");  
            break;  
  
        case R.id.button2:  
            Fragment2 fragment2 = new Fragment2();  
            transaction.replace(R.id.main, fragment2, "fragment2");  
            transaction.addToBackStack("fragment2");  
  
            break;  
        case R.id.button3:  
            Fragment3 fragment3 = new Fragment3();  
            transaction.replace(R.id.main, fragment3, "fragment3");  
            transaction.addToBackStack("fragment3");  
            break;  
        }  
        transaction.commit();  
    }  
  
    @Override  
    public boolean onKeyDown(int keyCode, KeyEvent event) {  
        // TODO Auto-generated method stub  
        if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {  
            AlertDialog.Builder builder = new AlertDialog.Builder(this);  
            builder.setTitle("提示");  
            builder.setMessage("再按一次就退出系统");  
            builder.setPositiveButton("确定",  
                    new DialogInterface.OnClickListener() {  
                        @Override  
                        public void onClick(DialogInterface arg0, int arg1) {  
                            // TODO Auto-generated method stub  
                            finish();  
                            // System.exit(code);  
                        }  
                    });  
            builder.setNegativeButton("取消",  
                    new DialogInterface.OnClickListener() {  
                        @Override  
                        public void onClick(DialogInterface arg0, int arg1) {  
                            // TODO Auto-generated method stub  
  
                        }  
                    });  
            builder.show();  
            return true;  
        }  
        return super.onKeyDown(keyCode, event);  
    }  
  
}  

```
activity_main.xml：
```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="horizontal"  
    tools:context=".MainActivity" >  
  
    <LinearLayout  
        android:layout_width="101dp"  
        android:layout_height="match_parent"  
        android:background="#CCCCCC"  
        android:orientation="vertical" >  
  
        <Button  
            android:id="@+id/button1"  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content"  
            android:text="界面一" />  
  
        <Button  
            android:id="@+id/button2"  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content"  
            android:text="界面二" />  
  
        <Button  
            android:id="@+id/button3"  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content"  
            android:text="界面三" />  
  
    </LinearLayout>  
  
    <LinearLayout  
        android:id="@+id/main"  
        android:layout_width="0dp"  
        android:layout_height="match_parent"  
        android:layout_weight="1"  
        android:orientation="vertical" >  
    </LinearLayout>  
  
</LinearLayout>  
```
#### ②Fragment之间的通信：
```java
public class Fragment2 extends Fragment {  
    private Button button;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.two, null);  
        button = (Button) view.findViewById(R.id.button2);  
        button.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View arg0) {  
                // TODO Auto-generated method stub  
                Fragment1 fragment1 = (Fragment1) getFragmentManager()  
                        .findFragmentByTag("fragment1");  
                EditText editText = (EditText) fragment1.getView()  
                        .findViewById(R.id.editText1);  
                Toast.makeText(getActivity(),  
                        "--one->>" + editText.getText().toString(), 1).show();  
            }  
        });  
        return view;  
    }  
}  
```
```java
public class Fragment1 extends Fragment {  
    private Button button;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.one, null);  
        button = (Button) view.findViewById(R.id.button1);  
        button.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View arg0) {  
                // TODO Auto-generated method stub  
                // 获得第二个Fragment的输入框的值  
                Fragment2 fragment2 = (Fragment2) getFragmentManager()  
                        .findFragmentByTag("fragment2");  
                EditText editText = (EditText) fragment2.getView()  
                        .findViewById(R.id.editText2);  
                Toast.makeText(getActivity(),  
                        "--two->>" + editText.getText().toString(), 1).show();  
            }  
        });  
        return view;  
    }  
}  
```
```java
public class MainActivity extends Activity {  
    private FragmentManager manager;  
    private FragmentTransaction transaction;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        manager = getFragmentManager();  
        transaction = manager.beginTransaction();  
        Fragment1 fragment1 = new Fragment1();  
        Fragment2 fragment2 = new Fragment2();  
        transaction.add(R.id.one, fragment1, "fragment1");  
        transaction.add(R.id.two, fragment2, "fragment2");  
        transaction.commit();  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
}  
```
#### ③ListFragment的使用：

leftFragment：
```java
public class LeftFragment extends ListFragment {  
    private ArrayAdapter<String> adapter;  
    private FragmentManager manager;  
    private FragmentTransaction transaction;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
        adapter = new ArrayAdapter<String>(getActivity(),  
                android.R.layout.simple_list_item_1, getData());  
        manager = getFragmentManager();  
    }  
  
    public List<String> getData() {  
        List<String> list = new ArrayList<String>();  
        for (int i = 0; i < 30; i++) {  
            list.add("文章" + i);// 模拟文章的标题  
        }  
        return list;  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.left, null);  
        setListAdapter(adapter);  
        adapter.notifyDataSetChanged();  
        return view;  
    }  
  
    // 完成用户的点击行为  
    @Override  
    public void onListItemClick(ListView l, View v, int position, long id) {  
        // TODO Auto-generated method stub  
        super.onListItemClick(l, v, position, id);  
        String item = adapter.getItem(position);  
        //  
        RightFragment fragment = new RightFragment();  
        transaction = manager.beginTransaction();  
        transaction.replace(R.id.right, fragment, "fragment");  
        transaction.addToBackStack("fragment");  
        Bundle bundle = new Bundle();  
        bundle.putString("item", item);  
        fragment.setArguments(bundle);  
        transaction.commit();  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
}  
```
rightFragment：
```java
public class RightFragment extends Fragment {  
  
    public RightFragment() {  
        // TODO Auto-generated constructor stub  
    }  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        View view = inflater.inflate(R.layout.right, null);  
        TextView textView = (TextView) view.findViewById(R.id.textView1);  
        Bundle bundle = getArguments();  
        if (bundle != null) {  
            String item = bundle.getString("item");  
            textView.setText(item);  
        }  
        // String item = getArguments().getString("item");  
        // 从左边的Fragment传递过来的值  
  
        return view;  
    }  
  
    @Override  
    public void onPause() {  
        // TODO Auto-generated method stub  
        super.onPause();  
    }  
}  
```
MainActivity：
```java
public class MainActivity extends Activity {  
    private FragmentManager manager;  
    private FragmentTransaction transaction;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        manager = getFragmentManager();  
        transaction = manager.beginTransaction();  
        LeftFragment leftFragment = new LeftFragment();  
        transaction.add(R.id.left, leftFragment, "leftFragment");  
        // 不能在MainActivity直接加载右边的Fragment  
        // RightFragment rightFragment = new RightFragment();  
        // transaction.add(R.id.right, rightFragment, "rightFragment");  
        transaction.commit();  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
}  

```
开发ListFragment的子类，无须重写onCreateView()方法，与ListActivity相似，只需调用ListFragment的setAdapter()方法为该Fragment设置Adapter即可。

### 四、总结：
掌握Fragment开发界面、添加Fragment到Activity、Fragment与Activity通信。了解Fragment的生命周期。