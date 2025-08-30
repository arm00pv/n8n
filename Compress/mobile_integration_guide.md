# Guide: Integrating the Compression API with Mobile Apps

This guide provides step-by-step instructions and code examples for calling the n8n-powered compression API from a mobile application. We will cover implementations for both Flutter and native Android (Kotlin).

**API Endpoint:**
Before you begin, make sure you have the **Production URL** for your n8n webhook from the `compress_api_guide.md`. It will look something like this: `https://n8n.your-domain.com/webhook-production/1234-abcd...`

---

## 1. Flutter (Dart) Implementation

This implementation uses the standard `http` package for making HTTP requests.

### a. Add Dependencies

First, add the necessary packages to your `pubspec.yaml` file:

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.4  # Or a more recent version
  path_provider: ^2.0.11 # To get a directory to save the file
  image_picker: ^0.8.6 # To pick an image from the gallery
```

Then, run `flutter pub get` in your terminal.

### b. Create the Upload Function

Here is a sample Dart function that takes an image file (`XFile` from `image_picker`), uploads it to the n8n API, and saves the compressed response.

```dart
import 'dart:io';
import 'package:http/http.dart' as http;
import 'package:path_provider/path_provider.dart';
import 'package:image_picker/image_picker.dart';

Future<File?> uploadAndCompressImage(XFile imageFile) async {
  // Your n8n webhook URL
  final url = Uri.parse('YOUR_N8N_PRODUCTION_WEBHOOK_URL');

  try {
    // Create a multipart request
    var request = http.MultipartRequest('POST', url);

    // Attach the file
    request.files.add(await http.MultipartFile.fromPath(
      'file', // This key 'file' is the default for n8n's binary data
      imageFile.path,
      filename: imageFile.name,
    ));

    print('Uploading image...');
    final streamedResponse = await request.send();

    if (streamedResponse.statusCode == 200) {
      print('Image uploaded successfully. Downloading compressed version...');
      // Get the response as bytes
      final responseBytes = await streamedResponse.stream.toBytes();

      // Get a temporary directory to save the file
      final directory = await getTemporaryDirectory();
      final compressedFilePath = '${directory.path}/compressed_${imageFile.name}';
      final compressedFile = File(compressedFilePath);

      // Write the compressed bytes to a new file
      await compressedFile.writeAsBytes(responseBytes);

      print('Compressed image saved to: $compressedFilePath');
      return compressedFile;
    } else {
      print('Error uploading image: ${streamedResponse.statusCode}');
      print(await streamedResponse.stream.bytesToString());
      return null;
    }
  } catch (e) {
    print('An error occurred: $e');
    return null;
  }
}
```

---

## 2. Native Android (Kotlin) Implementation

This implementation uses `OkHttp`, a popular and robust networking library for Android.

### a. Add Dependencies

First, add the OkHttp dependency to your app-level `build.gradle` file:

```groovy
dependencies {
    // ... other dependencies
    implementation("com.squareup.okhttp3:okhttp:4.9.3")
}
```

Also, ensure you have the `INTERNET` permission in your `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### b. Create the Upload Function

Here is a sample Kotlin function that uploads an image file from the device to the n8n API. This should be run in a background thread or a coroutine to avoid blocking the main UI thread.

```kotlin
import okhttp3.*
import okhttp3.MediaType.Companion.toMediaTypeOrNull
import okhttp3.RequestBody.Companion.asRequestBody
import java.io.File
import java.io.FileOutputStream
import java.io.IOException

fun uploadAndCompressImage(imageFile: File, outputDirectory: File) {
    // Your n8n webhook URL
    val url = "YOUR_N8N_PRODUCTION_WEBHOOK_URL"

    val client = OkHttpClient()

    try {
        // Create the request body with the image file
        val requestBody = MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart(
                "file", // This key 'file' is the default
                imageFile.name,
                imageFile.asRequestBody("image/jpeg".toMediaTypeOrNull())
            )
            .build()

        // Build the request
        val request = Request.Builder()
            .url(url)
            .post(requestBody)
            .build()

        // Execute the request
        client.newCall(request).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                println("Error uploading image: ${e.message}")
            }

            override fun onResponse(call: Call, response: Response) {
                if (response.isSuccessful) {
                    println("Upload successful. Saving compressed file...")
                    // Get the response body as bytes
                    val responseBytes = response.body?.bytes()

                    if (responseBytes != null) {
                        // Save the compressed file
                        val compressedFile = File(outputDirectory, "compressed_${imageFile.name}")
                        FileOutputStream(compressedFile).use {
                            it.write(responseBytes)
                        }
                        println("Compressed file saved to: ${compressedFile.absolutePath}")
                    }
                } else {
                    println("Upload failed: ${response.code} ${response.message}")
                }
            }
        })
    } catch (e: Exception) {
        println("An error occurred: ${e.message}")
    }
}
```
