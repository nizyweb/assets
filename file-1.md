# Activity life Cycle

The following example will download an image from the Internet in a thread and displays a dialog until the download is done. We will make sure that the thread is preserved even if the activity is restarted and that the dialog is correctly displayed and closed.
For this example create the Android project “de.vogella.android.threadslifecycle” and the Activity “ThreadsLifecycleActivity”. Also add the permission to use the Internet to your app.

1. AndroidManifest

You should have the following AndroidManifest.xml file.

<?xml version=“1.0” encoding=“utf-8”?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package=“de.vogella.android.threadslifecycle”
android:versionCode=“1”
android:versionName=“1.0” >
<uses-sdk android:minSdkVersion=“10” />
<uses-permission android:name=“android.permission.INTERNET” >
</uses-permission>
<application
android:icon="@drawable/icon"
android:label="@string/app_name" >
<activity
android:name=".ThreadsLifecycleActivity"
android:label="@string/app_name" >
<intent-filter>
<action android:name=“android.intent.action.MAIN” />
<category android:name=“android.intent.category.LAUNCHER” />
</intent-filter>
</activity>
</application>
</manifest>

    main layout

Change the layout main.xml to the following.

<?xml version=“1.0” encoding=“utf-8”?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width=“match_parent”
android:layout_height=“match_parent”
android:orientation=“vertical” >

<span style=“color: #008000; font-weight: bold”><LinearLayout</span>
<span style=“color: #7D9029”>android:id=</span><span style=“color: #BA2121”>"@+id/linearLayout1"</span>
<span style=“color: #7D9029”>android:layout_width=</span><span style=“color: #BA2121”>“match_parent”</span>
<span style=“color: #7D9029”>android:layout_height=</span><span style=“color: #BA2121”>“wrap_content”</span> <span style=“color: #008000; font-weight: bold”>></span>

<span style="color: #008000; font-weight: bold">&lt;Button</span>
    <span style="color: #7D9029">android:layout_width=</span><span style="color: #BA2121">&quot;wrap_content&quot;</span>
    <span style="color: #7D9029">android:layout_height=</span><span style="color: #BA2121">&quot;wrap_content&quot;</span>
    <span style="color: #7D9029">android:onClick=</span><span style="color: #BA2121">&quot;downloadPicture&quot;</span>
    <span style="color: #7D9029">android:text=</span><span style="color: #BA2121">&quot;Click to start download&quot;</span> <span style="color: #008000; font-weight: bold">&gt;</span>
<span style="color: #008000; font-weight: bold">&lt;/Button&gt;</span>

<span style="color: #008000; font-weight: bold">&lt;Button</span>
    <span style="color: #7D9029">android:layout_width=</span><span style="color: #BA2121">&quot;wrap_content&quot;</span>
    <span style="color: #7D9029">android:layout_height=</span><span style="color: #BA2121">&quot;wrap_content&quot;</span>
    <span style="color: #7D9029">android:onClick=</span><span style="color: #BA2121">&quot;resetPicture&quot;</span>
    <span style="color: #7D9029">android:text=</span><span style="color: #BA2121">&quot;Reset Picture&quot;</span> <span style="color: #008000; font-weight: bold">&gt;</span>
<span style="color: #008000; font-weight: bold">&lt;/Button&gt;</span>

<span style=“color: #008000; font-weight: bold”></LinearLayout></span>
<span style=“color: #008000; font-weight: bold”><ImageView</span>
<span style=“color: #7D9029”>android:id=</span><span style=“color: #BA2121”>"@+id/imageView1"</span>
<span style=“color: #7D9029”>android:layout_width=</span><span style=“color: #BA2121”>“match_parent”</span>
<span style=“color: #7D9029”>android:layout_height=</span><span style=“color: #BA2121”>“match_parent”</span>
<span style=“color: #7D9029”>android:src=</span><span style=“color: #BA2121”>"@drawable/icon"</span> <span style=“color: #008000; font-weight: bold”>></span>
<span style=“color: #008000; font-weight: bold”></ImageView></span>

</LinearLayout>

    Activity
    <![if !supportLineBreakNewLine]>
    <![endif]>

Now adjust your activity. In this activity the thread is saved and the dialog is closed if the activity is destroyed.

import java.io.IOException;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.StatusLine;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpUriRequest;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import android.app.Activity;
import android.app.ProgressDialog;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.widget.ImageView;
public class ThreadsLifecycleActivity extends Activity {
// Static so that the thread access the latest attribute
private static ProgressDialog dialog;
private static ImageView imageView;
private static Bitmap downloadBitmap;
private static Handler handler;
private Thread downloadThread;
/** Called when the activity is first created. */

<span style=“color: #AA22FF”>@Override</span>
<span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>onCreate</span><span style=“color: #666666”>(</span>Bundle savedInstanceState<span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>super</span><span style=“color: #666666”>.</span><span style=“color: #7D9029”>onCreate</span><span style=“color: #666666”>(</span>savedInstanceState<span style=“color: #666666”>);</span>
setContentView<span style=“color: #666666”>(</span>R<span style=“color: #666666”>.</span><span style=“color: #7D9029”>layout</span><span style=“color: #666666”>.</span><span style=“color: #7D9029”>main</span><span style=“color: #666666”>);</span>
<span style=“color: #408080; font-style: italic”>//Create a handler to update the UI</span>
handler <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>new</span> Handler<span style=“color: #666666”>();</span>
<span style=“color: #408080; font-style: italic”>//get the latest imageView after restart of the application</span>
imageView <span style=“color: #666666”>=</span> <span style=“color: #666666”>(</span>ImageView<span style=“color: #666666”>)</span> findViewById<span style=“color: #666666”>(</span>R<span style=“color: #666666”>.</span><span style=“color: #7D9029”>id</span><span style=“color: #666666”>.</span><span style=“color: #7D9029”>imageView1</span><span style=“color: #666666”>);</span>
<span style=“color: #408080; font-style: italic”>//Did we already download the image?</span>
<span style=“color: #008000; font-weight: bold”>if</span> <span style=“color: #666666”>(</span>downloadBitmap <span style=“color: #666666”>!=</span> <span style=“color: #008000; font-weight: bold”>null</span><span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
imageView<span style=“color: #666666”>.</span><span style=“color: #7D9029”>setImageBitmap</span><span style=“color: #666666”>(</span>downloadBitmap<span style=“color: #666666”>);</span>
<span style=“color: #666666”>}</span>
<span style=“color: #408080; font-style: italic”>//Check if the thread is already running</span>
downloadThread <span style=“color: #666666”>=</span> <span style=“color: #666666”>(</span>Thread<span style=“color: #666666”>)</span> getLastNonConfigurationInstance<span style=“color: #666666”>();</span>
<span style=“color: #008000; font-weight: bold”>if</span> <span style=“color: #666666”>(</span>downloadThread <span style=“color: #666666”>!=</span> <span style=“color: #008000; font-weight: bold”>null</span> <span style=“color: #666666”>&&</span> downloadThread<span style=“color: #666666”>.</span><span style=“color: #7D9029”>isAlive</span><span style=“color: #666666”>())</span> <span style=“color: #666666”>{</span>
dialog <span style=“color: #666666”>=</span> ProgressDialog<span style=“color: #666666”>.</span><span style=“color: #7D9029”>show</span><span style=“color: #666666”>(</span><span style=“color: #008000; font-weight: bold”>this</span><span style=“color: #666666”>,</span> <span style=“color: #BA2121”>“Download”</span><span style=“color: #666666”>,</span> <span style=“color: #BA2121”>“downloading”</span><span style=“color: #666666”>);</span>
<span style=“color: #666666”>}</span>
<span style=“color: #666666”>}</span>

<span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>resetPicture</span><span style=“color: #666666”>(</span>View view<span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>if</span> <span style=“color: #666666”>(</span>downloadBitmap <span style=“color: #666666”>!=</span> <span style=“color: #008000; font-weight: bold”>null</span><span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
downloadBitmap <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>null</span><span style=“color: #666666”>;</span>
<span style=“color: #666666”>}</span>
imageView<span style=“color: #666666”>.</span><span style=“color: #7D9029”>setImageResource</span><span style=“color: #666666”>(</span>R<span style=“color: #666666”>.</span><span style=“color: #7D9029”>drawable</span><span style=“color: #666666”>.</span><span style=“color: #7D9029”>icon</span><span style=“color: #666666”>);</span>
<span style=“color: #666666”>}</span>

<span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>downloadPicture</span><span style=“color: #666666”>(</span>View view<span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
dialog <span style=“color: #666666”>=</span> ProgressDialog<span style=“color: #666666”>.</span><span style=“color: #7D9029”>show</span><span style=“color: #666666”>(</span><span style=“color: #008000; font-weight: bold”>this</span><span style=“color: #666666”>,</span> <span style=“color: #BA2121”>“Download”</span><span style=“color: #666666”>,</span> <span style=“color: #BA2121”>“downloading”</span><span style=“color: #666666”>);</span>
downloadThread <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>new</span> MyThread<span style=“color: #666666”>();</span>
downloadThread<span style=“color: #666666”>.</span><span style=“color: #7D9029”>start</span><span style=“color: #666666”>();</span>
<span style=“color: #666666”>}</span>

<span style=“color: #408080; font-style: italic”>// Save the thread</span>
<span style=“color: #AA22FF”>@Override</span>
<span style=“color: #008000; font-weight: bold”>public</span> Object <span style=“color: #0000FF”>onRetainNonConfigurationInstance</span><span style=“color: #666666”>()</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>return</span> downloadThread<span style=“color: #666666”>;</span>
<span style=“color: #666666”>}</span>

<span style=“color: #408080; font-style: italic”>// dismiss dialog if activity is destroyed</span>
<span style=“color: #AA22FF”>@Override</span>
<span style=“color: #008000; font-weight: bold”>protected</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>onDestroy</span><span style=“color: #666666”>()</span> <span style=“color: #666666”>{</span>

<span style=“color: #008000; font-weight: bold”>if</span> <span style=“color: #666666”>(</span>dialog <span style=“color: #666666”>!=</span> <span style=“color: #008000; font-weight: bold”>null</span> <span style=“color: #666666”>&&</span> dialog<span style=“color: #666666”>.</span><span style=“color: #7D9029”>isShowing</span><span style=“color: #666666”>())</span> <span style=“color: #666666”>{</span>
dialog<span style=“color: #666666”>.</span><span style=“color: #7D9029”>dismiss</span><span style=“color: #666666”>();</span>
dialog <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>null</span><span style=“color: #666666”>;</span>
<span style=“color: #666666”>}</span>
<span style=“color: #008000; font-weight: bold”>super</span><span style=“color: #666666”>.</span><span style=“color: #7D9029”>onDestroy</span><span style=“color: #666666”>();</span>
<span style=“color: #666666”>}</span>

<span style=“color: #408080; font-style: italic”>// Utiliy method to download image from the internet</span>
<span style=“color: #008000; font-weight: bold”>static</span> <span style=“color: #008000; font-weight: bold”>private</span> Bitmap <span style=“color: #0000FF”>downloadBitmap</span><span style=“color: #666666”>(</span>String url<span style=“color: #666666”>)</span> <span style=“color: #008000; font-weight: bold”>throws</span> IOException <span style=“color: #666666”>{</span>
HttpUriRequest request <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>new</span> HttpGet<span style=“color: #666666”>(</span>url<span style=“color: #666666”>.</span><span style=“color: #7D9029”>toString</span><span style=“color: #666666”>());</span>
HttpClient httpClient <span style=“color: #666666”>=</span> <span style=“color: #008000; font-weight: bold”>new</span> DefaultHttpClient<span style=“color: #666666”>();</span>
HttpResponse response <span style=“color: #666666”>=</span> httpClient<span style=“color: #666666”>.</span><span style=“color: #7D9029”>execute</span><span style=“color: #666666”>(</span>request<span style=“color: #666666”>);</span>
StatusLine statusLine <span style=“color: #666666”>=</span> response<span style=“color: #666666”>.</span><span style=“color: #7D9029”>getStatusLine</span><span style=“color: #666666”>();</span>
<span style=“color: #B00040”>int</span> statusCode <span style=“color: #666666”>=</span> statusLine<span style=“color: #666666”>.</span><span style=“color: #7D9029”>getStatusCode</span><span style=“color: #666666”>();</span>
<span style=“color: #008000; font-weight: bold”>if</span> <span style=“color: #666666”>(</span>statusCode <span style=“color: #666666”>==</span> <span style=“color: #666666”>200)</span> <span style=“color: #666666”>{</span>
HttpEntity entity <span style=“color: #666666”>=</span> response<span style=“color: #666666”>.</span><span style=“color: #7D9029”>getEntity</span><span style=“color: #666666”>();</span>
<span style=“color: #B00040”>byte</span><span style=“color: #666666”>[]</span> bytes <span style=“color: #666666”>=</span> EntityUtils<span style=“color: #666666”>.</span><span style=“color: #7D9029”>toByteArray</span><span style=“color: #666666”>(</span>entity<span style=“color: #666666”>);</span>
Bitmap bitmap <span style=“color: #666666”>=</span> BitmapFactory<span style=“color: #666666”>.</span><span style=“color: #7D9029”>decodeByteArray</span><span style=“color: #666666”>(</span>bytes<span style=“color: #666666”>,</span> <span style=“color: #666666”>0,</span> bytes<span style=“color: #666666”>.</span><span style=“color: #7D9029”>length</span><span style=“color: #666666”>);</span>
<span style=“color: #008000; font-weight: bold”>return</span> bitmap<span style=“color: #666666”>;</span>
<span style=“color: #666666”>}</span> <span style=“color: #008000; font-weight: bold”>else</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>throw</span> <span style=“color: #008000; font-weight: bold”>new</span> <span style=“color: #0000FF”>IOException</span><span style=“color: #666666”>(</span><span style=“color: #BA2121”>"Download failed, HTTP response code “</span> <span style=“color: #666666”>+</span> statusCode <span style=“color: #666666”>+</span> <span style=“color: #BA2121”>” - "</span> <span style=“color: #666666”>+</span> statusLine<span style=“color: #666666”>.</span><span style=“color: #7D9029”>getReasonPhrase</span><span style=“color: #666666”>());</span>
<span style=“color: #666666”>}</span>

<span style=“color: #666666”>}</span>

<span style=“color: #008000; font-weight: bold”>static</span> <span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #008000; font-weight: bold”>class</span> <span style=“color: #0000FF; font-weight: bold”>MyThread</span> <span style=“color: #008000; font-weight: bold”>extends</span> Thread <span style=“color: #666666”>{</span>
<span style=“color: #AA22FF”>@Override</span>
<span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>run</span><span style=“color: #666666”>()</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>try</span> <span style=“color: #666666”>{</span>
<span style=“color: #408080; font-style: italic”>// Simulate a slow network</span>
<span style=“color: #008000; font-weight: bold”>try</span> <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>new</span> <span style=“color: #0000FF”>Thread</span><span style=“color: #666666”>().</span><span style=“color: #7D9029”>sleep</span><span style=“color: #666666”>(5000);</span>
<span style=“color: #666666”>}</span><span style=“color: #008000; font-weight: bold”>catch</span> <span style=“color: #666666”>(</span>InterruptedException e<span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
e<span style=“color: #666666”>.</span><span style=“color: #7D9029”>printStackTrace</span><span style=“color: #666666”>();</span>
<span style=“color: #666666”>}</span>
downloadBitmap <span style=“color: #666666”>=</span> downloadBitmap<span style=“color: #666666”>(</span><span style=“color: #BA2121”>"http://www.vogella.com/img/lars/LarsVogelArticle7.png"</span><span style=“color: #666666”>);</span>
handler<span style=“color: #666666”>.</span><span style=“color: #7D9029”>post</span><span style=“color: #666666”>(</span><span style=“color: #008000; font-weight: bold”>new</span> MyRunnable<span style=“color: #666666”>());</span>
<span style=“color: #666666”>}</span> <span style=“color: #008000; font-weight: bold”>catch</span> <span style=“color: #666666”>(</span>IOException e<span style=“color: #666666”>)</span> <span style=“color: #666666”>{</span>
e<span style=“color: #666666”>.</span><span style=“color: #7D9029”>printStackTrace</span><span style=“color: #666666”>();</span>
<span style=“color: #666666”>}</span>
<span style=“color: #666666”>}</span>
<span style=“color: #666666”>}</span>

<span style=“color: #008000; font-weight: bold”>static</span> <span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #008000; font-weight: bold”>class</span> <span style=“color: #0000FF; font-weight: bold”>MyRunnable</span> <span style=“color: #008000; font-weight: bold”>implements</span> Runnable <span style=“color: #666666”>{</span>
<span style=“color: #008000; font-weight: bold”>public</span> <span style=“color: #B00040”>void</span> <span style=“color: #0000FF”>run</span><span style=“color: #666666”>()</span>
<span style=“color: #666666”>{</span>
imageView<span style=“color: #666666”>.</span><span style=“color: #7D9029”>setImageBitmap</span><span style=“color: #666666”>(</span>downloadBitmap<span style=“color: #666666”>);</span>
dialog<span style=“color: #666666”>.</span><span style=“color: #7D9029”>dismiss</span><span style=“color: #666666”>();</span>
<span style=“color: #666666”>}</span>
<span style=“color: #666666”>}</span>

}

Run your application and press the button to start a download. You can test the correct lifecycle behavior in the emulator via pressing “Ctrl+F11” as this changes the orientation.
It is important to note that the Thread is a static inner class. It is important to use a static inner class for your background process because otherwise the inner class will contain a reference to the class in which is was created. As the thread is passed to the new instance of your activity this would create a memory leak as the old activity would still be referred to by the Thread.
