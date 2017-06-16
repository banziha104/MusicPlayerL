# MusicPlayer

---
## PermissionControl
---

```java
package com.veryworks.android.musicplayer;

import android.Manifest;
import android.annotation.TargetApi;
import android.app.Activity;
import android.content.pm.PackageManager;
import android.os.Build;
import android.support.annotation.NonNull;
import android.widget.Toast;

/**
 * Created by pc on 6/15/2017.
 */

public class PermissionControl {
    public static final int REQ_FLAG = 1000232412;
    private static String permissions[] = {
            Manifest.permission.READ_EXTERNAL_STORAGE
    };

    public static void checkVersion(Activity activity){
        // 마시멜로 이상버전에서만 런타임 권한체크
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            checkPermission(activity);
        }else {
            callInit(activity);
        }
    }

    @TargetApi(Build.VERSION_CODES.M)
    public static void checkPermission(Activity activity){
        //1 권한체크 - 특정권한이 있는지 시스템에 물어본다
        boolean denied = false;
        for(String perm : permissions){
            if(activity.checkSelfPermission(perm) != PackageManager.PERMISSION_GRANTED){
                denied = true;
                break;
            }
        }
        if(denied){
            // 2. 권한이 없으면 사용자에 권한을 달라고 요청
            activity.requestPermissions(permissions ,REQ_FLAG);
        }else{
            callInit(activity);
        }
    }

    public static void onResult(Activity activity, int requestCode, @NonNull int[] grantResults){
        if(requestCode == REQ_FLAG){
            boolean granted = true;
            for(int grant : grantResults){
                if(grant != PackageManager.PERMISSION_GRANTED){
                    granted = false;
                    break;
                }
            }
            // 3.1 사용자가 승인을 했음
            if(granted){
                callInit(activity);
            }else{
                Toast.makeText(activity, "권한요청을 승인하셔야 앱을 사용할 수 있습니다.", Toast.LENGTH_SHORT).show();
                //activity.finish();
            }
        }
    }


    private static void callInit(Activity activity){
        if(activity instanceof CallBack)
            ((CallBack)activity).init();
        else
            throw new RuntimeException("must implement this.CallBack");
    }

    public interface CallBack {
        public void init();
    }
}
```

---
## checkVersion
---
```java
public static void checkVersion(Activity activity){ //액티비티를 넘겨받음
        // 마시멜로 이상버전에서만 런타임 권한체크
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) { 
            checkPermission(activity);
        }else {
            callInit(activity);
        }
    }
```
 액티비티를 인자로 넘겨받아 해당 액티비티에서 버전 체크를 하는 함수
 ---
##
---
```java
 public static void onResult(Activity activity, int requestCode, @NonNull int[] grantResults){
        if(requestCode == REQ_FLAG){
            boolean granted = true;
            for(int grant : grantResults){
                if(grant != PackageManager.PERMISSION_GRANTED){
                    granted = false;
                    break;
                }
            }
            // 3.1 사용자가 승인을 했음
            if(granted){
                callInit(activity);
            }else{
                Toast.makeText(activity, "권한요청을 승인하셔야 앱을 사용할 수 있습니다.", Toast.LENGTH_SHORT).show();
                //activity.finish();
            }
        }
    }
```
요청한 권한이 있는지 확인하는 메소드

 ---
## callInit
---
```java
private static void callInit(Activity activity){
        if(activity instanceof CallBack) //CallBack으로 형변환이 가능한지 체크
            ((CallBack)activity).init(); //가능하다면 형변화후 해당 액티비티의 init 함수실행
        else
            throw new RuntimeException("must implement this.CallBack");
    }
```
인터페이스로 캐스팅 하는 이유는 해당 init이 여러곳에서 구현되었다면 해당 객체에 구현된
init만을 사용할 수 있도록 하기 위함.

---
## interface CallBack
---
```java
 public interface CallBack {
        public void init();
    }
```
CallBack 인터페이스, init함수를 이용하여 초기화시키기 위함.

---
## MainActivity
---
```java
package com.veryworks.android.musicplayer;

import android.os.Bundle;
import android.support.annotation.NonNull;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.widget.FrameLayout;

public class MainActivity extends AppCompatActivity
        implements ListFragment.OnListFragmentInteractionListener
                    , PermissionControl.CallBack{

    FrameLayout layout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        PermissionControl.checkVersion(this);
    }



    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionControl.onResult(this, requestCode, grantResults);
    }

    @Override
    public void init(){
        setViews();
        setFragment(ListFragment.newInstance(1)); // 목록 프래그먼트
    }

    private void setViews(){
        layout = (FrameLayout) findViewById(R.id.layout);
    }

    private void setFragment(Fragment fragment){
        FragmentManager manager = getSupportFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.add(R.id.layout, fragment);
        transaction.commit();
    }

    private void addFragment(Fragment fragment){
        FragmentManager manager = getSupportFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.add(R.id.layout, fragment);
        transaction.addToBackStack(null);
        transaction.commit();
    }

    // Fragment 를 통해 Adapter 까지 interface 를 전달하고
    // Adapter 에서 interface 를 직접호출해서 사용한다.
    @Override
    public void goDetailInteraction(int position) {
        addFragment(DetailFragment.newInstance(position));
    }
}

```

---
## onCreate
---
```java
protected void onCreate(Bundle savedInstanceState) {
        protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        PermissionControl.checkVersion(this); //권한 체크 및 요청실행
    }
 }
```
onCreate에 PermissionControl의 checkVersion 메소드를 이용해 권한체크를 함

---
## onRequestPermissionsResult
---
```java
@Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionControl.onResult(this, requestCode, grantResults);//onResult에서 결과처리
    }
```
onRequestPermissionsResult는 콜백메소드로 유저의 반응후에 자동으로 호출되는 메소드.

---
## init()과 view 부분
---
```java

    @Override
    public void init(){
        setViews();
        setFragment(ListFragment.newInstance(1)); // 목록 프래그먼트
    }

    private void setViews(){
        layout = (FrameLayout) findViewById(R.id.layout);
    }

    private void setFragment(Fragment fragment){
        FragmentManager manager = getSupportFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.add(R.id.layout, fragment);
        transaction.commit();
    }

    private void addFragment(Fragment fragment){
        FragmentManager manager = getSupportFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.add(R.id.layout, fragment);
        transaction.addToBackStack(null);
        transaction.commit();
    }

    // Fragment 를 통해 Adapter 까지 interface 를 전달하고
    // Adapter 에서 interface 를 직접호출해서 사용한다.
    @Override
    public void goDetailInteraction(int position) {
        addFragment(DetailFragment.newInstance(position));
    }
}
```
결과 처리 후, init에서 View와 관련된 메서드를 호출하여 처리.


