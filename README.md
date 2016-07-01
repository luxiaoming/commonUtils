# commonUtils
android常用接口

需要交流，联系微信：code_gg_boy
更多精彩，时时关注微信公众号code_gg_home

![一个显示图](https://github.com/luxiaoming/dagger2Demo/raw/master/images/0.jpg)

#dip转px
```java
    public int convertDipOrPx(int dip) {
        float scale = MarketApplication.getMarketApplicationContext()
                .getResources().getDisplayMetrics().density;
        return (int) (dip * scale + 0.5f * (dip >= 0 ? 1 : -1));
    }
 ```
#获取当前窗体,并添加自定义view:
```java
    getWindowManager()
            .addView(
                     overlay,
                     new WindowManager.LayoutParams(
                        LayoutParams.WRAP_CONTENT,
                        LayoutParams.WRAP_CONTENT,
                        WindowManager.LayoutParams.TYPE_APPLICATION,
                        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                        | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE,
                        PixelFormat.TRANSLUCENT));
                        
```

#自定义fastScrollBar图片样式：
```java
        try {
            Field f = AbsListView.class.getDeclaredField("mFastScroller");
            f.setAccessible(true);
            Object o = f.get(listView);
            f = f.getType().getDeclaredField("mThumbDrawable");
            f.setAccessible(true);
            Drawable drawable = (Drawable) f.get(o);
            drawable = getResources().getDrawable(R.drawable.ic_launcher);
            f.set(o, drawable);
            Toast.makeText(this, f.getType().getName(), 1000).show();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        
```
=网络=

#判断网络是否可用：

```java
/**
     * 网络是否可用
	 * 
	 * @param context
	 * @return
	 */
	public static boolean isNetworkAvailable(Context context) {
		ConnectivityManager mgr = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
		NetworkInfo[] info = mgr.getAllNetworkInfo();
	    if (info != null) {
	    	for (int i = 0; i < info.length; i++) {
	    		if (info[i].getState() == NetworkInfo.State.CONNECTED) {
	    			return true;
	    		}
	    	}
	    }
		return false;
	}
```
方法二：
```java
    /*
     * 判断网络连接是否已开 2012-08-20true 已打开 false 未打开
     */
    public static boolean isConn(Context context) {
        boolean bisConnFlag = false;
        ConnectivityManager conManager = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo network = conManager.getActiveNetworkInfo();
        if (network != null) {
            bisConnFlag = conManager.getActiveNetworkInfo().isAvailable();
        }
        return bisConnFlag;
    }
 ```
#判断是不是Wifi连接

```java
    public static boolean isWifiActive(Context icontext) {
        Context context = icontext.getApplicationContext();
        ConnectivityManager connectivity = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo[] info;
        if (connectivity != null) {
            info = connectivity.getAllNetworkInfo();
            if (info != null) {
                for (int i = 0; i < info.length; i++) {
                    if (info[i].getTypeName().equals("WIFI")
                            && info[i].isConnected()) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
 ```
#判断当前网络类型

```java
/**
	 * 网络方式检查
	 */
	private static int netCheck(Context context) {
		ConnectivityManager conMan = (ConnectivityManager) context
				.getSystemService(Context.CONNECTIVITY_SERVICE);
		State mobile = conMan.getNetworkInfo(ConnectivityManager.TYPE_MOBILE)
				.getState();
		State wifi = conMan.getNetworkInfo(ConnectivityManager.TYPE_WIFI)
				.getState();
		if (wifi.equals(State.CONNECTED)) {
			return DO_WIFI;
		} else if (mobile.equals(State.CONNECTED)) {
			return DO_3G;
		} else {
			return NO_CONNECTION;
		}
	}
```

#获取下载文件的真实名字
```java
    public String getReallyFileName(String url) {
        StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                .detectDiskReads().detectDiskWrites().detectNetwork() // 这里可以替换为detectAll()
                                                                      // 就包括了磁盘读写和网络I/O
                .penaltyLog() // 打印logcat，当然也可以定位到dropbox，通过文件保存相应的log
                .build());
        StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                .detectLeakedSqlLiteObjects() // 探测SQLite数据库操作
                .penaltyLog() // 打印logcat
                .penaltyDeath().build());

        String filename = "";
        URL myURL;
        HttpURLConnection conn = null;
        if (url == null || url.length() < 1) {
            return null;
        }

        try {
            myURL = new URL(url);
            conn = (HttpURLConnection) myURL.openConnection();
            conn.connect();
            conn.getResponseCode();
            URL absUrl = conn.getURL();// 获得真实Url
            // 打印输出服务器Header信息
            // Map<String, List<String>> map = conn.getHeaderFields();
            // for (String str : map.keySet()) {
            // if (str != null) {
            // Log.e("H3c", str + map.get(str));
            // }
            // }
            filename = conn.getHeaderField("Content-Disposition");// 通过Content-Disposition获取文件名，这点跟服务器有关，需要灵活变通
            if (filename == null || filename.length() < 1) {
                filename = URLDecoder.decode(absUrl.getFile(), "UTF-8");
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (conn != null) {
                conn.disconnect();
                conn = null;
            }
        }

        return filename;
    }
```
=图片==========================
#bitmap转Byte数组(微信分享就需要用到)

```java
public byte[] bmpToByteArray(final Bitmap bmp, final boolean needRecycle) {
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        bmp.compress(CompressFormat.PNG, 100, output);
        if (needRecycle) {
            bmp.recycle();
        }

        byte[] result = output.toByteArray();
        try {
            output.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return result;
    }
```
    
#Resources转Bitmap
```java
public Bitmap loadBitmap(Resources res, int id) {
        BitmapFactory.Options opt = new BitmapFactory.Options();
        opt.inPreferredConfig = Bitmap.Config.RGB_565;
        opt.inPurgeable = true;
        opt.inInputShareable = true;

        InputStream is = res.openRawResource(id);// 获取资源图片
        return BitmapFactory.decodeStream(is, null, opt);
    }
```
#保存图片到SD卡
```java
public void saveBitmapToFile(String url, String filePath) {
        File iconFile = new File(filePath);
        if (!iconFile.getParentFile().exists()) {
            iconFile.getParentFile().mkdirs();
        }

        if (iconFile.exists() && iconFile.length() > 0) {
            return;
        }

        FileOutputStream fos = null;
        InputStream is = null;
        try {
            fos = new FileOutputStream(filePath);
            is = new URL(url).openStream();

            int data = is.read();
            while (data != -1) {
                fos.write(data);
                data = is.read();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null) {
                    is.close();
                }
                if (fos != null) {
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 ```
=系统==============================
```java
#根据包名打开一个应用程序

    public boolean openApp(String packageName) {
        PackageInfo pi = null;
        try {
            pi = mPM.getPackageInfo(packageName, 0);
        } catch (NameNotFoundException e) {
            e.printStackTrace();
            return false;
        }

        if (pi == null) {
            return false;
        }

        Intent resolveIntent = new Intent(Intent.ACTION_MAIN, null);
        resolveIntent.addCategory(Intent.CATEGORY_LAUNCHER);
        resolveIntent.setPackage(pi.packageName);

        List<ResolveInfo> apps = mPM.queryIntentActivities(resolveIntent, 0);

        ResolveInfo ri = null;
        try {
            ri = apps.iterator().next();
        } catch (Exception e) {
            return true;
        }
        if (ri != null) {
            String tmpPackageName = ri.activityInfo.packageName;
            String className = ri.activityInfo.name;

            Intent intent = new Intent(Intent.ACTION_MAIN);
            intent.addCategory(Intent.CATEGORY_LAUNCHER);

            ComponentName cn = new ComponentName(tmpPackageName, className);

            intent.setComponent(cn);
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            MarketApplication.getMarketApplicationContext().startActivity(
                    intent);
        } else {
            return false;
        }
        return true;
    }
```
#判断是否APK是否安装过
```java
public boolean checkApkExist(Context context, String packageName) {
        if (packageName == null || "".equals(packageName))
            return false;
        try {
            ApplicationInfo info = context.getPackageManager()
                    .getApplicationInfo(packageName,
                            PackageManager.GET_UNINSTALLED_PACKAGES);
            return true;
        } catch (NameNotFoundException e) {
            return false;
        } catch (NullPointerException e) {
            return false;
        }
    }
    ```
#安装APK
```java
    public void installApk(Context context, String strFileAllName) {
        File file = new File(strFileAllName);
        Intent intent = new Intent();
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setAction(Intent.ACTION_VIEW);
        String type = "application/vnd.android.package-archive";
        intent.setDataAndType(Uri.fromFile(file), type);
        context.startActivity(intent);
    }
    ```
#卸载APK
```java
    public void UninstallApk(Context context, String strPackageName) {
        Uri packageURI = Uri.parse("package:" + strPackageName);
        Intent uninstallIntent = new Intent(Intent.ACTION_DELETE, packageURI);
        context.startActivity(uninstallIntent);
    }
    ```
#判断SD卡是否可用
```java
    public boolean CheckSD() {
        if (android.os.Environment.getExternalStorageState().equals(
                android.os.Environment.MEDIA_MOUNTED)) {
            return true;
        } else {
            return false;
        }
    }
    ```
#创建快捷方式：
```java
    public void createShortCut(Context contxt) {
        // if (isInstallShortcut()) {// 如果已经创建了一次就不会再创建了
        // return;
        // }

        Intent sIntent = new Intent(Intent.ACTION_MAIN);
        sIntent.addCategory(Intent.CATEGORY_LAUNCHER);// 加入action,和category之后，程序卸载的时候才会主动将该快捷方式也卸载
        sIntent.setClass(contxt, Login.class);

        Intent installer = new Intent();
        installer.putExtra("duplicate", false);
        installer.putExtra("android.intent.extra.shortcut.INTENT", sIntent);
        installer.putExtra("android.intent.extra.shortcut.NAME", "名字");
        installer.putExtra("android.intent.extra.shortcut.ICON_RESOURCE",
                Intent.ShortcutIconResource
                        .fromContext(contxt, R.drawable.icon));
        installer.setAction("com.android.launcher.action.INSTALL_SHORTCUT");
        contxt.sendBroadcast(installer);
    }
    ```
#判断快捷方式是否创建：
```java
private boolean isInstallShortcut() {
        boolean isInstallShortcut = false;
        final ContentResolver cr = MarketApplication
                .getMarketApplicationContext().getContentResolver();
        String AUTHORITY = "com.android.launcher.settings";
        Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY
                + "/favorites?notify=true");

        Cursor c = cr.query(CONTENT_URI,
                new String[] { "title", "iconResource" }, "title=?",
                new String[] { "名字" }, null);
        if (c != null && c.getCount() > 0) {
            isInstallShortcut = true;
        }

        if (c != null) {
            c.close();
        }

        if (isInstallShortcut) {
            return isInstallShortcut;
        }

        AUTHORITY = "com.android.launcher2.settings";
        CONTENT_URI = Uri.parse("content://" + AUTHORITY
                + "/favorites?notify=true");
        c = cr.query(CONTENT_URI, new String[] { "title", "iconResource" },
                "title=?", new String[] { "名字" }, null);
        if (c != null && c.getCount() > 0) {
            isInstallShortcut = true;
        }

        if (c != null) {
            c.close();
        }

        AUTHORITY = "com.baidu.launcher";
        CONTENT_URI = Uri.parse("content://" + AUTHORITY
                + "/favorites?notify=true");
        c = cr.query(CONTENT_URI, new String[] { "title", "iconResource" },
                "title=?", new String[] { "名字" }, null);
        if (c != null && c.getCount() > 0) {
            isInstallShortcut = true;
        }

        if (c != null) {
            c.close();
        }

        return isInstallShortcut;
    }
    ```
#过滤特殊字符：
```java
    private String StringFilter(String str) throws PatternSyntaxException {
        // 只允许字母和数字
        // String regEx = "[^a-zA-Z0-9]";
        // 清除掉所有特殊字符
        String regEx = "[`~!@#$%^&*()+=|{}':;',//[//].<>/?~！@#￥%……&*（）——+|{}【】‘；：”“’。，、？]";
        Pattern p = Pattern.compile(regEx);
        Matcher m = p.matcher(str);
        return m.replaceAll("").trim();
    }
    ```
#执行shell语句：
```java
    public int execRootCmdSilent(String cmd) {
        int result = -1;
        DataOutputStream dos = null;

        try {
            Process p = Runtime.getRuntime().exec("su");
            dos = new DataOutputStream(p.getOutputStream());
            dos.writeBytes(cmd + "\n");
            dos.flush();
            dos.writeBytes("exit\n");
            dos.flush();
            p.waitFor();
            result = p.exitValue();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (dos != null) {
                try {
                    dos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return result;
    }
    ```
#获得文件MD5值：
```java
    public String getFileMD5(File file) {
        if (!file.isFile()) {
            return null;
        }

        MessageDigest digest = null;
        FileInputStream in = null;
        byte buffer[] = new byte[1024];
        int len;
        try {
            digest = MessageDigest.getInstance("MD5");
            in = new FileInputStream(file);
            while ((len = in.read(buffer, 0, 1024)) != -1) {
                digest.update(buffer, 0, len);
            }
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        BigInteger bigInt = new BigInteger(1, digest.digest());
        return bigInt.toString(16);
    }
    ```
