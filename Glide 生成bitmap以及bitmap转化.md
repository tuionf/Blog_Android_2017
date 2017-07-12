---
title: Glide 生成bitmap以及bitmap转化
tags: 小书匠,Android,Glide
grammar_cjkRuby: true
---

# Glide 
##  1.图片下载好之后获取Bitmap

``` java
Glide.with(mContext).load(url).asBitmap().into(new SimpleTarget<Bitmap>() {  
                @Override  
                public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {  
                    image.setImageBitmap(resource);  
                }  
            });


```


## 2.获取图片缓存path

``` java
FutureTarget<File> future = Glide.with(mContext)  
                    .load("url")  
                    .downloadOnly(500, 500);  
            try {  
                File cacheFile = future.get();  
                String path = cacheFile.getAbsolutePath();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            } catch (ExecutionException e) {  
                e.printStackTrace();  
            }

```


## 3.通过url获取bitmap

``` java
Bitmap myBitmap = Glide.with(applicationContext)  
    .load(yourUrl)  
    .asBitmap() //必须  
    .centerCrop()  
    .into(500, 500)  
    .get()

```

# Bitmap转化

## Bitmap 转换成 Drawable

``` java
Drawable drawable = new BitmapDrawable(bmp);
```

## Drawable 转换成 Bitmap

**方法一**

通过 BitmapFactory 中的 decodeResource 方法，将资源文件中的 R.drawable.ic_drawable 转化成Bitmap

``` java
Resources res = getResources();
Bitmap    bmp = BitmapFactory.decodeResource(res, R.drawable.ic_drawable);
```

方法二

将 Drable 对象先转化成 BitmapDrawable ，然后调用 getBitmap 方法 获取

``` java

Drawable drawable =gerResource().getDrawable(R.drawable.ic_drawable);//获取drawable
BitmapDrawable bd = (BitmapDrawable) drawable;
Bitmap bm         = bd.getBitmap();
```
## Bitmap 转换成 byte[]

``` java
public static byte[] getBytes(Bitmap bitmap){

        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        bitmap.compress(Bitmap.CompressFormat.PNG, 100, baos);

        return baos.toByteArray();   

    }
```

## byte[] 转化成 Bitmap

    public static Bitmap Bytes2Bimap(byte[] b) {

        if (b.length != 0) {
            return BitmapFactory.decodeByteArray(b, 0, b.length);
        } else {
            return null;
        }

    }
	

参考 ：
http://www.jianshu.com/p/45ef32825c69
http://blog.csdn.net/l_lhc/article/details/50923372#t1
