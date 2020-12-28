# Why do we use ViewModel?

- When configuration changes, data won't be persistent.
- When we are developing professional level Android applications, one of the most common thing we need to consider is configuration changes.
- ViewModel never dies unless `onCleared()` is called while activity is going through its own lifecycle.
- ViewModel dies  when put into the background and the app process is killed in order to free up the system's memory.

# What are the configuration changes?

- Screen rotations
- Keyboard changes
- Language changes
- Enabling multiwindow mode

# ViewModel

- A model class for a view. (One ViewModel - One Activity)

⇒ ViewModel will always hold data. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa918dc4-5f0e-4218-a0a9-29ee131f49a7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa918dc4-5f0e-4218-a0a9-29ee131f49a7/Untitled.png)

# A Simple ViewModel Example

- On principle, one Activity has one `ViewModel`

### Gradle

```kotlin
implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
```

```kotlin
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "com.anushka.viewmodeldemo1"
        minSdkVersion 22
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
     dataBinding{
         enabled = true
     }
}

dependencies {
    def lifecycle_version = "2.2.0"
    
    //ViewModel and LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"

    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```

### Class that extends `ViewModel`

```kotlin
class MainActivityViewModel: ViewModel() {

    private var count = 0

    fun getCurrentCount(): Int {
        return count
    }

    fun getUpdatedCount(): Int {
        return ++count
    }

}
```

### Initialize ViewModel on Activity

```kotlin
class MainActivity : AppCompatActivity() {
   private lateinit var viewModel: MainActivityViewModel

    //private var count = 0
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
				
        viewModel = ViewModelProvider(this).get(MainActivityViewModel::class.java)

  
    }
}
```

### MainActivityViewModel

```kotlin
package com.anushka.viewmodeldemo1

import androidx.lifecycle.ViewModel

class MainActivityViewModel: ViewModel() {

    private var count = 0

    fun getCurrentCount(): Int {
        return count
    }

    fun getUpdatedCount(): Int {
        return ++count
    }

}
```

### MainActiviy

```kotlin
package com.anushka.viewmodeldemo1

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.ViewModelProvider
import com.anushka.viewmodeldemo1.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    private lateinit var viewModel: MainActivityViewModel

    //private var count = 0
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        viewModel = ViewModelProvider(this).get(MainActivityViewModel::class.java)

        binding.countText.text = viewModel.getCurrentCount().toString()
        binding.button.setOnClickListener {
            binding.countText.text = viewModel.getUpdatedCount().toString()
        }
    }
}
```

# When is the ViewModel's onCleared() called?

A ViewModel's onCleared() is called when the app is put into the background and the app process is killed in order to free up the system's memory.

# ViewModel Factory Class

### Why do we need ViewModelFactory?

We can not create ViewModel on our own. We need ViewModelProviders utility provided by Android to create ViewModels.

But ViewModelProviders can only instantiate ViewModels with **no arg constructor.**

**So if I have a ViewModel with multiple arguments, then I need to use a Factory** that I can pass to ViewModelProviders to use when an instance of MyViewModel is required.

### Process

1. Give constructor an argument to `ViewModel` Class
2. Create `class` file which implements `ViewModelProvider.Factory`

# Process

### 1. Prepare for MainActivityViewModel with constructor arguments

```kotlin
class MainActivityViewModel(startingTotal: Int) : ViewModel() {
    private var total = 0

    init {
        total = startingTotal
    }

    fun getTotal():Int{
        return total
    }

    fun setTotal(input:Int){
        total +=input
    }
}
```

### 2. Prepare `MainActivityViewModelFactory` with constructor arguments which implement `ViewModelProvider.Factory`

```kotlin
package com.anushka.viewmodeldemo2

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import java.lang.IllegalArgumentException

class MainActivityViewModelFactory(private val startingTotal: Int): ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if(modelClass.isAssignableFrom(MainActivityViewModel::class.java)) {
            return MainActivityViewModel(startingTotal) as T
        }
        throw IllegalArgumentException("Unkown View Model Class")
    }

}
```

### 3. Initialize it on Activity

```kotlin
package com.anushka.viewmodeldemo2

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.ViewModelProvider
import com.anushka.viewmodeldemo2.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    private lateinit var viewModel: MainActivityViewModel
    private lateinit var viewModelFactory: MainActivityViewModelFactory

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        /** Initialize ViewModelFactory **/
        viewModelFactory = MainActivityViewModelFactory(125)

        /** Initialize ViewModel With Factory Instance **/
        viewModel = ViewModelProvider(this, viewModelFactory).get(MainActivityViewModel::class.java)

        binding.resultTextView.text = viewModel.getTotal().toString()

        binding.insertButton.setOnClickListener {
            viewModel.setTotal(binding.inputEditText.text.toString().toInt())
            binding.resultTextView.text = viewModel.getTotal().toString()

        }

    }
}
```