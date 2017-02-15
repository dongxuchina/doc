默认情况下，Android 的 WebView 是不支持文件上传的，可以通过重写 WebChromeClient 来实现。

**MyWebChromeClient**

```
	public class MyWebChromeClient extends WebChromeClient {
	
	    // For Android 4.1
	    public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
	         mUploadMsg = uploadMsg;
            Intent intent = createDefaultOpenableIntent();
            activity.startActivityForResult(intent, 1000);
	    }
	    
		//Android 5.0+   
	    @Override
	    @SuppressLint("NewApi")
	    public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
	         mUploadMsgArray = uploadMsgArray;
            Intent intent = createDefaultOpenableIntent();
            activity.startActivityForResult(intent, 1000);
            return true;
	    }
	}

	//支持拍照，录像，录音及文件选择（照片，视频）
    private Intent createDefaultOpenableIntent() {
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.setData(android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/* video/*"); 
        Intent chooser = createChooserIntent(createCameraIntent(), createSoundRecorderIntent(), createVideoIntent());
        chooser.putExtra(Intent.EXTRA_INTENT, i);
        return chooser;
    }

    private Intent createSoundRecorderIntent() {
        return new Intent(MediaStore.Audio.Media.RECORD_SOUND_ACTION);
    }

    private Intent createChooserIntent(Intent... intents) {
        Intent chooser = new Intent(Intent.ACTION_CHOOSER);
        chooser.putExtra(Intent.EXTRA_INITIAL_INTENTS, intents);
        chooser.putExtra(Intent.EXTRA_TITLE, "File Chooser");
        return chooser;
    }

    private Intent createCameraIntent() {
        Intent cameraIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        File externalDataDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM);
        File cameraDataDir = new File(externalDataDir.getAbsolutePath() + File.separator + "browser-photos");
        cameraDataDir.mkdirs();
        mCameraFilePath = cameraDataDir.getAbsolutePath() + File.separator + System.currentTimeMillis() + ".jpg";
        cameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(new File(mCameraFilePath)));
        return cameraIntent;
    }

    private Intent createVideoIntent() {
        Intent cameraIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
        File externalDataDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM);
        File cameraDataDir = new File(externalDataDir.getAbsolutePath() + File.separator + "browser-videos");
        cameraDataDir.mkdirs();
        mVideoFilePath = cameraDataDir.getAbsolutePath() + File.separator + System.currentTimeMillis() + ".mp4";
        cameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(new File(mVideoFilePath)));
        return cameraIntent;
    }
    
```
openFileChooser 的 ValueCallback 用于我们在选择完文件后，接收文件回调到网页内处理。Android5.0 之后系统提供了 onShowFileChooser 来完成文件选择，仍然有 ValueCallback 与之前不同的是类型变成了数组。

**处理选择后的文件**

```
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

		 //此处通过ValueCallback来判断是否走哪个处理
        if (mUploadMsgArray != null){ //Android 5.0+
            onActivityResultUpload(requestCode, resultCode, data);
        }else if (mUploadMsg != null) {
            if (resultCode == Activity.RESULT_OK) {
                File cameraFile = new File(mCameraFilePath);
                if (cameraFile.exists()) {
                    mUploadMsg.onReceiveValue(Uri.fromFile(cameraFile));
                    mUploadMsg = null;
                    return;
                }
                File videoFile = new File(mVideoFilePath);
                if(videoFile.exists()){
                    mUploadMsg.onReceiveValue(Uri.fromFile(videoFile));
                    mUploadMsg = null;
                    return;
                }
                
                Uri result = data == null || resultCode != Activity.RESULT_OK ? null : data.getData();
                Uri fileUri = getFileUri(result);
                if (fileUri != null) {
                    mUploadMsg.onReceiveValue(fileUri);
                    mUploadMsg = null;
                } else {
                    mUploadMsg.onReceiveValue(data.getData());
                    mUploadMsg = null;
                }
            } else {
                mUploadMsg.onReceiveValue(null);
                mUploadMsg = null;
            }
        }

    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void onActivityResultUpload(int requestCode, int resultCode, Intent intent) {
        Uri[] results = null;
        if (resultCode == Activity.RESULT_OK) {

            //拍照和录像返回的intent＝null,所以需要通过文件是否存在来判断
            File cameraFile = new File(mCameraFilePath);
            if (cameraFile.exists()) {
                mUploadMsgArray.onReceiveValue(new Uri[]{Uri.fromFile(cameraFile)});
                mUploadMsgArray = null;
                return;
            }

            File videoFile = new File(mVideoFilePath);
            if(videoFile.exists()){
                mUploadMsgArray.onReceiveValue(new Uri[]{Uri.fromFile(videoFile)});
                mUploadMsgArray = null;
                return;
            }

            if (intent != null) {
                String dataString = intent.getDataString();
                ClipData clipData = intent.getClipData();
                if (clipData != null) {
                    results = new Uri[clipData.getItemCount()];
                    for (int i = 0; i < clipData.getItemCount(); i++) {
                        ClipData.Item item = clipData.getItemAt(i);
                        results[i] = item.getUri();
                    }
                }
                if (dataString != null) {
                    results = new Uri[]{Uri.parse(dataString)};
                }
            }
        }
        mUploadMsgArray.onReceiveValue(results);
        mUploadMsgArray = null;
    }
```
即使获取的结果为 null，也要传给web，即直接调用 mUploadMsg.onReceiveValue(null),否则网页会阻塞。

选择文件使用系统提供的组件或者其他支持的 app，返回的 uri 有的是文件的 uri，有的是 contentprovider 的 uri，所以我们需要统一转成文件的 uri。

```
    protected Uri getFileUri(Uri uri) {
        String[] proj = { MediaStore.Images.Media.DATA };
        Cursor cursor = activity.getContentResolver().query(uri, proj, null, null, null);
        if (cursor == null) {
            return null;
        }
        int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
        cursor.moveToFirst();
        String path = cursor.getString(column_index);
        cursor.close();
        if (!TextUtils.isEmpty(path)) {
            File file = new File(path);
            if (file.exists()) {
                return Uri.fromFile(file);
            }
        }
        return null;
    }
    
```