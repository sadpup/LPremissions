package com.example.yons.myapplication;

import android.Manifest;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Build;
import android.provider.Settings;
import android.support.annotation.NonNull;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.widget.Toast;
import com.elex.chatservice.model.LanguageManager;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by yons on 2017/12/20.
 */
public class LPermissions {

    public static final int LPermissions_REQUEST_CODE = 1111;


    String[] permissions = new String[]{};//Manifest.permission.CALL_PHONE, Manifest.permission.WRITE_EXTERNAL_STORAGE};
    List<String> mPermissionList = new ArrayList<>();//未授予权限列表

    AppCompatActivity activity = null;
    LPermissionsCallback lPermissionsCallback = null;

    public LPermissions(AppCompatActivity activity, String[] permisssions, LPermissionsCallback lPermissionsCallback){
        this.permissions = permisssions;
        this.activity = activity;
        this.lPermissionsCallback = lPermissionsCallback;
        checkPermissions();
        requestPermissions();
    }


    private void checkPermissions() {
        if(Build.VERSION.SDK_INT < 23)
            return;
        /**
         * 判断哪些权限未授予
         */
        mPermissionList.clear();
        for (int i = 0; i < permissions.length; i++) {
            if (ContextCompat.checkSelfPermission(activity, permissions[i]) != PackageManager.PERMISSION_GRANTED) {
                mPermissionList.add(permissions[i]);
            }
        }
    }

    private void requestPermissions(){
        if(Build.VERSION.SDK_INT < 23)
            return;
        /**
         * 判断是否为空
         */
        if (mPermissionList.isEmpty()) {//未授予的权限为空，表示都授予了
            lPermissionsCallback.gotPermissionsAndDo();
        } else {//请求权限方法
            String[] permissions = mPermissionList.toArray(new String[mPermissionList.size()]);//将List转为数组
            ActivityCompat.requestPermissions(activity, permissions, LPermissions_REQUEST_CODE);
        }
    }


    public void onRequestLPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode) {
            case LPermissions_REQUEST_CODE:
                for (int i = 0; i < grantResults.length; i++) {
                    if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                        //判断是否勾选禁止后不再询问
                        boolean showRequestPermission = ActivityCompat.shouldShowRequestPermissionRationale(activity, permissions[i]);
                        if (showRequestPermission) {//
//                            ActivityCompat.requestPermissions(this, new String[]{permissions[i]}, 1);//重新申请权限
                        } else {
                            showPermissionDialog();
                        }
                        return;
                    }
                }
                lPermissionsCallback.gotPermissionsAndDo();
                break;
            default:
                break;
        }
    }

    /**
     * 不再提示权限 时的展示对话框
     */
    AlertDialog mPermissionDialog;
    private void showPermissionDialog() {
        if (mPermissionDialog == null) {
            mPermissionDialog = new AlertDialog.Builder(activity)
                    .setMessage(LanguageManager.getLangByKey("87003000"));//"已禁用权限，请手动授予")
                    .setPositiveButton(LanguageManager.getLangByKey("101218"), new DialogInterface.OnClickListener() {//"设置"
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            cancelPermissionDialog();
                            Uri packageURI = Uri.parse("package:" + activity.getPackageName());
                            Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS, packageURI);
                            activity.startActivity(intent);
                        }
                    })
                    .setNegativeButton(LanguageManager.getLangByKey("82001154"), new DialogInterface.OnClickListener() {//"取消"
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            cancelPermissionDialog();
                        }
                    })
                    .create();
        }
        mPermissionDialog.show();
    }

    private void cancelPermissionDialog() {
        mPermissionDialog.cancel();
    }


    interface LPermissionsCallback{
        public void gotPermissionsAndDo();
    }
}
