# idocr
身份证识别-百度

	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
  
  
  dependencies {
	        implementation 'com.github.lib093:idocr:1.0.0'
	}
	
	
百度云平台地址：https://console.bce.baidu.com/
下载license 放入assets文件夹中

1.调用前进行初始化
------------------------
 OCR.getInstance(Utils.getContext()).initAccessToken(new OnResultListener<AccessToken>() {
                @Override
                public void onResult(AccessToken accessToken) {
                    String token = accessToken.getAccessToken();
                    hasGotToken = true;

                    //  初始化本地质量控制模型,释放代码在onDestory中
                    //  调用身份证扫描必须加上 intent.putExtra(CameraActivity.KEY_NATIVE_MANUAL, true); 关闭自动初始化和释放本地模型
                    CameraNativeHelper.init(getApplication(), OCR.getInstance(getApplication()).getLicense(),
                            new CameraNativeHelper.CameraNativeInitCallback() {
                                @Override
                                public void onError(int errorCode, Throwable e) {
                                    String msg;
                                    switch (errorCode) {
                                        case CameraView.NATIVE_SOLOAD_FAIL:
                                            msg = "加载so失败，请确保apk中存在ui部分的so";
                                            break;
                                        case CameraView.NATIVE_AUTH_FAIL:
                                            msg = "授权本地质量控制token获取失败";
                                            break;
                                        case CameraView.NATIVE_INIT_FAIL:
                                            msg = "本地质量控制";
                                            break;
                                        default:
                                            msg = String.valueOf(errorCode);
                                    }
                                    Log.d("lib",msg);
                                }
                            });
                }

                @Override
                public void onError(OCRError error) {
                    error.printStackTrace();
                    Log.d("lib","自定义文件路径licence方式获取token失败"+ error.getMessage());
                }
            }, "aip.license", getApplication());
	    
	    

2.调用页面
-------------
                Intent intent = new Intent(LoginActivity.this, CameraActivity.class);
                intent.putExtra(CameraActivity.KEY_OUTPUT_FILE_PATH,
                        new File(LoginActivity.this.getFilesDir(), "pic.jpg").getAbsolutePath());
                intent.putExtra(CameraActivity.KEY_NATIVE_ENABLE,
                        true);
                // KEY_NATIVE_MANUAL设置了之后CameraActivity中不再自动初始化和释放模型
                // 请手动使用CameraNativeHelper初始化和释放模型
                // 推荐这样做，可以避免一些activity切换导致的不必要的异常
                intent.putExtra(CameraActivity.KEY_NATIVE_MANUAL,
                        true);
                intent.putExtra(CameraActivity.KEY_CONTENT_TYPE, CameraActivity.CONTENT_TYPE_ID_CARD_FRONT);
                startActivityForResult(intent, 100);
3.获取结果
---------------

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 100 && resultCode == Activity.RESULT_OK) {
            if (data != null) {
                String contentType = data.getStringExtra(CameraActivity.KEY_CONTENT_TYPE);
                if (!TextUtils.isEmpty(contentType)) {
                    if (CameraActivity.CONTENT_TYPE_ID_CARD_FRONT.equals(contentType)) {
                        Toast.makeText(LoginActivity.this, data.getStringExtra("result"), Toast.LENGTH_LONG).show();
                    } else if (CameraActivity.CONTENT_TYPE_ID_CARD_BACK.equals(contentType)) {
                    }
                }
            }
        }
    }
4.注销释放
-----------
    @Override
    public void onDestroy() {
        // 释放本地质量控制模型
        CameraNativeHelper.release();
        super.onDestroy();
        // 释放内存资源
        OCR.getInstance(this).release();
    }
